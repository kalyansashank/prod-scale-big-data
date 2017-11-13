# Production Scale Big Data Implementation: Week 3

So far we have discussed about how we plan to implement an object detection based data-pipeline to tackle the problem of false alarms faced by video surveillance systems. We have been through the details of the data storage element of our data pipeline in previous articles. In this article, we are goin to discuss the actuall object detection model and the tools we are going to use to build it. 

Our object detection model will be a neural network model which we will build using the TensorFlow library on Python.

## Neural networks

A neural network is a machine learning method, in which a computer learns to perform some task by analyzing training examples. An object detection neural net would be fed thousands of pictures labeled as cars, dogs, people, etc. The neural net would then find the patterns of similarity among the pictures which are identically labeled.

A neural net consists of thousands or even millions of interconnected processing nodes. Below is an illustration of a simple neural net.

![neralnet](https://upload.wikimedia.org/wikipedia/commons/thumb/4/46/Colored_neural_network.svg/300px-Colored_neural_network.svg.png)
[Source](https://upload.wikimedia.org/wikipedia/commons/thumb/4/46/Colored_neural_network.svg/300px-Colored_neural_network.svg.png)

The nodes in neural nets are organized into layers, and they’re “feed-forward,” meaning that data moves through them in only one direction. An individual node might be connected to several nodes in the layer beneath it, from which it receives data, and several nodes in the layer above it, to which it sends data. Each node's output is a function of all the inputs that feed into it (each input could be weighted differently).

For our object detection model, we will be using a combination of **convolutional nueral network** layers and **fully connected layers**. The convolution layers involves filter matrices that identify the features in the image and the fully connected layer helps in classifying the features and in turn the actuall image (more about this [ here](https://ujjwalkarn.me/2016/08/11/intuitive-explanation-convnets/)). The image below illustrates the two different layers.


![cnn](https://github.com/kristjankorjus/Replicating-DeepMind/blob/master/doc/report/images/Convolutional_NN2.png?raw=true)
[Source](https://github.com/kristjankorjus/Replicating-DeepMind/blob/master/doc/report/images/Convolutional_NN2.png?raw=true)

Here is an illustration of object detection with convolutional neural networks. The convolutional layers abstract the features from the given input image which are then used in the fully connected layers to make the final classification.
![8](https://ujwlkarn.files.wordpress.com/2016/08/conv_all.png?w=1024)
[Source](https://ujwlkarn.files.wordpress.com/2016/08/conv_all.png?w=1024)

Now that we understand the model that we will be using for object detection, let us take a look at the tech platform that allows us to build this model: **TensorFlow** 

## TensorFlow : Introduction

TensorFlow is an open source software library for implementing machine learning algorithms using data flow graphs. It is based on dataflow programming, in which the program is modelled as a directed graph of data flowing between operations. In dataflow graphs the nodes represent units of computation, and the edges represent the data consumed or produced by a computation. The image below is an illustration of generic dataflow graphs.
![dataflow](https://www.tensorflow.org/images/tensors_flowing.gif)

## Advantages of TensorFlow
There are several advantages of using a dataflow approach to programming. A few are listed below:
1. Portability: The dataflow graph is language agnostic and thus can be very easily implemented on a different machine with different architecture and operating system or even on an entirely different programming language.
2. Distributed & Flexible execution: The graph allows TensorFlow to easily identify the processes which can be run parallelly and across various CPUs & GPUs available which lead faster computations.
3. Active support communities: The user base of TensorFlow is large and growing and these users are very active on GitHUb and stack overflow. This makes it relatively easy to find solutions to problems that we might face while using TensorFlow.
4. Competent pre existing Object Detection API:  The TensorFlow Object Detection API is an open source framework built on top of TensorFlow that makes it easy to construct, train and deploy object detection models.

## Our Implementation

For our data pipeline we will be using the data from [coco dataset](http://cocodataset.org/#home) which is a large-scale object detection, segmentation, and captioning dataset. We will also utilize a [pre-built object detection model](https://github.com/Zehaos/MobileNet) based on TensorFlow. 


