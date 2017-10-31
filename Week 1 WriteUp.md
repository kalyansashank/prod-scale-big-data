# Prodution Scale Big Data Implementation: Week 1
## Introduction
This article is the first in a series of 7 articles which document the process of setting up a analytical pipeline, with the objective of solving a common problem faced by modern security service providers. In this article we shall discuss the problem, our approach to the solution, & the tools that we will use for building the solution.

## Problem Statement
We live in a generation in which all of our devices are connected and can communicate with each other in ways that were never possible before. With such advances in technology several security service providers have come up with security cameras which are capable of calling in the property owners / emergency services in the event of a security breach. These cameras are already proving to be more than useful either in preventing robberies or in helping the law enforcement authorities catch the perpetrators. The video below is a an example provided by one such security company, **Nest**.
[![Nest Video](https://img.youtube.com/vi/CVnU1nowH0I/0.jpg)](https://www.youtube.com/watch?v=CVnU1nowH0I)

However, even with all the advantages of security cameras, they have one big problem: **False alarms**. Security cameras will detect motion from insects crawling or flying in front of the camera lenses during the day and especially at night. This can cause false positives every few seconds and annoy the property owners every time thier mobile devices goes off as a result. In some cases the false alerts could also have monetary penalties accociated with them.
It is of paramount importance for the security companies to decrease the rate of false positives. This can be achieved by improving the accuracy of thier **object identification** techniques such that the difference between intruders and domestic animals and other objects can be established programatically. 

While **accuracy** is clearly one important attribute of any acceptable solution to this problem, **scalability** is also as important.


