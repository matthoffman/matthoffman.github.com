---
layout: post
title: "Refactoring and Technical Debt (or, legacy apps have all the fun)"
date: 2013-10-08 01:29
summary: "where I try to rationalize continuous refactoring and large, unweildy applications"
comments: true
categories: 
---

There's a lot of great ideas under the "agile development" umbrella. It now encompasses ideas pulled from older software engineering practices, manufacturing techniques, behavioral science, and goodness knows what else. But any method emphasizes some things at the expense of others, and in my experience a lot of projects have real problems with how to handle significant technical changes -- whether they're called refactoring, or redesign, or technical features -- within the context of an agile practice. 

I'm not going to solve that here, but I'm going to describe what I've seen in case others have answers.

First, for background, typical agile development methods say this about refactoring:

1. It should be done continuously.
2. It shouldn't be called out as its own user story; it should either be done as part of an existing user story, or baked into the "slack" time of an iteration. As you work on any feature, you should clean up the code where you are. "Leave it better than you found it."
3. Meanwhile, don't design anything too far in advance (YAGNI), and let them instead emerge over time.

I don't disagree with that. But I think don't think it always works quite so neatly. To break it down, let's classify the "technical work" into three buckets[^1]:

[^1]: I'm using the term "technical" here to contrast it with "functional," meaning something that the user will explicitly ask for.

1. Simple, localized refactors. 
2. Larger, system-wide refactors. 
3. Non-functional features. 

For #1 (Simple, localized refactors), I think everyone agrees. Good development practice dictates that you leave the code better than you found it. If you're working in some code that needs documentation, or has some complex logic should be broken down into easier-to-understand methods, or there's some code duplication, clean it up.  There's no need for a separate user story or ticket; that's just run-of-the-mill as-you-go refactoring. No arguments there, right? 

But there are cases that aren't so simple, and those are the cases I've seen projects have problems with. 

## larger, cross-feature refactors. ## 

These are the refactors that are bigger.  Perhaps you have a repeated data-access pattern that could be abstracted away, or you have three different patterns to do the same basic thing, and they need to be merged into one. This is the kind of thing that NEEDS to be done on a large codebase as time goes by[^2], or it becomes so unweildy that your maintenance time completely overshadows your development time, your new features take months when they used to take weeks, and ramp-up time for new developers is longer their average life expectency.

[^2]: Perhaps this doesn't happen on all projects. It could be that there are projects out there where these refactors are always be caught early enough that they're still fairly small, and so are always fixed as part of run-of-the-mill ongoing refactoring. I'm not sure if that's true or not, but I am sure that if you ever walk into an app that is large enough (say, a hundred thousand lines of code and up), there will be cases like this that need to be fixed. I suspect that as an application reaches a size where a single developer is no longer intimately familiar with the entire thing, you're going to have duplication, repetition, and leaky abstractions. I'd love to think that the next app that I grow from the ground up will be diligently refactored intelligently enoguh and often enough that there will be no latent patterns that need extracting or larger refactorings or reorganizations that need to be done... but at the same time I won't put money on it.

As an example, I worked on a project a few years back that had started out using Hibernate as its persistence layer. Hibernate was no longer working for us, for a number of reasons, so it was time to ditch it and find something else. To make matters worse, there were a few different attempts at non-Hibernate (or half-Hibernate) DAOs already in the codebase, but each worked differently and handled only a subset of our use cases. We needed to come up with a better solution, and ideally refactor old uses so that our data access patterns were consistent. That was going to take some time.

A lot of people would argue that this is no longer really "refactoring" as much as building a new feature -- just an internal, developer-facing feature. I think you could argue it either way, but they each present some challenges for agile shops.

When they are needed, these refactors often fall into a gray area -- not as huge as "we need to rewrite the application", but not as small as "this interface needs some documentation." This is the gray area that could, potentially, be done as part of a client-facing feature, but in doing so it will probably make that feature take drastically longer.

In this scenario, the iteration planning or grooming conversation might look like this: 

