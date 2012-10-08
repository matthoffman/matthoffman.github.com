---
title: In Defense of Easy
layout: post
summary: where I attempt to apply Rich Hickey's "Simple Made Easy" to a current case study
comments: true
---

the setup
---------

I watched Rich Hickey's talk from Strange Loop called ["Simple Made Easy"](http://www.infoq.com/presentations/Simple-Made-Easy), which has been making the rounds recently[^1]. He's a great speaker, and always makes interesting points. But something in that talk didn't sit right with me[^2]. While his slide about Development Speed ("emphasizing ease gives early speed...ignoring complexity will slow you down over the long haul") really struck a chord[^3], it felt like he was shortchanging "easy" in favor of "simple"[^4].  This, then, is my long, somewhat circuitous exposition on that thought.

[^1]: I started writing this in late 2011, when it was in fact "recently". It's now old news, and firmly in the canon of tech talks to watch. Since then, he's given another wildly popular talk at RailsConf called ["Simplicity Matters"](http://www.youtube.com/watch?v=rI8tNMsozo0).  I won't be referencing it here, but it's worth your time as well.
[^2]: I'm well aware that, when I disagree with Rich Hickey, the chances of me being right aren't very good. 
[^3]: Yes, a chord. Which, being two or more interwoven notes, is complex. But that's ok.
[^4]: Perhaps someday, I'll distill it down into a shorter, clearer message. Maybe even a couple dozen powerpoint slides. And maybe then, someone else will point out that the broad brush I used for brevity covered over some other subtlety that they found very important...

the case study 
--------------

I'm working on software that I've been involved with for several years. Like a lot of projects, its evolved as the company's (and their clients') needs have changed, and it's been written under schedule pressures and ever-shifting requirements – in other words, it's like most software, as far as I can tell. I dream some days of VC-funded startups in their Herman Miller chairs building beautiful software from scratch with plenty of cash and plenty of time, but I suspect the grass isn't really so green over there. 

The application has gotten to the age where the layers of rapid development are starting to take their toll: new developers coming onto the project complain that the software is just too complex. By 'complex', they mean, "it's hard to wrap my brain around all of the moving parts." Some of the symptoms of this version of 'complexity' include: 

  * it takes a very long time to get new developers up to speed
  * there are a fair amount of bugs caused by changes made to one part of the application which had downstream impacts on other parts of the application. Since most developers don't have the "big picture" in mind, they don't know they're breaking things when they make changes.
  * writing new features takes a long time, since the developer needs to know a lot, juggle a lot, and dodge all kinds of pitfalls in order to put something new in place.

When I interviewed developers about where the problems were, I got answers like this: 

* You have to keep too many things in mind at any one time. 
* what different parts of the application are doing, upstream and downstream of the piece you're looking at right now – how all the pieces fit together.
* details of the domain: you have to have some familiarity of what the end goal is in order to understand what things are doing.
* terminology: there's a lot of terminology that is domain-specific, but it also isn't entirely consistent throughout the application.
* the pattern at work in the part of the application you're looking at: different parts of the application follow different general patterns to do what they do.
* the version of the code you're looking at:  since there are several clients using different versions of the application, some several years old, even if you've gotten your head around a section of code in one codebase, there is no guarantee that the accomplishment will translate to another client.

But, is simplicity the root issue? How do we fix it? And how would we prevent it in the first place?

theoretical aside 
-----------------

Here's my limited understanding of educational psychology:  
We're built to only handle a small number of things in working memory (7 things, on average – like a phone number[^5]). To understand more complicated things, we have to either "swap out" to longer-term memory or synthesize several related things into one well-understood thing. 

