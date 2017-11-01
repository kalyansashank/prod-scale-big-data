# Production Scale Big Data Implementation: Week 1
## Introduction
This article is the first in a series of 7 articles which document the process of setting up a analytical pipeline, with the objective of solving a common problem faced by modern security service providers. In this article we shall discuss the problem, our approach to the solution, & the tools that we will use for building the solution.

## Problem Statement
We live in a generation in which all our devices are connected and can communicate with each other in ways that were never possible before. With such advances in technology several security service providers have come up with security cameras which can call in the property owners / emergency services in the event of a security breach. These cameras are already proving to be more than useful either in preventing robberies or in helping the law enforcement authorities catch the perpetrators. The video below is an example provided by one such security company, **Nest**.

[![Nest Video](https://img.youtube.com/vi/CVnU1nowH0I/0.jpg)](https://www.youtube.com/watch?v=CVnU1nowH0I)

However, even with all the advantages of security cameras, they have one big problem: **False alarms**. Security cameras will detect motion from insects crawling or flying in front of the camera lenses during the day and especially at night. This can cause false positives every few seconds and annoy the property owners every time their mobile devices go off as a result. In some cases, the false alerts could also have monetary penalties associated with them.

It is of paramount importance for the security companies to decrease the rate of false positives. This can be achieved by improving the accuracy of their **object identification** techniques such that the difference between intruders and domestic animals and other objects can be established programmatically. 

In other words, the objective is to build a model that can distinguish between:
This:
![burglar](https://cdn.agriland.ie/uploads/2015/11/breaking-and-entering-425x235.jpg)

and this:

![dog](https://us.123rf.com/450wm/thesupe87/thesupe870805/thesupe87080500115/3098389-ritratto-di-un-giovane-tri-color-beagle-cucciolo.jpg?ver=6)

## Our Approach to The Solution: Building a Data Pipeline
While **accuracy** is clearly one important attribute of any acceptable solution to this problem, **scalability** is also as important. Security firms collect copious amounts of video data from various locations across all their clients. With the amount of data available to them for training and the amount of data flowing-in every second to be analyzed for threats, it is impractical for them to implement any solution which involves manual handling of the data. 

The ideal solution for this problem would be a framework in which all the various steps like data cleaning, manipulation, modelling, visualization, etc. are setup to work automatically and in the proper order. This is what we call a **data pipeline**. The idea is that give any new data as input, it would flow through the data pipeline and automatically produce the required output; which in our case would be the classification of an object.

![datapipe](https://cdn-images-1.medium.com/max/1600/1*8-NNHZhRVb5EPHK5iin92Q.png)

Given our problem statement, main goals of our data pipeline would be as follows:
1. Gather the image data from all the sources
2. Analyze the data and identify potential threats
3. Harmonize the data to facilitate consumption and decision making

## The Tools
Now let us look at the various tools / technologies that we would be utilizing to setup our data pipeline. In the interest of meeting our objectives of accuracy, scalability, & reproducibility, we make sure that the technology that we use to build the data pipeline is:

- **Open Source:** This is an essential requirement to make our data pipeline reproducible
- **Cloud Native:** This means that our tools allow us to build all the elements of the data pipeline in the cloud infrastructure itself and this property is crucial for scalability of the pipeline

The major elements of the data pipeline are:
1. Data Storage
2. Data Pipelining and Scheduling
3. Analysis and model building

Each element requires a technology that is best suited for the task. Below are the tools that we choose to work with in our data pipeline.

### Technologies chosen for Data Storage
#### Object Storage
Object storage is a data storage architecture which treats data as objects and not as a file hierarchy (File storage) or well defined spaces in a table structure(block storage). The image below explains the differences between these three storage systems.

![storagesystems](https://www.emc.com/content/dam/uwaem/production-design-assets/en/sdsaemmodule/images/ecs-object-storage-the-3-pillars-of-modern-enterprise-data-storage.gif)

The major advantages of object storage over the other kinds of storage are:
- Scalability - The ability to increase amount of storage available as and when required
- Flexibility - The ability to store any form of data without being limited by the limitations of table or folder structures
- Low Cost

There are a variety of object storage services available from major technology companies like S3 by Amazon, Azure Blob Storage by Microsoft, & Google's Cloud Storage on which we could store our data. 

#### Pachyderm
We would use Pachyderm as an additional layer on top of the object storage to keep track of the changes in data and manage data security. Object storage by itself has some limitations in the areas of data versioning and access control of data and thus Pachyderm could help in overcoming these.

### Technologies chosen for Data Pipelining and Scheduling
#### Docker
As having all the different elements of a data pipeline on different servers would not be practical, generally all these elements are hosted on different virtual instances of the same server. There are two ways to achieve this kind of virtualization:
- **Traditional Virtualization:** The this involves a  hypervisor which allows multiple Virtual Machines to run on a single machine. Each VM includes a full copy of an operating system, one or more apps, necessary binaries and libraries - taking up tens of GBs. VMs can also be slow to boot.

![vm](https://blogtechniquealtimate.files.wordpress.com/2017/04/photo4.png?w=300&h=270)

- **Containerization:** This involves containers which are stand-alone software packages which have all the dependencies bundled along with them but do not need an entire operating system to be attached. Multiple containers can run on the same machine and share the OS kernel with other containers, each running as isolated processes in user space. Containers take up less space than VMs (container images are typically tens of MBs in size), and start almost instantly. 

![container](https://bluesentryit.com/wp-content/uploads/docker-example-left.png)

We choose to use containers for our data pipeline as they take up lesser storage space and are relatively quick to boot up. We will use ***Docker*** to build these containers.

#### Kubernetes

Kubernetes is a platform for working with containers. Once we have a handful of containers running on a few servers, it can get really confusing to figure out how best to deploy the containers across the servers. Kubernetes provides a means to **deploy, scale and monitor** the various containers. This is done via the Kubernetes master node, which decides how best to schedule the different containers / apps to run on the servers / nodes available.

![kuber](http://cimage.tianjimedia.com/uploadImages/2015/210/R1XNZ9Y14CG0.jpg)

Thus, Kubernetes is a necessary complement to the Docker containers to build our automated data pipeline.

#### Pachyderm
We will use the services provided by Pachyderm again here in the Data Pipelining and Scheduling process. Pachyderm helps in tying together all the different elements of the data pipeline and takes care of triggering, data sharding, parallelism, and resource management on the backend.

### Technologies chosen for Analysis and Model Building

#### Python & TensorFlow

The actual problem of object detection would require us to use several image processing algorithms for which we will use Python language and TensorFlow libraries.

## Conclusion
With this week's article, we have a basic understanding of the different technologies that we are going to use to build the data pipeline which solves the problem of false alarms from security camera footages. In the next article we will go through the process of setting up the object storage required for our analytical pipeline.
