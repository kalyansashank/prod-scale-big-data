# Production Scale Big Data Implementation: Week 7
## Introduction
This is the last article in this series of articles where we have learnt how to build an automated data pipeline to solve the problem of false alarms from security cameras by using object detection. We have studied different aspects of each element of the pipeline and the technology used to build it like Docker, Kubernetes, Pachyderm, Object Storage, Python & TensorFlow. In last week's articlle we built the pipeline by tying together all these elements using Pachyderm. Below is the outline of the pipeline for reference.

![pipeline](https://docs.google.com/drawings/d/e/2PACX-1vQN-KSMWbQwsftzff-TP71U0L5twlKThzp0i3IWxbenakpQFLN_g6zI0DrjbfGf_R3vA2biTkA53T3t/pub?w=2417&h=992)

Let us briefly revisit the different elemnts of the pipeline and its architechture.
## Automated Data Pipeline: Summary
- We used Amazon's S3 object storage to store all the data that is required as input to the different stages of the pipeline and also thier outputs([Week 2](https://github.com/kalyansashank/prod-scale-big-data/blob/master/Week%202%20WriteUp.md))
- We performed object detection using artifical neural networks built on Python and TensorFlow([Week 3](https://github.com/kalyansashank/prod-scale-big-data/blob/master/Week%203%20WriteUp.md))
- We built validation, threat detection, & notification elements using Python and send grid API([Week 4](https://github.com/kalyansashank/prod-scale-big-data/blob/master/Week%204%20WriteUp.md))
- Once all the elements were built individually we packaged them into Docker containers for easy deployment([Week 5](https://github.com/kalyansashank/prod-scale-big-data/blob/master/Week%205%20WriteUp.md))
- We then tied all the different elements together and built the pipeline using Kubernetes & Pachyderm([Week 6](https://github.com/kalyansashank/prod-scale-big-data/blob/master/Week%206%20WriteUp.md))

At the end of this process we have a data pipeline with an underlying infrastructure as shown below:
![infrastructure](https://keep.google.com/u/0/media/v2/10anRnCYDEeMzzLnQ_xByCj15ID_yP4g_nWKmj0vNbGq9tTEcdSgvCzahjMVPOg/1I3X2IW0Q1VRd1kBRBbW0IQjGif1AZl2YNw-zZLO2FMuFHNo6S0qf9OAKNP0g0zA?accept=image/gif,image/jpeg,image/jpg,image/png,image/webp,audio/aac&sz=600)

We now have our pipeline running on Amazon's EC2 cloud. We have used **kops** to create the required EC2 instances and Kubernetes clusters but we will not get into the details of using [kops](https://kubernetes.io/docs/getting-started-guides/kops/). We can always set up these clusters as per our requirements manually as well. Now that the data pipeline is up and running on EC2 and is ready to automatically process any images that are put into the images repo, we shall explore the ways to **update, maintain, & scale** our pipeline in this article.

## Updating the Pipeline

With machine learning algorithms, like the neural network we used for object detection in our pipeline, data scientists are always tweaking them and improving thier accuracy. Once we have tested our new code and are sure to change the code currenlty deployed in the pipeline we have to follow these steps to update the pipeline:
1. **Make the code changes**
2. **Re-build your Docker image**: Here we have to be sure to **rename the docker image** with a name that is different from what it was before. This is required because the if the same name is used in the new pipeline specification file then Pachyderm would assume the image to be already running and continues running the old docker image with the older version of the code. We can rename it with a version number or GitHub commit code.
3. **Change the pipeline specification to reflect the change in docker image name and call ```pachctl update-pipeline -f pipeline.json``` again to update the pipeline with new code**

Alternatively, we can let the docker image name to be the same but call the ```update-pipeline``` command with additional arguments ```--push-images``` to force Pachyderm to reload the docker image.
```pachctl update-pipeline -f edges.json --push-images```
When ```--push-images``` is specified, Pachyderm will do the following:
1. Tag your image with a new unique tag.
2. Push that tagged image to your registry (e.g., DockerHub).
3. Update the pipeline specification that you previously gave to Pachyderm with the new unique tag.

## Scaling the Pipeline : Distributed Computing

In addition to having the ability to update the pipeline, we would want our pipeline to have the ability to distribute the data to be processed at each step across all the available workers so that the runtime for each elements decreases and we use the resources optimally. Two implement distributed computing in our pipeline we need to think about two main considerations:
1. How many workers to involve in the distributed computing?
2. How to split the data that is processed by each of the workers?

Before we get into how we can handle the above two considerations in Pachyderm, let us understand what are Pachyderm workers.

### Pachyderm Workers
Each "Pachyderm worker" for a given element of a pipeline is an identical pod running the Docker image you specified in the pipeline spec of that element. For each step in our data pipeline, Pachyderm distributes the input data across these workers.

Pachyderm workers are spun up when the pipeline is first created and are left running in the cluster waiting for new jobs (data) to be available for processing (committed).

Now that we understand what workers are, let us see how Pachyderm decides how many workers to involve and how to distribute the data.
### 1. How many workers to involve in the distributed computing? (Parallelism)
We can specify the number of workers to involve for each pipeline element in the pipeline specification json.
```
  "parallelism_spec": {
    // Exactly one of these two fields should be set
    "constant": int
    "coefficient": double
```
Specifying the constant parameter, always tries to involve that constant number of workers. However, setting the coefficient field will start a number of workers that is a multiple of your Kubernetes cluster’s size.

Therefore, we can use the ```"parallelism_spec"``` argument in the pipeline specification to set the number of workers across which the input data for a pipeline element be distributed.
### 2. How to spread the data across these workers? (Glob patterns)
Here we will use a concept similar to Map / Reduce in hadoop to specify how the data should be split for each worker and how the output data should be collected called **Glob patterns**. However, **Glob Patterns** offers a much more flexible approach to data distribution.

We can specify the way we want the input data to a pipeline to be split in to **"datums"** using the ```atom``` argument within ```input``` argument of the **pipeline specification json**.

```
"input": {
    "atom": {
        "repo": "string",
        "glob": "string",
    }
}
```

Datums are the primary unit of parallelism in Pachyderm and each  worker works on one datum at a time. Therefore, we can use the ```glob``` argument within ```atom```, to specify how finely the data is to be separated for distributed computing.

## Future Improvements 
Now that we have built our pipeline, discussed about the ways to update it and scale it up/down using distributed computing let us see what improvement or changes we could make in the pipeline in the future.

1. **Ability to Handle Video Input**: We could add an additional layer before the image validation step, to check if the input is a video feed and extract the frames after every 5 seconds as images and tag it with proper names. In this way we can handle video input as well.
2. **Dynamic Threat Rules Based on Device ID**: We know that not all cameras are in similar environments and thus the rules for threat detection should be updated such that we look up a different set of rules for each category of devices.
3. **Sending the Image of the Threat as Part of the Notification**: We should be able to send the image along with the tag of the identified object to the user so that they can assess the real danger.

##Conclusion
In this series of seven articles we have seen how serious the problem of false alarms is for Security companies. We have learnt how to build an automated data pipeline from the scratch to help reduce these occurences of false alarms. We finally have a working pipeline deployed on the cloud, that can actually solve this problem for a security company if they use it. We have also learnt how to update, maintain & scale this pipeline in case a company really requires this and wants to deploy it in production.