[^5]: The famous ["Magical Number 7, Plus or Minus Two"](http://en.wikipedia.org/wiki/The_Magical_Number_Seven,_Plus_or_Minus_Two)

Swapping out takes a long time, and is tiring...how many of us have been reading through technical documentation and gotten to a point where we said, "Oh, wait... the author explained what that term meant, or what that component did...what was it again?"  That's the effect of running out of working memory and having to look in long-term memory to find something you recently read. But more often than not, that factoid probably hadn't been committed to long-term memory yet, so we had to go back and re-read part of the document.
Synthesizing is the process whereby you understand a group of related things as a whole, so that you're able to think of the whole as one unit, rather than the component parts. This takes time: you have to "wear grooves" in your brain in an identifiable pattern. For example, when we see a face, we don't see individual facial features – ears, a nose, a mouth, and eyes – and then try to juggle all of these features to understand whose face it is. We recognize a face as an identifiable whole. Through many repeated exposures, our brains have synthesized that pattern as "a face", and deal with it as a single item in our memory. That's also why elementary school teachers drill math facts – if you have internalized that 8 x 7 = 56, you don't need to store 8 and 7 in working memory and perform the arithmetic..."8x7=56" is a discrete whole in your brain.

For folks learning a new piece of software, especially a large software system, there's a lot of individual pieces that have not yet been synthesized into understandable, well-understood wholes. And for many large software systems – mine included – getting to that point of familiarity with is more difficult than it needs to be. 


definitions
-----------

Now that I've butchered psychology and complexity science, I'll move on to butcher Rich Hickey's talk. 
In case you haven't watched it, here's the rough paraphrase of the definitions that he bases his thesis around:  
* "complex" means combining two or more things ("complecting"). 
* "simple" means "concerned with only one thing". 
* "easy" means "lie near" or "close at hand".  Relative to the speaker. 
* Conversely, "hard" means "(conceptually) distant".

I think it's fair to say that Rich felt simplicity is the important metric, while "ease" is superficial. 

Meanwhile, it's worth noting that [Merriam-Webster's definition of "complex"](http://www.merriam-webster.com/dictionary/complex) is, in part: 

> 1: a whole made up of complicated or interrelated parts <a complex of welfare programs> <the military-industrial complex>
> 2c : a group of obviously related units of which the degree and nature of the relationship is imperfectly known

(some unrelated definitions omitted)

complex or hard?
----------------

Here's where things get dicey. It seems clear that making things simpler, in Rich Hickey's sense, would be a real benefit. In this particular application, there are a lot of places that are combining "what to do" and "how to do it". There are optimizations that have led to I/O-related code mixed into business logic, for example. And the level of abstraction is generally too high; our business logic has to concern itself with a lot of plumbing around "when" and "where", instead of focusing on "what". If each component focused only on the "what" of one portion of the business logic, it would certainly be easier to understand. 

But it also seems clear that many of the developers' complaints – "we can't keep it all in our heads at once", "we don't understand how the pieces fit together", and so on – aren't really about "complexity" in Rich Hickey's sense. They are in Merriam-Webster's sense: "a group of obviously related units of which the degree and nature of the relationship is _*imperfectly known*_". But by Rich Hickey's definition, being hard to conceptualize is "hard", not "complex". 

Perhaps there's a broader way that the "single purpose" definition of simplicity could be applied to the application as a whole that I haven't thought of. But at the micro level, lots of very distinct pieces with a single concern don't necessarily make it easier to understand the big picture. In some cases, his suggestions may make that worse: a rules engine, for example. 

Having important information ("should we follow path X or Y?") abstracted out to a rules engine is, in Hickey's terms, _hard_: it makes important parts of the application no longer near the code itself.  So the code determines what, the rules engine determines why (or is it when? not sure) by, i assume, composing the "whats" into a control flow.  And then when you're trying to debug something, you have to go back and forth between the two.  That's simple, in that the two are separated. But the conceptual "surfaec area", if you will, is much larger. You have to hold a lot in your head to understand the behavior of the application.

Perhaps generally, simplicity in this "single-purpose" sense help greatly when understanding the individual component, but make understanding the application as a whole _harder_ (using, again, Rich Hickey's definition of _hard_). But _easy_ and _hard_ don't matter, right? They're relative to the particular developer, they're a matter of convenience...right? 

Perhaps I am reading more into that dichotomy than I should. Casting things in black and white is a perfectly reasonable rhetorical device – especially in a fairly short talk – and that's what Rich Hickey has done here. Exploring all of the subtleties distracts from the thrust of the talk, and simply doesn't fit into that format. But I just want to explore what those subtleties are. Because in the case we're looking at here, "easy" matters if it's making things hard enough to understand that you have to work on the application for years before understanding enough to add new features. And they certainly matter when there's only a couple developers who can get paged at 3 AM when things go wrong. 

There are real advantages of "easy", where "easy" is defined as "near to our understanding/skill set". Less conceptual weight means more working memory can be applied to the problem at hand. Less conceptual weight means less effort expended on the tool itself, more effort to expend on the problem at hand. That is inherently pleasant for engineers -- it is very frustrating to put a lot of effort into the tool instead of the problem.  On the flip side, once a tool is mastered, it is very, very pleasant to be able to whip through a problem efficiently: see Vi vs. Notepad.

The argument is similar to the argument that Scala proponents make[^6]: eventually, the concepts that are currently very conceptually weighty (_so_ many things to keep in mind!) will eventually be internalized and synthesized, and then it is better, because you can attack your problems better. The question is, is the frustration of the synthesis time (the "learning curve") worth it?

[^6]: I like Scala, for the record. 

So perhaps my argument is one of degree. When understanding the application becomes hard enough that the ramp-up time for new developers becomes untenable for the developers themselves – not just the managers that want an easily-interchangable workforce – then steps need to be taken to make things _easier_. 


fix it! fix it!
---------------

So, if hard matters, what can we do about it?   

In this particular case study, there are several things that would help, and everyone reading this far has thought of several more. In fact, I'd be willing to bet that everyone reading said, "Well, clearly, what you should be doing is ----", where each reader will fill in that blank a different way. That's because the world of Software Engineering has come up with all kinds of ways to make large systems easier to deal with. Here's a few: 


### visualizing the abstractions ###

Visualizing the various related portions together saves you from having to hold parts in memory as you look at other parts – it's all available right in front of you. Seeing how things work together also helps you synthesize them into a whole. To get a clear picture by looking at the code, you have to have internalized what each of the components do and how they connect. And since there's far more than 7 components in those networks, they don't all fit in working memory. So until you've internalized and synthesized the components and how they interact, it just feels overwhelming. 

Visualizing the flow of data through the system is a similar problem. We have sequences of jobs that read various values from the database and then insert or update other values.  There's no way to visualize at a high level what components set data and what components are reading that data. 

When reading through the code, I'm essentially trying to construct a mental picture of how it's all fitting together. Add in any abstraction layer, like a dependency injection framework, and things get even harder: "what components do" and "how the components fit together" are defined in two different places (Java files and Spring context xml files, for example, for the Spring framework). If I could somehow visualize it all on one screen, it would save me the effort of constructing a mental picture... ideally I would be able to see both what components do (popup or mouseover descriptions?) and how they fit together. That would be a lot less to juggle in working memory.

### contracts ###

In cases where the pieces of the application need to fit together but developers aren't likely to hold all of those pieces in memory, defined contracts are an age-old solution. The theory is, you don't _have_ to hold it all in memory; just the compoent you're working on, up to and including the contracts it has with neighboring components. How they fulfill their contracts is outside your concern. 

Whatever form those contracts take, you need to be able to explicitly specify (in an automatically-verified way) what each component expects from others.

In the software in question, there _are_ contracts in place between components...why those aren't making things simpler for developers is a subject for another blog post.

### consistency ###

If everything followed the same pattern, there you would only have to learn that pattern once. Then hopefully, like a face, every new job you saw would just look like a different but recognizable instance of the same pattern. 

Consistency also helps in tooling – if everything fits the same pattern, you could create visualizations that worked for every use. 

Of course, you can only be as consistent as your actual patterns of use. In this case study, there's not yet a single pattern that meets the needs of all parts of the system. Some are iterating over data elements and performing relatively simple calculations on each, while others are constructing huge in-memory graphs of time-phased data and using those for some rather complicated calculations. With some more thought, though... 

Another form of consistency is a universal vocabulary: if everything used the same term for the same concept, you'd only have to learn the definition once. Terminology was specifically called out as a barrier to understanding. ["Domain-Driven Design"](http://en.wikipedia.org/wiki/Domain-driven_design) practitioners stress a consistent "ubiquitous language" for this reason. Honestly, I'm not sure how big of an issue this ends up being; it is a huge early barrier for new hires, but one that fades pretty quickly.

### comprehensive integration test coverage ### 

This doesn't really help in understanding the system, but does give developers some level of confidence that, while they don't understand the whole system, they'll know if their change has broken it. Depending on the tests, they can potentially serve as a form of documentation. And of course it has other benefits as well – like, having components that are tested. 

And of course, the classic 

### documentation ###
As a last resort, documenting preconditions and postconditions, defining terms, explaining patterns and paradigms in static documentation can help.

Documentation is a distant last place. At best it can help you internalize some piece of the application, but in practice it will just presenting data that you then have to hold in working memory while you try to reason about the system. You as the reader then have to go through the effort of reading and reviewing it in order to try to commit it into long-term memory. The more complicated the component or concept is, the less useful documentation is.  

Documentation is also out-of-date as soon as it is written, it's never maintained, has to be discoverable for the person trying to learn about things, and is high-effort for both the writer and reader. I secretly suspect that, if we could track the number of times that any given page on our internal wiki has been really read, the average across all pages would be < 1. We write more documentation than we read.

There are many, many more. Feel free to tell me your "obviously, you should be doing ---!" in the comments. 

finally, a conclusion
----------------

I think Rich Hickey's division between "simple" and "easy" is useful and instructive...but I think that many of the things that we care about in designing simpler software that falls into what he would call "easy", rather than "simple". And if you have a nontrivial number of components, with interactions that will not easily fit within a developer's working memory until they've done a significant amount of synthesis... don't discount easy. It can save you from that 3AM page. 