> **Scrum Master (SM)**: "Ok, what tasks should we create for Story X?"
>
> **Dev**: "To start with, we need to refactor Interface A, and change all users of it to use a new paradigm."
>
> **SM**: "Does that really need to get done for Story X? It seems unrelated."
>
> **Dev**: "It isn't strictly necessary to implement the story, but that interface has turned out to be a leaky abstraction that has made all users of it more complicated as a result. We didn't realize it would be such a problem when we wrote it; but now it just needs to be refactored. And since this story involves that same interface, now is the time."
>
>**SM**: "How long with that take?"
>
>**Dev**: "Probably 20 hours, total."
>
>  (they then proceed to break it down into smaller tasks)
>

...now things could go one of a few ways. In an ideal agile scenario, where timelines are entirely flexible and there is complete trust between product owners and developers, the rest of the conversation is:

> **SM**: Ok. 

But most scenarios aren't ideal. And many (most?) timelines aren't really entirely flexible, nor relationships as entirely trusting. So the rest of the conversation might be:

> **SM/Product Owner/someone with a stake in the timeline**: "20 hours? Um... we don't have time for that right now. We need to just do what's necessary for Story X and refactor that interface when we have more time."

Unfortunately, when that happens, developers learn fairly quickly to work around it. I've seen several teams where unfortunately the conversation starts playing out like this instead:

> **Scrum Master (SM)**: "Ok, what tasks should we create for Story X?"
> **Dev**: "To start with, we need to refactor Interface A, and change all users of it to use a new paradigm." 
> **SM**: "Does that really need to get done for Story X? It seems unrelated." 
> **Dev (lying)**:  "Yes, it absolutely needs to get done for Story X.  And I estimate it will take 20 hours." 

We call this "sneaking in technical stories", where we get things done under the disguise of sometimes tangentially related user stories.
It's "sneaking in" because, in reality, these improvements -- whether refactors or new technical features -- are rarely _necessary_ to complete the features. That's the nature of technical debt: You can pay off the collateral, or you can keep paying interest on the debt. It is always cheaper, on any given story, to pay the interest instead of paying down the whole collateral.  Paying down the whole collateral, in some cases, can take a significant amount of time -- often the cost of paying down the debt dwarfs the cost of the actual story you're estimating. It starts sounding very artificial and "sneaky" when a story should take 2 days, but in order to pay down technical debt you estimate it at 21. 

I know that there are teams where the relationship between the developers and the product owner is good enough, and there is enough trust built up that when a developer says, "I really think we need to clean this up, but it will take an extra week," the product owner says, "Ok, you know the timelines, I trust you to get it done." But in the typical Scrum team, the odds are stacked against that relationship. A typical product owner is non-technical, so they see only features and timelines and find technical debt a fairly abstract concept. So given the choice between having a feature completed in 4 hours or 20 hours, with the only difference being the amount of technical debt removed from the codebase, the product owner, seeing a very real schedule constraint, will pick the 4 hour option, 9 times out of 10. Often they'll try to strike a very rational bargain: "Let's do the 4 hour option now, and we'll try to get to the 21 hour option if we have extra time at the end of the release." Then a separate technical story is created ("clean up interface A") and it falls into the backlog, never to rise to the top. Knowing that this is going to happen, devs learn to lie and say it's "absolutely necessary" in order to get it into the sprint. Or worse, they might just pad estimates with the debt in mind, and not add a task for it at all.

You could argue that saying "yes, this is absolutely required to complete Story X" isn't that bad. The development team is responsible for the quality of the code, so if they feel it improves the quality of the code, it's required. And perhaps it is a lesser evil, but I think it is problematic. One particularly nasty side-effect that I've seen is that, since developers feel like they're doing the work "under the radar", they end up doing it as quickly as possible -- perhaps including shortcuts, or "I'll have to deal with this part of the problem later". They tend to want to understate the time they're putting into it in standups. They might not document the new interface fully, or advertise it broadly, since it is being done undercover. It leads to sloppy work.
 
