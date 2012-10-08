---
title: SSL curiosity
layout: post
summary: where I mention a particular error with locally-signed SSL certs
---
Ok, perhaps this isn't a curiosity to you all.  Perhaps everyone knew this.  But I didn't, and I didn't find it in a quick Google search, so I thought I'd put it out here.    

I'm working on a project that uses 2-way (client-auth) SSL; in doing so, I've created a test root CA and used it to generate an array of test certs for clients and servers. I've done a bit of testing on my local PC, and it's working well; the server uses client certificates for authentication and authorization of web service requests, and so on. Acegi is our friend. 

However, another developer on the project tried accessing the site not long ago and told me it didn't work.  "I just get 'Cannot find server'", he said.  Turns out that he was using Internet Explorer, and I had been using Firefox. I could replicate his results with IE -- the browser would give the prompt saying "This site's certificate is not trusted" or something to that extent, and then when you said "accept the certificate", it would throw up a "Cannot find server" error.  The same series of steps in Firefox would work without a problem.    

After fiddling a bit, it turns out that adding the test CA to my Trusted Root Certificate Authorities list fixed the problem.  Honestly, I'm not sure why this would be necessary, if Internet Explorer had already asked me whether to accept the site's certificate, but my best guess is that IE was simply ignoring my answer and refusing to load the site whose certificate was signed with an unknown root CA.   

Again, maybe this is a known bug, but hey, I thought I'd throw it out there.  Adding the test CA to IE's "trusted root certificates" list fixed the problem.
