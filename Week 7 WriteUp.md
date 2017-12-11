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
3. **Change the pipeline specification to reflect the chnage in docker image name and call ```pachctl update-pipeline -f pipeline.json``` again to update the pipeline with new code**

Alternatively, we can let the docker image name to be the same but call the ```update-pipeline``` command with additional arguments ```--push-images``` to force Pachyderm to reload the docker image.
```pachctl update-pipeline -f edges.json --push-images```
When ```--push-images``` is specified, Pachyderm will do the following:
1. Tag your image with a new unique tag.
2. Push that tagged image to your registry (e.g., DockerHub).
3. Update the pipeline specification that you previously gave to Pachyderm with the new unique tag.

## 


