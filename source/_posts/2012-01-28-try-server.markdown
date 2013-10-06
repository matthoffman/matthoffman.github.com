---
title: Takes too long to build your project? Try harder.
layout: post 
summary: where I talk, eventually, about setting up Jenkins as a precommit testing server (a "try" server)
comments: true
---

So, we have a problem at work. Well, more than one, perhaps...but one of them is that builds take too long. I've talked to folks I know in different places, and this is a pretty common complaint. But of course, [we have it tough](http://www.youtube.com/watch?v=-eDaSvRO9xA). We have a large codebase and a lot of tests. The compile time for the codebase is one issue; I'll get to that in another blog post. But the test time is worse. 

<div id="the-amazon-clouds" style="display:none; float:right" class="span4 offset4" onclick="$('div[id=the-amazon-clouds]').hide('fast')"><img src="/images/carealot.gif"/></div>

Most of our tests are proper unit tests: they test a single unit of code, mock out all components other than the one under test, and run quite quickly. But we have a significant minority of tests that go against a database, and the development databases live out on EC2 instances in the Amazon <a id="cloud-link" onclick='$("div[id=the-amazon-clouds]").show("fast");' >clouds</a>. The time to run all pure unit tests across the project is pretty significant, but the time to run all the unit tests on a developer machine is untenable. Running tests for one module (out of 10 or so) takes half an hour when the code is on a developer's machine and the development database is on an EC2 instance.  

Whoa, hold on there – I can hear some of you now: "Tests that go against the database!? Those aren't unit tests! Unit tests CAN'T TOUCH ANYTHING! AT ALL!" Ok, fine. They're not unit tests. Call them integration tests. Call them whatever you want. The fact remains, there is logic contained in the DAO layer, and in the queries themselves (some of the SQL queries are substantially complicated). Somehow or other, that logic needs to be tested. 

So, there are a few options I know of: 

### Mock out the database itself


This would have to be a complete enough fake-database that we could test the queries themselves and catch unbound bind variables, bad column names, and things like that, while retaining full Oracle compatibility. I haven't seen anything that actually does this, but if you know of it, let me know. In-memory databases like H2 are [often used as a substitute for the database while testing](http://martinfowler.com/bliki/InMemoryTestDatabase.html), but they don't work for us because we use a lot of Oracle-specific SQL extensions (partition clauses, SQL hints, and so on). Even Oracle's own TimesTen database doesn't support these, unfortunately. If they did work, they'd be a nice option, though.


### Cut and run: Mock out as much DB interaction as possible, and put the tests that do hit the DB in a separate group that run only occasionally. 


We tried this first. I created a Maven profile, so that adding <code>-P quick</code> to the Maven command line skips all the database tests. I could have done it the other way around, so that you had to add a profile to run the database tests, but I was afraid we'd never run them. I had nightmares of folks setting up CI jobs for a new branch and forgetting to add the magic "run the database tests, too" parameter, and not seeing database-related errors until it got out onto a server. Yes, we have a QA process after unit tests, but if a bug makes it to that layer, it's made it too far; it's holding up other testing. I may still move the tests that hit the database into the [integration-test phase](http://stackoverflow.com/questions/1228709/best-practices-for-integration-tests-with-maven), but that feels like a hack; they're not really integration tests.


There's also something that makes me uncomfortable when there is a disincentive to writing tests. In an ideal world, we'd be able to say, "Write all the tests you want. Load all the test data you need. We'll run them." So the "quick" profile was OK, but I knew we could do better. And it just happened that, elsewhere in the company, some folks were working on another project:


### Move the data to the computation.


The team that maintains the development databases created a VirtualBox image of a database server, so that developers could run it locally. No more network lag, and much faster tests (about 2x faster, in early benchmarks). But there were some problems: developer laptops are only so fast, and they have only so memory. I'd love to say that each devleper could have a separate desktop workstation to use as a development database, but unfortunately the budget isn't there. And while we have 8gb of RAM on our Macbooks, after running a local version of our app, some development tools, and Firefox, we're often hitting swap space. So… the dev VM is less than ideal. If I could get 16gb of RAM on my machine, then I'd be in business. But the ops team balked at that. Something about not being officially supported by Apple. So that leads us to...


### Move the computation to the data


Hadoop does it, why can't I?  I think the first few options have a lot going for them, and are probably enough for a lot of environments. But if software gets big enough, there's always going to be a point where the tests take too long to run locally. And as fast as your development machine is, you can always get a cluster of build servers that are faster. And that's probably always going to be more economical than giving each developer a beefy PC. So it's not hard to imagine eventually wanting to do your test builds out on the CI machine. 

We use Hudson…er… Jenkins as a CI server already; it builds the proejct after each checkin. So all we needed was a way to make it build the project BEFORE a checkin – what a lot of people call a "try" server (as in, "try it before you commit it"). 

