# Prodution Scale Big Data Implementation: Week 1
## Introduction
This article is the first in a series of 7 articles which document the process of setting up a analytical pipeline, with the objective of solving a common problem faced by modern security service providers. In this article we shall discuss the problem, our approach to the solution, & the tools that we will use for building the solution.

## Problem Statement
We live in a generation in which all of our devices are connected and can communicate with each other in ways that were never possible before. With such advances in technology several security service providers have come up with security cameras which are capable of calling in the property owners / emergency services in the event of a security breach. These cameras are already proving to be more than useful either in preventing robberies or in helping the law enforcement authorities catch the perpetrators. The video below is a an example provided by one such security company, **Nest**.

[![Nest Video](https://img.youtube.com/vi/CVnU1nowH0I/0.jpg)](https://www.youtube.com/watch?v=CVnU1nowH0I)

However, even with all the advantages of security cameras, they have one big problem: **False alarms**. Security cameras will detect motion from insects crawling or flying in front of the camera lenses during the day and especially at night. This can cause false positives every few seconds and annoy the property owners every time thier mobile devices goes off as a result. In some cases the false alerts could also have monetary penalties accociated with them.

It is of paramount importance for the security companies to decrease the rate of false positives. This can be achieved by improving the accuracy of thier **object identification** techniques such that the difference between intruders and domestic animals and other objects can be established programatically. 

In other words, the objective is to build a model that can distinguish between:
This:
![burglar](https://cdn.agriland.ie/uploads/2015/11/breaking-and-entering-425x235.jpg)

and this:

![dog](https://us.123rf.com/450wm/thesupe87/thesupe870805/thesupe87080500115/3098389-ritratto-di-un-giovane-tri-color-beagle-cucciolo.jpg?ver=6)

## Our Approach To The Solution: Building a Data Pipeline
While **accuracy** is clearly one important attribute of any acceptable solution to this problem, **scalability** is also as important. Security firms collect copious amounts of video data from various locations across all of their clients. With the amount of data available to them for training and the amount of data flowing-in every second to be analysed for threats, it is impractical for them to implement any solution which involves manual handling of the data. 

The ideal solution for this problem would be a framework in which all the various steps like data cleaning, manipulation, modelling, visualization, etc are setup to work automatically and in the proper order. This is what we call a **data pipeline**. The idea is that give any new data as input, it would flow through the data pipeline and automatically produce the required output; which in our case would be the classification of an object.

![datapipe](https://cdn-images-1.medium.com/max/1600/1*8-NNHZhRVb5EPHK5iin92Q.png)

The main goals of our data pipeline would be as follows:
1. Gather the image data from all the sources
2. Analyze the data and identify potential threats
3. Harmonize the data to facilitate consumption and decision making

## The Tools
Now let us a take a look at the various tools / technologies that we would be utilizing to setup our data pipeline.