None of this is to villify product owners: the product owner doesn't know the difference between a "nice to have" and "must have", and devs can't quantify the cost of paying the interest on techical debt vs. paying down the debt itself. So in my experience product owners will often entertain a few of these instances early in a release cycle, but then once a couple stories get ballooned up from 4 hours to 20 hours because of technical work, they loose their appetite. And to their credit, developers can _always_ find more things that need to be refactored and cleaned up. So midway through every release cycle, the mandate comes down: "We just need to get the features done, we are on a tight timeline, no more technical stories." No matter how closely I've worked with product owners, or how good their relationship with development is, it seems to end up in a similar place.

Larger refactors are even worse. There's a class of technical work that is too large to conceivably fit within a functional story. Perhaps you foresee performance issues and you need to change your persistence strategy. That's going to take R&D, and a fair amount of rework, and you'd like to do that before it's a critical burning issue.  

Or perhaps you've put off doing those medium-sized refactorings for long enough that you've got a pile of leaky abstractions or similar-but-not-quite-identical patterns that need to be unified, and it's no longer a convenient sprint-sized task.

Now it has to be called out explicitly, and that relationship between the dev team and the product owners is tested further. I've seen very few nontechnical product owners who will prioritize this type of work into a sprint.

Then there's non-functional features. These are standalone features, like a console for maintenance, which are not refactoring or technical debt, but are features with benefits to development or operations -- someone other than the canonical business user that the product owner represents. Perhaps the tools will speed debugging, or make operations easier, or whatever, but they're not specifically called out in a client SOW.

This has the same problem: with one team, and a nontechnical product owner, it's hard to figure out how to put these into sprints. Because at the end of the day, "business value" is usually defined as "value delivered to clients," where  "clients" is always defined as "the set of business users that the product owner is most familiar with". So you stack your releases with as many high-value user stories as you can -- always a subset of what the client wants. 

And again, I don't say that to vilify product owners at all. It puts them in a tough spot.
Imagine you're a product owner:
You have a story in front of you for a proof of concept for restructuring the data model of the app. It will take a week to prove the concept, and after that an indeterminate amount of time to implement for real, but it may give you a significant benefit in speed of development or speed of onboarding or performance some time later.  You also have 20 client-facing features that you'd like to get into this next release. A product owner will rarely, in good conscience, put that technical story over the user stories that need to get into this release.  If you're in the unenviable task of fitting an agile process into a SOW-driven sales process, then even less so. So that proof-of-concept story will never get into a sprint. They're not strictly necessary for a release, clients X and Y need features A through G in this release, so we'll get to those "in the next release." 

There's just no way for a product owner to prioritize possible future benefit over certain immediate benefit, even if the possible future benefit is significantly higher.  All the cards are stacked against the technical stories: they have no deadline, they are not in an SOW, and they are intangible and uncertain. It's not product owners' fault -- that's just the way the system is structured. If they don't get features A...G into a release, the client is unhappy and it's their fault. But if they don't get technical stories Q..Z into the sprint...well, we've lost a possible future benefit, but perhaps we can get to it in the next release. Since technical stories don't have a deadline, we can always push them into the next release. Features, on the other hand, often have a deadline -- in practice, there's often agreements with clients, or roadmaps (however rough) posted in advance, or some other way to let users and potential users know what is coming. Given those two choices, I know which one I would pick, too. ALL the pressure is on getting the features done by the release date, for THIS release...not making future releases faster or easier.  

Of course we'd hope that, somehow, the possible value of a technical story would outweigh the certain value of a feature story, eventually, if the possible value of the technical story was high enough. But I don't think the system is set up to make that happen. I've seen high-value technical stories that have remained in various backlogs for years on end. 

So while I understand that in an ideal world, you either express the value of large refactors or technical features and let the product owners prioritize them, or else let the technical team decide that it's time to take a couple sprints off to tackle a large refactor while the product owner looks on obligingly. But that seems to be the exception rather than the rule.

There are alternatives, though. Some companies use 20% time as an alternate way of scheduling in this kind of work. Some use a flat ratio of technical to functional work. Some use a dedicated team that handles "framework" or "tools"-related work to remove at least that portion of the technical backlog. 

If you're in a situation where you hear things like, "in the next release, we'll bite off a more reasonable timeline and prioritize these things in," look a little closer: you might have this kind of systemic problem where technical work really can't get prioritized in. Perhaps it's time to look for other options. 

