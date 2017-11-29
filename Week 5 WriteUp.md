# Production Scale Big Data Implementation: Week 5
## Introduction
In the past 4 weeks we have made considerable progress in our journey of learning to build an automated data pipeline. As you know we are aiming to solve the problem of false alarms from security cameras with our data pipeline. As part of this exercise, we have seen how we can build an object detection model using Python & TensorFlow, store the required data in Object Storages (like Amazon S3), and have even outlined all the different elements of our pipeline. 

The outline of our Data Pipeline for reference:
![pipeline](https://docs.google.com/drawings/d/e/2PACX-1vQN-KSMWbQwsftzff-TP71U0L5twlKThzp0i3IWxbenakpQFLN_g6zI0DrjbfGf_R3vA2biTkA53T3t/pub?w=2417&h=992)

Today we will discuss how we plan to overcome the next big problem in building data pipelines: **getting our software to run as expected when moved from development environment to the production environment**.

## Deploying applications
Before I get into the solution that we are using for our data pipeline, let us look at the different solutions people have used for this problem of software deployment in the past:

### The Old Ways
1. Having each script / app (this would be the validation / object detection / threat detector parts of our pipeline) in a separate compute instance: 
    - This method involves having a separate server for each app and installing all the dependencies, libraries and other binaries, and configuration files needed for running the app on the respective servers manually
    - This method is extremely slow to setup and involves a lot of avoidable expense as each app might not require an entire compute instance
2. Having multiple apps per instance and installing all the dependencies directly on this instance:
    
    - This method is again too slow to setup and we might run into problems with different apps requiring conflicting versions of the dependencies
![multiapp](https://imgur.com/8mbaqVg.jpg)
[Source](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/#why-do-i-need-kubernetes-and-what-can-it-do)

3. Utilizing Virtual Machines to isolate multiple apps on one instance:

    - This involves a hypervisor which allows multiple Virtual Machines to run on a single machine
    - Each VM includes a full copy of an operating system, one or more apps, necessary binaries and libraries - taking up tens of GBs. VMs can also be slow to boot.
    
    ![vm](https://blogtechniquealtimate.files.wordpress.com/2017/04/photo4.png?w=300&h=270)

To overcome the problems of portability faced by the VMS, but still have a way to isolate the working environments of different we utilize software containerization.

### Software Containers : The New, Better Way

**Containerization:** This involves containers which are stand-alone software packages which have all the dependencies bundled along with them but do not need an entire operating system to be attached. 
- Multiple containers can run on the same machine and share the OS kernel with other containers, each running as isolated processes in user space
- Containers take up less space than VMs (container images are typically tens of MBs in size), and start almost instantly

![container](https://bluesentryit.com/wp-content/uploads/docker-example-left.png)

With high boot speed and smaller size, containers provide several advantages in application deployment, a few of which are mentioned below:
1. **Reproducibility & Shareability**: Containerizing , by abstracting away the differences in OS distributions and underlying infrastructure, has made it easier to share and reproduce the results of an app as expected
2. **Software Modularity**: As the containerized apps can be started almost instantaneously we can split the various parts of our data pipeline into separate modules. This has helped us in keeping each of our modules(validation / object detection / threat detector) simple and easy to maintain.

## So, What is Docker?

Docker is a software technology which provides container building and holding services. While Docker is not the only container platform service provider, they are the most popular ones and have become almost synonymous with this technology.

Now that we have decided to use containers to deploy our scripts in, let us get accustomed with some Docker terminology before start building our containers.

## Docker Terminology

**Docker Images**: An image is the package that contains our actual app (the code) and all the dependencies required to run it.

**Docker Container**: A container is a running instance of an image

**Engine**: The application that builds and runs the images based on the instructions from Dockerfiles

**Registry**: The place where the images are stored

**Dockerfile**: Dockerfile is a setup file which tells the engine how to build a particular image.

## Building Docker Containers
Now that we are familiar with the terminology of Docker, let us look at the process of creating the images and deploying containers.

1. Develop the application to deploy by packaging into a container: 
    - Not surprisingly, this is the first step. In addition to building the app itself, we also must create the Dockerfile with the specifications of dependencies and environment variables required for building a docker image for this app. Both the app and the Dockerfile are pushed to a GitHub repo.
    - This is generally the only manual step. All the steps below are to be automated in production but we will go through them to understand the process.

2. Build a Docker Image for the app with Docker Engine
3. Upload the image to a registry (This registry can be Docker Hub)
4. Deploy a Docker container, based on the image and from the registry(Docker Hub in our case), to a VM / cloud compute instance

Now we will follow the above process to Dockerize the image validation part of our data pipeline. In the actual pipeline we will be using Pachyderm & Kubernetes to automatically deploy these containers. However, we shall go through the exercise of actually deploying a container to have an understanding of what is happening behind the curtains in our final pipeline.

## Hands-on Docker Container Deployment

Here the first three steps of the process: creating the app, the Docker Image, & uploading it to the Docker Hub registry are already done. We can find many readymade images on Docker Hub which is like a GitHub repo for Docker Images.

The image we need for the validation is on Docker hub with the name dwhitena/mgmt-validate:slim. We can **pull** this image into our local system by running the following line of code:
```sudo docker pull dwhitena/mgmt-validate:slim```

Now we have downloaded the validation app with all the dependencies required to run it.
After we have the required image in our local system we can deploy the image as a container by running the following line of code:
```sudo docker run dwhitena/mgmt-validate:slim```

This will run the image as it was intended. However, we can also run the container interactively to control the way it runs and perform additional tasks in the environment using the following line of code:
```sudo docker run -it dwhitena/mgmt-validate:slim /bin/ash```

We can see the validation app docker container in action in the image below:
![code](https://i.imgur.com/U0buBS5.png)

As mentioned before, in our actual data pipeline we will not be running each container manually. So, let us now see what we will do differently in our pipeline.

## Additional Considerations for our Pipeline

1. We will have to deploy multiple instances of the containers for each stage of the data pipeline
2. We will have to deploy these instances over a large cluster of resources and do it efficiently so that there is least chance of failure
3. We would want the different instances to be run in a proper sequence and have the right data being fed into each stage and the output collected properly

Hence in the final pipeline, we will build Docker Images as explained today but use Pachyderm and Kubernetes to deploy the containers so as to take care of these additional considerations. 

## Conclusion
In next week's article we will see how we will use Kubernetes to deploy the containers and Pachyderm to tie all the elements of our data pipeline together.


