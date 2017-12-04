# Production Scale Big Data Implementation: Week 6
## Introduction
So far, we have made considerable progress in our journey of learning to build an automated data pipeline. As you know we are aiming to solve the problem of false alarms from security cameras with our data pipeline. As part of this exercise, we have seen how we can build an object detection model using Python & TensorFlow, store the required data in Object Storages (like Amazon S3), and have even outlined all the different elements of our pipeline.

The outline of our Data Pipeline for reference:
![pipeline](https://docs.google.com/drawings/d/e/2PACX-1vQN-KSMWbQwsftzff-TP71U0L5twlKThzp0i3IWxbenakpQFLN_g6zI0DrjbfGf_R3vA2biTkA53T3t/pub?w=2417&h=992)

Last week, we discussed about containerizing the different elements of the pipeline using Docker. In this article, we shall see how we can deploy these containers efficiently using **Kubernetes** and tie together all the different elements using **Pachyderm**.

We already discussed what are the additional considerations are required for the containerized elements in our data pipeline:
1. We will have to deploy multiple instances of the containers for each stage of the data pipeline - **Kubernetes takes care of this**
2. We will have to deploy these instances over a large cluster of resources and do it efficiently so that there is least chance of failure - **Kubernetes takes care of this**
3. We would want the different instances to be run in a proper sequence and have the right data being fed into each stage and the output collected properly - **Pachyderm takes care of this**

## What is Kubernetes? Why Kubernetes?

Kubernetes is a platform for working with containers. Once we have a handful of containers running on a few servers, it can get really confusing to figure out how best to deploy the containers across the servers. Kubernetes provides a means to **deploy, scale and monitor** the various containers. This is done via the Kubernetes master node, which decides how best to schedule the different containers / apps to run on the servers / nodes available.

![kuber](http://cimage.tianjimedia.com/uploadImages/2015/210/R1XNZ9Y14CG0.jpg)

Kubernetes master node is also helpful in re-assigning the containers to different nodes when nodes fail. This is a sort of auto-healing feature that is provided by Kubernetes.

## Why Pachyderm?

While Kubernetes is helpful in managing the deployment of the containers on nodes, there is not much it can do in terms of making sure the right tasks are performed in the right order, the correct input data is taken and output data is written to the right repositories in our Object Storage. This is where Pachyderm is useful.

Pachyderm helps in tying together all the different elements of the data pipeline and takes care of triggering, data sharding, parallelism, and resource management on the backend. 

There are three steps to build a pipeline element in Pachyderm:
1. **Writing the code**:  This is the app/code that we want to execute
2. **Building the Docker Image**: We have seen how we can build Docker Images using Dockerfiles in the previous week. In the Dockerfile, we specify the code as well as all the dependencies required for running this code.
3. **Creating a Pipeline referencing the Docker Image built in step 2**: In this step, we tell Pachyderm which image to run, how to run it, where (which repository) to take the input data from, and where to store the output. This is done using a JSON specification file. The four main components of the this pipeline specification JSON file are: name, transform (where the docker image and the code to run are specified), parallelism and input.

We will repeat the above steps for each of the elements in our pipeline. This will give us our full pipeline with proper connections between the different elements.

## Building the Pipeline: Hands On!!
We first need all the containers of different parts of the pipeline, the model for object detection, the rules of threat detection, and the input images. We have created all these required files and they are ready to use in this [GitHub repository](https://github.com/dwhitena/mgmt690-pipeline.git). So, we first pull this repository into our local system to start building our pipeline using following line of code:
```git clone https://github.com/dwhitena/mgmt690-pipeline.git```

From the pipeline diagram above, we see that the explicit input repositories required are three: one for the input images, one for object detection model, & one for threat detection rules. We can create these repositories using the following ```pachctl``` commands:
```
## Creating images repository
pachctl create-repo images
## Creating repository for storing the model
pachctl create-repo model
## Creating repository for storing the threat detection rules
pachctl create-repo rules
```

Once we have these repos setup, we now put the model in the ```model``` repo and the rules in the rules repo.
```
## Downloading the TensorFlow Object detection model
wget http://download.tensorflow.org/models/object_detection/ssd_mobilenet_v1_coco_11_06_2017.tar.gz
## Unzipping the model
tar -xvf ssd_mobilenet_v1_coco_11_06_2017.tar.gz
## Accessing the ProtoBuff Model graph
cd ssd_mobilenet_v1_coco_11_06_2017
## Placing the model in the model repo
pachctl put-file model master -c -f frozen_inference_graph.pb
```

Now, before we put the rules in the rules repo we open the json file and change the rules or any other parameter that we want. For example, we might want to change the email address to which the alerts are sent. We can open the json file for this using :
```
## Navigating into the threat detect folder in the git repo
cd ../threat-detect/example_rule
## loading the rule.json file and making changes
pico rule.json
```

After making the required changes we can place the rule.json file in the ```rules``` repo to use in the threat detection exercise.
```
pachctl put-file rules master -c -f rule.json
```

All the required repos are now created, and the rules and model repos are appropriately populated with proper data. We will put the images in the ```images``` repo at the end. This is because the pipeline is designed such that any image put into the input images repo will be immediately processed. This is an essential feature as in the actual production environment we want the images to be processed as and when they arrive.

So, we first build the entire pipeline and then add images. Before we build the pipeline, elements using ```pachctl``` commands we can open the pipeline spec json files and make any required changes. For example, we have to open the threat detector specifications file to mention the API key for **sendgrid**. Again this can be done by :
```pico threat-detect.json```

Now we can build all the pipelines:
```
pachctl create-pipeline -f validate.json
pachctl create-pipeline -f object-detect.json
pachctl create-pipeline -f threat-detect.json
pachctl create-pipeline -f plot.json
```
At this point we can add an example image to the images repo and this will trigger the pipeline. The image will be processed, and the notification sent to the customer.
```
## navigating to the folder where example images are saved
cd test_images

## putting the images in images repo
pachctl put-file images master -c -f dev1_1511891155.jpg
```

We can then open the pachyderm dashboard to see if all the elements have worked as expected.
As we can see below, the pipeline is built and is working properly. 
![dash](https://imgur.com/Hfub0zJ.jpg)

If everything worked correctly, the email id mentioned in the rules.json file should get a threat notification (if the image processed has a threat) and the plot should be updated.

## Conclusion

This week we have finally built the pipeline we have been working on for the last 6 weeks. In the next iteration of this series we will see how we can update, maintain and scale this pipeline.
