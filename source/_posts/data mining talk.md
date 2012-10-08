---
title: data mining notes
layout: post 
summary: a scratchpad for a presentation on data mining
published: false
---


Two lists on the board: 
1. Talk about afterward
2. Talk about later

Why? 

A firehose -- lots of different things, some (many?) of which you're guaranteed to already know.

So that list leaves me an out to not remember things: I expect a lot of things on that list. 

Leads me to my credentials: 
not much. 

What we're talking about? 
Machine learning algorithms
Data Science
Big Data Processing
things that _can_ be done on large datasets, and _are_ done in practice. 

Some assumptions: 
You have large dataset, m records long. 
In this dataset, you identify a series of features, [X] (x1, x2, x3…)
You probably have a value "y" you want to get out of this.


I did a quick, informal survey of a minority of the science team about the things they'd be interested in hearing. Among the things that came up: 
1.) Things that might have changed in the last few years
2.) Clustering
3.) Quiet, boy. I know all of these things

So I'll start with a list, the result of a poll of popular algorithms among people calling themselves "data miners" in 2011: 

# most popular algorithms among data miners, 2011: 
* Decision Trees/Rules (186) 	59.8 %
* Regression (180) 	57.9 %
* Clustering (163) 	52.4 %
* Statistics (descriptive) (149) 	47.9 %
* Visualization (119) 	38.3 %
* Time series/Sequence analysis (92) 	29.6 %
* Support Vector (SVM) (89) 	28.6 %
* Association rules (89) 	28.6 %
* Ensemble methods (88) 	28.3 %
* Text Mining (86) 	27.7 %
* Neural Nets (84) 	27.0 %
* Boosting (73) 	23.5 %
* Bayesian (68) 	21.9 %
* Bagging (63) 	20.3 %
* Factor Analysis (58) 	18.7 %
* Anomaly/Deviation detection (51) 	16.4 %
* Social Network Analysis (44) 	14.2 %
* Survival Analysis (29) 	9.32 %
* Genetic algorithms (29) 	9.32 %
* Uplift modeling (15) 	4.82 %
((KDNuggets)[http://www.kdnuggets.com/polls/2011/algorithms-analytics-data-mining.html])

"We grouped Industry/Gov in one group and Academic researchers/Students into a second group, and computed the "affinity" of the algorithm to Industry/Gov as

    N(Alg,Ind_Gov) / N(Alg,Aca_Stu)
    ----------------------------------
    N(Ind_Gov) / N(Aca_Stu) 

Thus algorithm with affinity 1.5 is used 50% more in Industry/Government than by Academic Researchers or students, and the algorithm with affinity 0.6 is used only 60% as much in Industry.

The most "industrial" algorithms ( with the highest Industry / Gov "affinity") are:

    Uplift modeling, INF (no academic users)
    Survival Analysis, 2.47
    Regression, 2.00 

The most "academic" algorithms ( with the lowest Industry / Gov "affinity") are:

    Genetic algorithms, 0.60
    Support Vector (SVM), 0.66
    Association Rules, 0.83 


I'm going to start this brief tour with the second item on the list -- regressions. I think this is fairly well-known by everyone on the call, but for the sake of those that aren't familiar, here's the problem. 

Common example, but one near to my heart at the moment: 
House price graph. 
Want to draw a line through it?  Great, here.
The line can be expressed as y = ax + b…the algorithm's job is to find "a" and "b". 

About regression
 - starts out as a linear regression
 - want to make it non-linear?  Great, add polynomial features. 
 - been around forever. 
 - of course, super-common

Seasonality curves might be a linear regression. But wait, it's not linear? Add some polynomial features. 

But wait -- that's a lot of features!  That's ok. Well, it starts out ok. We can handle thousands of features no problem… it can be represented as matrix math. 
That can be represented as ThetaTx. Matrix operation. Can be handled much easier than it used to. 
How much easier? 
A password that can be cracked in about four days to crack on a well-spec'd Intel system can be cracked in just over an hour on a fast graphics card. Like the ones that Amazon rents on EC2 instances for $4/hour. 
(Change your passwords.)

But what if you had a LOT of features: say, tens of thousands. And let's say you know your answer is highly non-linear. So you'll want polynomial terms… but if you add all combinations of second- and third-order polynomial terms, you probably want to look at something like a neural networks. 

Really popular in AI in the 80's and 90's. Still really popular in image processing, video processing, text processing… 
Image processing example. 
Neural nets: interesting to us?  We tried them, of course, in the early days.  

Classifying: logistic regression is an option
 - you can do multi-class by doing a series of "is it Class X or Not? Is it Class Y or not?" and take the higher of those classes. 


or SVM (Support Vector Machine) 
 - algorithm starts out quite similar to logistic regression: "is this new feature more likely to be a 1 or a 0"? A couple of differences: 
     - because of how the algorithm works, it handles margin better (it's referred to as a "large margin classifier"): that is, it will pick the line with the largest margin between classes. 
     - when used along with kernels, it can do some very complex nonlinear classification. 
     - "SVM + kernels" = add new features defined as "Similarity to landmark L", where in practice landmarks are added for each training set. You can actually replace your old features. So # of features = # of training examples (!). 
     		- with an optimized matrix library, this isn't as bad as you'd think. 
     - you don't write this yourself. You use a library that does it for you.  Much like Neural Nets…there's no point. It's like writing your own square root or matrix inversion. 
 
Netflix prize: the prize was to beat Netflix's current rating system by 10%.  

The first 5% was by classifying each movie and each user using SVDs. Classify each movie into, say, 40 buckets, and then classify each user based on how they rate movies in each bucket. 

ratingsMatrix[user][movie] = sum (userFeature[f][user] * movieFeature[f][movie]) for f from 1 to 40

(from http://sifter.org/~simon/journal/20061211.html)
The other 5% was from applying many different models and averaging them together. Things like the day of the week the user watched the movie, how long ago they watched it, etc.
The winning entry ended up using over 800 different algorithms, averaged together. The description takes 150 pages. 

The most interesting thing to me about reading about the Netflix prize, actually, is about the trial-and-error process that people followed. They had a huge dump of 100 million movie-user-rating triplets, making an 8.5 billion cell sparse matrix, and they just try things, over and over, and see what it does to the final rating. They typically partition their data set into three categories: "training", "v" and "test", and then just keep trying things and seeing what increases their test score.   



Where to Use Neural Networks
----------------------------
Neural networks are used in a wide variety of applications.  They have been used in all facets of business from detecting  the fraudulent use of credit cards and credit risk prediction to increasing the hit rate of targeted mailings.  They also have a long history of application in other areas such as the military for the automated driving of an unmanned vehicle at 30 miles per hour on paved roads to biological simulations such as learning the correct pronunciation of English words from written text.

Decision Trees
--------------

Not a real strength of mine. But we do use it, somewhat: Shadow could be considered a decision tree leading to a set of models. Actual uses it in, say, deciding whether to update RC_AVG_DEMAND. 
