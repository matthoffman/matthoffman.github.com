---
title: Greasemonkey is Underrated
layout: post 
summary: where I wax on about Greasemonkey, a firefox extension for decorating web pages
---

I've heard about Greasemonkey a few times over the past few years.  I remember reading a few people wax eloquent about it, mainly how great it is for changing the layout of web pages (like a more convenient [Youtube page](http://userscripts.org/scripts/show/101753), or even changing [Google News back to its old layout](http://userscripts.org/scripts/show/80853)). 
And I remember the [big security kerflaffle](http://ejohn.org/blog/greasemonkey-security/) a number of years ago (wow, 2005 – has it been that long?). 
But I'd never had occasion to use it. 

But when I started browsing real estate listings in earnest, I got my excuse. I like [Zillow](www.zillow.com) and [Trulia](www.trulia.com), but the realtor I am working with has a website of his own that he prefers – clients can save listings as "favorites" and "possibilities", make notes, and he can make notes or suggestions in return...which makes good sense. But the site is no Zillow or Trulia: it displays far less data, it's a bit slow, and it's fixed at 10 results per page. And when there's over 100 listing my price range to sort through, every little bit helps. 

Greasemonkey can't help the site speed, but I've made some inroads on how much data it displays. 
I hacked up a little Greaseonkey script that runs in Firefox everytime I load the realtor's search page. The script just goes through each listing and fires off a query to the Zillow API, returning Zillow's value estimate, the historical min and max value of the house, and a direct link to the listing's page on Zillow. I may add a couple other things, like tax value and last sold date (is it a flip?). I was excited about adding some neighborhood statistics, too, but unfortunately Zillow doesn't have neighborhood data for my area.   

Up next: look into the Trulia's API (I haven't checked if they have neighborhood information) and maybe Walkscore, Panoramio and Flickr (for neighborhood pictures). 
After that, some Google Map hacks...but that's a bit more involved.

If anyone else is considering pimping their webpages – I highly recommend it – I recommend Mark Pilgrim's [Greasemonkey Hacks](http://commons.oreilly.com/wiki/index.php/Greasemonkey_Hacks/Getting_Started). I heard some disparaging comments about the docs for Greasemonkey, but that page is great. Several of the other docs on [Greasespot's wiki]("http://wiki.greasespot.net/Main_Page}) are worth reading as well. Those of us that are only passable Javascript programmers can be confused by Greasemonkey's weird sandbox environment, so the documentation is necessary (objects in Greasemonkey are all wrapped with XPCNativeWrappers for security, in response to the security issues from 2005...). I don't know if it's mentioned in the documentation, but the newest Greasemonkey version has a "create a new script" menu action that makes getting started dead easy. 

As an aside, I have to say that after taking the Stanford's online [Machine Learning class](ml-class.org) this past 
fall, I was excited about doing some data science to help in the real estate search ("how does the addition of light rail affect nearby housing prices?"). Predicting home prices was actually one of the driving examples through the class. But I've been disappointed in how difficult it is to be to get access to large sets of real estate data in the US.  I guess I naïvely thought that since tax valuations and home sales are public record, there'd be a way to access a sizeable dataset. If it's out there, I haven't found it. And the listings themselves are almost all closed, living inside each area's version of the MLS. The closest thing I've found are the API's offered by Zillow and Trulia, but at least Zillow's terms of service prevent you from storing their service responses locally, and they don't return MLS results (i.e. most results added by real estate agents) in their search results via the API...you have to ask for the properties individually. 

So I'll have to play around with big data in other ways... 