I did a bit of searching, and a lot of smart people have solved this problem already: 
Etsy, for example, [uses Jenkins as a "try" server](http://codeascraft.etsy.com/2011/10/11/did-you-try-it-before-you-committed/). Their solution is clean and simple: just send the diff of your code against master, and Jenkins will apply the diff, compile, and build. Nice. Most other solutions seem to take similar approaches: [Mozilla](https://wiki.mozilla.org/ReleaseEngineering/TryServer), [buildbot](http://buildbot.net/buildbot/docs/0.8.4/try.html) (used by the [Chromium project](http://www.chromium.org/developers/testing/try-server-usage), among others)… and so on. 

Unfortunately, we have a lot of modules and a lot of branches. The branches are unavoidable – we support a number of clients. The number of modules…well, that my fault. I split the software into a lot of modules several years ago to try to encourage code reuse between projects, but the cost has been much higher than the benefit. Unfortunately, it's now costly to merge things back together – but that's another post for another time.

So, rather than setting up one "try" build per branch per module, doubling the number of CI jobs, I opted to do things differently.  I created a "try.sh" script that runs the "compile" and "package" portions of the build, skipping tests, and then sends the compiled classes and test classes over to Jenkins. The Jenkins job just unpackages the classes and runs the enclosed tests. 

The Jenkins job, in this scenario, is entirely generic – it knows nothing about the code its testing. It doesn't connect to source control at all. It doesn't even see the source – we're sending it compiled class files, and it's just running the tests. 

Is this the right solution for you?  Perhaps, if your constraints are similar to ours. Read the Etsy post linked above, and see if that solution seems simpler: it may well be. If you want to set up something like we did, read on. 

I'll say upfront that this method has some disadvantages. The main disadvantage is that it's slower. The traditional try server does a diff of your local code vs. the mainline, then sends that diff up to the server and applies it, then runs the build and tests up on the server. So all that goes over the wire is your diff, and all your machine has to do is diff the code – all building is done on the CI server. 

My approach, instead, does the building on your local machine, and sends the artifacts to the CI server. This is a compromise: your machine may or may not build faster than the CI server (probably not) but the artifacts are likely a larger upload. So, if in doubt, try it out and see if it works for you. 

Setting it up is easy. Since we use Maven, we just have to make sure each of our Maven builds creates a test-jar: 

<pre class='brush: xml'>
    &lt;plugin&gt;
                &lt;groupId&gt;org.apache.maven.plugins&lt;/groupId&gt;
                &lt;artifactId&gt;maven-jar-plugin&lt;/artifactId&gt;
                &lt;version&gt;2.3.2&lt;/version&gt;
                &lt;configuration&gt;
                    &lt;archive&gt;
                        &lt;manifest&gt;
                            &lt;addClasspath&gt;true&lt;/addClasspath&gt;
                            &lt;addDefaultSpecificationEntries&gt;true&lt;/addDefaultSpecificationEntries&gt;
                            &lt;packageName&gt;${project.groupId}&lt;/packageName&gt;
                        &lt;/manifest&gt;
                        &lt;manifestEntries&gt;
                            &lt;url&gt;${project.url}&lt;/url&gt;
                            &lt;Implementation-Title&gt;${project.name}&lt;/Implementation-Title&gt;
                            &lt;Implementation-Version&gt;${project.version}-${buildNumber}&lt;/Implementation-Version&gt;
                            &lt;Implementation-Vendor-Id&gt;${project.groupId}&lt;/Implementation-Vendor-Id&gt;
                            &lt;Implementation-Vendor&gt;${project.organization.name}&lt;/Implementation-Vendor&gt;
                        &lt;/manifestEntries&gt;
                    &lt;/archive&gt;
                &lt;/configuration&gt;
                &lt;executions&gt;
                    &lt;execution&gt;
                        &lt;id&gt;jar&lt;/id&gt;
                        &lt;phase&gt;package&lt;/phase&gt;
                        &lt;goals&gt;
                            &lt;goal&gt;jar&lt;/goal&gt;
                        &lt;/goals&gt;
                    &lt;/execution&gt;
                    &lt;execution&gt;
                        &lt;id&gt;test-jar&lt;/id&gt;
                        &lt;phase&gt;test&lt;/phase&gt;
                        &lt;goals&gt;
                            &lt;goal&gt;test-jar&lt;/goal&gt;
                        &lt;/goals&gt;
                    &lt;/execution&gt;
                    &lt;execution&gt;
                        &lt;id&gt;source-jar&lt;/id&gt;
                        &lt;phase&gt;test&lt;/phase&gt;
                        &lt;goals&gt;
                            &lt;goal&gt;jar&lt;/goal&gt;
                        &lt;/goals&gt;
                        &lt;configuration&gt;
                            &lt;classesDirectory&gt;src/main/java&lt;/classesDirectory&gt;
                            &lt;classifier&gt;sources&lt;/classifier&gt;
                        &lt;/configuration&gt;
                    &lt;/execution&gt;
                    &lt;execution&gt;
                        &lt;id&gt;test-source-jar&lt;/id&gt;
                        &lt;phase&gt;test&lt;/phase&gt;
                        &lt;goals&gt;
                            &lt;goal&gt;jar&lt;/goal&gt;
                        &lt;/goals&gt;
                        &lt;configuration&gt;
                            &lt;classesDirectory&gt;src/test/java&lt;/classesDirectory&gt;
                            &lt;classifier&gt;tests-sources&lt;/classifier&gt;
                        &lt;/configuration&gt;
                    &lt;/execution&gt;
                &lt;/executions&gt;
   &lt;/plugin&gt;
</pre>

That snippet is in a parent pom, so that's taken care of for all of our projects. 

The try.sh script looks like this, currently. It could be a lot prettier, but this was the relatively quick-n'-dirty experiment: 


<pre class='brush: bash'>
    #!/bin/sh 
    
    ###############################################################################
    # try.sh
    # 
    # The idea behind this script is to build and package up your project locally, 
    # then send the code (the classes and the test classes) to the Jenkins server
    # to be executed there. 
    # This relies on the build being configured to generate a "test-jar" artifact
    # It has some limitations: 
    #   * You're not going to have exactly the same classpath (yet)
    #   * It can't handle multi-module projects (yet)
    #   * You can't add Maven profiles to the build (yet)
    ###############################################################################
    
    set -u   # fail on unitialized variable
    set -e   # fail if any command exits with a non-zero status
    
    # Find our local directory, so we can locate the wait-and-notify.sh script later
    DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
    
    JENKINS_HOST=your_build_server_here
    JENKINS_HTTP_PORT=7700
    # This is set in Jenkins' configuration; you probably have to configure Jenkins to have a static port. By default it changes every restart.
    JENKINS_SSH_PORT=54512
    
    # clear out old builds, so we don't upload the wrong thing
    rm -f target/try-$USER-*
    
    mvn -Pdeva -DskipTests=true $@ package  
    
    _date=`date -u +%F.%H%M%S%s`
    _filename=try-$USER-try-$_date.tar.gz
    _filepath=target/$_filename
    
    # This would be the simple version: 
    # tar czvf $_filepath pom.xml target/*.jar  
    # ...but that only handles single-module projects.  This handles multi-module projects: 
    find . -name "pom.xml" -or -name "*.jar" | xargs tar -czvf $_filepath
    
    echo "created try package at $_filepath"
    echo "now uploading to Jenkins..."

    $DIR/wait-and-notify.sh $JENKINS_HOST $JENKINS_SSH_PORT $JENKINS_HTTP_PORT $_filename $_filepath $USER &
    #Backgrounded; we don't want to wait.  The script will notify on completion using Growl.
</pre>

The "wait-and-notify.sh" script just looks like this: 


<pre class='brush: bash'>
    #!/bin/sh 

    set -e 
    set -u 
    
    HOST=$1
    SSH_PORT=$2
    HTTP_PORT=$3
    FILENAME=$4
    FILEPATH=$5
    USERNAME=$6
    
    scp $FILEPATH  $HOST:/tmp/$FILENAME
    ssh $HOST -p $SSH_PORT build try-build -p filename=/tmp/$FILENAME -p username=$USERNAME -s  
    # This is also an option, but jenkins-cli doesn't handle an encrypted private key, and who leaves their private key unencrypted?
    #java -jar jenkins-cli.jar -i ~/.ssh/id_dsa -s $1/job/try-build  build try-build -p attempt.tar.gz=$2 -p username=$3 -s  
    # This would be an alternative to using private keys for jenkins-cli.jar, but you'd need to enter a username and password 
    # --username `git config user.name` --password (something entered at the command line)
    
    echo "Job has been started on Jenkins. See http://$HOST:$HTTP_PORT/job/try-build for status."
    
    # Did the build succeed?
    if [ $? -eq 0 ] 
    then
      growlnotify "Try completed successfully"
    else  
      growlnotify "Try failed!"
    fi
</pre> 

The Jenkins job is a basic build with two parameters, one called "username" and one called "filename". 
The build runs this script as its first step: 


<pre class='brush: bash'>
    echo "Building try number ${BUILD_NUMBER} for user ${username}"
    
    # This shouldn't be necessary -- Jenkins should keep each build separate -- but it seems that it wasn't the case. So we'll keep each build in a subdirectory by build number, which is easy enough. 
    _dirname=${BUILD_NUMBER} 
    mkdir $_dirname
    tar  xzvf ${filename} -C ${_dirname}
    cd ${_dirname}
    
    # unzip each of our -tests.jar files into a test-classes directory
    find . -name "*-tests.jar" -execdir unzip -o {} -d test-classes \;
    
    # unzip the normal jar files into a classes directory
    find . -name "*SNAPSHOT.jar" -execdir unzip -o {} -d classes \;
</pre>

It then invokes a top-level Maven target:  <code>surefire:test -fae</code>. This runs the actual Maven tests, but without any build step (since we're invoking the surefire:test plugin explicitly and not the "test" phase.  <code>-fae</code> tells it to fail at the end instead of as soon as it sees a test failure; it's entirely optional. 

There's a lot of potential improvement.  For example, right now it's hard to tell just by looking at the Jenkins build queue who's try is building. We could fix that within the build by using the [description setter plugin](https://wiki.jenkins-ci.org/display/JENKINS/Description+Setter+Plugin). I haven't tried that yet. 

Try it out, let me know what you think. Or, if you have a better idea of how to get around long database-bound tests, I'm all ears.
