---
layout: post
title: "Getting started with Solr and Clojure" 
date: 2013-12-31 11:29
comments: true
summary: "where I offer a quickstart application for Clojure applications using Solr"
categories: 
---

I've been doing a lot of work with Solr recently, at work and on some personal projects. I use Clojure for most personal projects these days, and the few guides out there about using Clojure with Solr (as of this writing) were fairly out-of-date, so I thought I'd offer this one.

The primary way of interacting with Solr in Java is via the SolrJ client library. You can use it directly in Clojure if you want -- it's a fairly simple library with a straightforward API -- but there are no less than 4 libraries that wrap it in Clojurey syntax: 

* [Flux](https://github.com/mwmitchell/flux)
* [Solrclj](https://github.com/mlehman/solrclj)
* [Icarus](https://github.com/mattdeboard/Icarus)
* [Clojure-Solr](https://github.com/mikejs/clojure-solr)

Of these, Flux, by Matt Mitchell, is the most recently updated and the only one that bundles SolrJ 4.x. Solrclj uses version 3.6.2, so it would likely work fine with 4.x without modifications, but I haven't tried. The other two are significantly older. 

If you're interested in following along, I've put together [a quickstart application](https://github.com/matthoffman/solr-clojure-sample) that you might find useful. The samples in this post are all in that project. I used Flux as the Clojure wrapper of choice, but it would be simple to switch that out for Solrclj or to use SolrJ directly, if you felt so inclined.



A note on IDEs, etc.
------------------
Chances are, you already have a Clojure setup that you are quite happy with. But just in case you don't:

 * If you're comfortable with Emacs, go ahead and use that. There's a lot of information about getting started with Emacs on [clojure-doc](http://clojure-doc.org/articles/tutorials/emacs.html).
 * If you're not fully conversant in Emacs (and there's no shame in that), and you're coming from the Java world, I've included IntelliJ project files in the quickstart project. [IntelliJ Community Edition](http://www.jetbrains.com/idea/download/) and the [Cursive Clojure](http://cursiveclojure.com/eap.html) plugin make for an easy way to get started. Eclipse and the Counterclockwise plugin work too, and are quite popular, but I haven't used it in a while...if you want to contribute Eclipse project files for the quickstart, send me a pull request.

If you don't have it already, [download Leiningen](http://leiningen.org), the most common Clojure build tool. It has a one-line installation script. 

Setting up a sample Solr server
-------------------------------

This sample application connects to a running Solr server. Setting up a server is simple. One simple way to do it:

Download Solr from [http://lucene.apache.org/solr/](http://lucene.apache.org/solr/) and unzip it.
Go into the `examples/` directory and run:

     java -jar start.jar

That will start a sample Solr server.  Keep that terminal open.

REPL
------------

Now that there's a Solr server running, time to fire up a REPL in the solr-clojure-sample project. 
If you're using IntelliJ, you can do that using `Run -> lein repl`, and you'll have the REPL there inside your IDE, and you can execute things in the editor window directly in the REPL and all that goodness that Emacs users are used to. You can also choose "debug" instead of "run", and you'll be able to set breakpoints, which Emacs users are not used to. 

Alternately, you can just run "lein repl" from the command line. You miss out on the IDE integration goodness, but it will work fine.

Either way you start the REPL it will automatically load `dev/user.clj` (because project.clj says to).
This loads a set of useful namespaces as well as some helpful functions for development: init, start, stop, go, and reset.
For more on the philosophy behind these, see [Stuart Sierra's blog post](http://thinkrelevance.com/blog/2013/06/04/clojure-workflow-reloaded) or the [related podcast](http://thinkrelevance.com/blog/2013/05/29/stuart-sierra-episode-032). These have nothing to do with Solr specifically -- they're just a Clojure software architecture design pattern -- but since the quickstart doesn't make much sense if you don't understand what they're for, I'll go into a little detail.

In short, there's a map called `blast.solr/config` that contains a variety of configuration properties for the app. A production app would take these config settings and use them to build a Solr connection which it would then hold on to for the life of the application. How it holds onto that connection will vary by application... for development purposes, we just have an atom in dev/user.clj called "system". Calling `(go)` from your REPL will use the config map in `blast.solr/config` and connect to your sample Solr server, then populate the `system` atom with that connection. 

If for some reason you make changes that require resetting the connection, or changes to the configuration map, you can run:

    (reset)

This will close any existing connection, reload the namespaces, and reconnect. 

Load Data
-----------------

Solr isn't very interesting without any data, so let's load some. I've included a text file called "sonnets.edn" which contains all of Shakespeare's sonnets, in [EDN](https://github.com/edn-format/edn) format. The fields are already named using some Solr conventions: *_t to denote text fields, *_i to denote ints, and so on. 
To load the data, make sure you've run `(go)` in the REPL to establish a connection, and then run: 

     ;; We'll read a file line-by-line and load it into Solr
     (flux/with-connection (conn)
                        (with-open [rdr (io/reader "sonnets.edn")]
                          (doseq [line (line-seq rdr)]
                            (println line)
                            (flux/add (edn/read-string line)))
                          (flux/commit)
                          (flux/query "*:*"))) ; query our new data




Run some queries
----------------

The quickstart app has a few helper functions in user.clj that might be useful. `q`, for example, runs queries:

    (q "*:*")

There are some commented-out forms there that will load some sample data into Solr for experimentation. For example:

     (q "line_t:desire")
     (q "line_t:desire" {:deftype "edismax"}) ; send query args if you want
     (q "line_t:desire" {:deftype :edismax})  ; flux  will convert keywords into strings, so use them at will.
     (q "{!edismax qf=\"line_t\"}desire")     ; version with local variables. Anything that's valid in a Solr query is valid here.


Enjoy!

