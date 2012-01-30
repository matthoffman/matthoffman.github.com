---
title: Takes too long to build your project? Try harder.
layout: post 
summary: where I talk, eventually, about setting up Jenkins as a precommit testing server (a "try" server)
---

So, we have a problem at work. Well, more than one, perhaps...but one of them is that builds take too long. I've talked to folks I know in different places, and this is a pretty common complaint. But of course, [we have it tough](http://www.youtube.com/watch?v=-eDaSvRO9xA). We have a large codebase and a lot of tests. The compile time for the codebase is one issue; I'll get to that in another blog post. But the test time is worse. 

<div id="the-amazon-clouds" style="display:none; float:right" class="span4 offset4" onclick="$('div[id=the-amazon-clouds]').hide()"><img src="/images/carealot.gif"/></div>

Most of our tests are proper unit tests: they test a single unit of code, mock out all	 connected components, and run quite quickly. But we have a significant minority of tests that go against a database, and the development databases live out on EC2 instances in the Amazon <a id="cloud-link" onclick='$("div[id=the-amazon-clouds]").show();' >clouds</a>. The time to run all pure unit tests across the project is pretty significant, but the time to run all the unit tests on a developer machine untenable. Running tests for one module (out of 10 or so) takes half an hour on a developer machine hitting a development database on an EC2 instance.  


<!-- I don't like this effect as much; I can't seem to control the title header: a href="#" rel="popover" data-html="true" data-offset="0" title="Amazon clouds"  data-content="<img src='/images/carealot.gif' border=0 width=247 height=201>">clouds</a> -->
<script>
/*
            $(function () {
              $("a[rel=popover]")
                .popover({
                  html: true 
                })
                .click(function(e) {
                  e.preventDefault()
                })
            })
            $(function () { 
              $("a[id=cloud-link]").onclick( 
                    function(e) {
                         $("img[id=the-amazon-clouds]").show();
                    })
            })
*/

</script>


Whoa, hold on there – I can hear some of you now: "Tests that go against the database!? Those aren't unit tests! Unit tests CAN'T TOUCH ANYTHING! AT ALL!" Ok, fine. They're not unit tests. Call them integration tests. Call them whatever you want. The fact remains, there is logic contained in the DAO layer, and in the queries themselves (some of the SQL queries are substantially complicated). Somehow or other, that logic needs to be tested. 

So, there are a few options I know of: 

### Mock out the database itself
[Martin Fowler suggests this](http://martinfowler.com/bliki/InMemoryTestDatabase.html). This would have to be a complete enough fake-database that we could test the queries themselves and catch unbound bind variables, bad column names, and things like that, while retaining full Oracle compatibility. I haven't seen anything that actually does this, but if you know of it, let me know. Before someone suggests it, H2 and similar in-memory databases don't work for us because we use a lot of Oracle-specific SQL extensions (partition clauses, SQL hints, and so on). Even Oracle's own TimesTen database doesn't support these, unfortunately. If they did work, they'd be a nice option, though.

### Cut and run: Mock out as much DB interaction as possible, and put the tests that do hit the DB in a separate group that run only occasionally. 

We tried this first. I created a Maven profile, so that adding <code>-P quick<code> to the Maven command line skips all the database tests. I could have done it the other way around, so that you had to add a profile (or use the [integration test pattern](maven site) to run the database tests, but I was afraid they'd never run. I had nightmares of folks setting up CI jobs for a new branch and forgetting to add the magic "run the database tests, too" parameter, and not seeing database-related errors until it got out onto a server. Yes, we have a QA process after unit tests, but if a bug makes it to that layer, it's made it too far; it's holding up other testing. 

There's also something in me that makes me uncomfortable when there is a disincentive to writing tests. In an ideal world, we'd be able to say, "Write all the tests you want. Load all the test data you need. We can run them." So the "quick" profile was OK, but I knew we could do better. And it just happened that, elsewhere in the company, some folks were working on another project:

### Move the data to the computation.
The team that maintains the development databases created a VirtualBox image of a database server, so that developers could run it locally. No more network lag, and much faster tests (about 2x faster, in limited tests). But there were some problems: a.) developer laptops are only so fast, and b.) memory. I'd love to say that each devleper could have a separate desktop workstation to use as a development database, but unfortunately the budget isn't there. And while we have 8gb of RAM on our Macbooks, after running a local version of our app, some development tools, and Firefox, we're often hitting swap space. So… the dev VM is less than ideal. If I could get 16gb of RAM on my machine, then we'd be in business. But the ops team balked at that. Something about not being officially supported by Apple. So that leads us to..

### Move the computation to the data
Hadoop does it, why can't I?  (link to the MapReduce paper...that mentions moving computation to data, right?) I think the first two options have a lot going for them, and are probably enough for a lot of environments. But if software gets big enough, there's always going to be a point where the tests take too long to run locally. And as fast as your development machine is, you can always get a cluster of build servers that are faster. And that's probably always going to be more economical than giving each developer a beefy PC. So it's not hard to imagine eventually wanting to do your test builds out on the CI machine. 

We use Hudson…er… Jenkins as a CI server already; it builds the proejct after each checkin. So all we needed was a way to make it build the project BEFORE a checkin – what a lot of people call a "try" server (as in, "try it before you commit it"). 

I did a bit of searching, and a lot of smart people have solved this problem already: 
Etsy, for example, [uses Jenkins as a "try" server](http://codeascraft.etsy.com/2011/10/11/did-you-try-it-before-you-committed/). Their solution is clean and simple: just send the diff of your code against master, and Jenkins will apply the diff, compile, and build. Nice. Most other solutions seem to take similar approaches: [Mozilla](https://wiki.mozilla.org/ReleaseEngineering/TryServer), [buildbot](http://buildbot.net/buildbot/docs/0.8.4/try.html) (used by the [Chromium project](http://www.chromium.org/developers/testing/try-server-usage), among others)… and so on. 

Unfortunately, we have a lot of modules and a lot of branches. The branches are unavoidable – we support a number of clients. The number of modules…well, that my fault. I split the software into a lot of modules several years ago to try to encourage code reuse between projects, but the cost has been much higher than the benefit. Unfortunately, it's now costly to merge things back together – but that's another post for another time.

So, rather than setting up one "try" build per branch per module, doubling the number of CI jobs, I opted to do things differently.  I created a "try.sh" script that runs the "compile" and "package" portions of the build, skipping tests, and then sends the compiled classes and test classes over to Jenkins. The Jenkins job just unpackages the classes and runs the enclosed tests. 

The Jenkins job, in this scenario, is entirely generic – it knows nothing about the code its testing. It doesn't connect to source control at all. It doesn't even see the source – we're sending it compiled class files, and it's just running the tests. 

Is this the right solution for you?  Perhaps, if your constraints are similar to ours. Read the Etsy post linked above, and see if that solution seems simpler: it may well be. If you want to set up something like we did, read on. 

Setting it up is easy. Since we use Maven, we just have to make sure each of our Maven builds creates a test-jar: 
<maven package snippet here>

The try.sh script looks like this: 


The Jenkins job looks like this: 


parameterized builds: http://morethanseven.net/2011/11/16/Jenkins-parameterized-builds.html   

Remember to mention setting up ssh keys for login. 

Todo: set up ssh keys on the jenkins server so that I can use the description setter plgin:  https://wiki.jenkins-ci.org/display/JENKINS/Description+Setter+Plugin

Description : 

Lets you upload local code that hasn't been committed to the repository yet in order to let Jenkins run your tests for you. Running tests (especially DB-intensive tests) can take forever, so in practice we tend to not run them So this build is an attempt to make it more likely that we actually run tests. To use it, use the try.sh script that is in git:repos.quantumretail.com:qi/bin (the bin directory of the qi uberproject). 

username: 
you rusername, for logigng purposes

filename: 
the full path of the tar.gz file on the jenkins server. (note: if we're multi-node, this gets...interesting, right?  Might have to explore jenkins-cli again)

echo "Building try number ${BUILD_NUMBER} for user ${username}"

# This shouldn't be necessary -- Jenkins should keep each build separate -- but it seems that it wasn't the case. So we'll keep each build in a subdirectory by build number, which is easy enough. 
_dirname=${BUILD_NUMBER} # this has to match up with the Root POM setting below. 
mkdir $_dirname
tar  xzvf ${filename} -C ${_dirname}
cd ${_dirname}

# unzip each of our -tests.jar files into a test-classes directory
find . -name "*-tests.jar" -execdir unzip -o {} -d test-classes \;

# unzip the normal jar files into a classes directory
find . -name "*SNAPSHOT.jar" -execdir unzip -o {} -d classes \;



---

invoke top-level maven target: 

surefire:test -fae

pom: ${BUILD_NUMBER}/pom.xml

    


