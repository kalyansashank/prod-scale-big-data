# Production Scale Big Data Implementation: Week 4

In the last three weeks we have discussed how we plan on building a data pipeline for helping security companies overcome the problem of false alerts from security cameras. We have discussed about a few individual elements like storage and object detection of the data pipeline. This week we are going to discuss the pre-and post object processing steps of the data pipeline and eventually build the outline of the entire pipeline.

## Designing the Full Pipeline

In the broadest terms our pipeline can be split into the following parts:
1. Input : The image data coming from the various cameras in different locations
2. Output : We plan to have two kinds of outputs:
    - **Security Alert**: API call to alert service in case of an actual threat that is detected by our object detection system
    - **Threat Detection Analysis**: A plot to analyze the number of threats detected from each location/ user. This is important for compliance issues as well as to make sure that the detection system is working as intended for all the locations
3. Building Blocks : These are all the intermediate steps including processing steps (both machine learning algorithms & heuristic rules/conventions) & the intermediate storages. All the steps before the object detection are the preprocessing steps required to convert the raw input into the proper form for the detection algorithm to work properly. All the steps after are required to convert the output of detection algorithm into the form of output we require.

In building our data pipeline we are going to define these building blocks and define the connections between them. Following are the building blocks of our data pipeline:
### Pre-Processing
1. **Repository of Input Images**: This will be an object storage bucket managed by pachyderm. The images will be coming from all the camera **locations** at different **times**. Here we will have certain **rules** for the providers of the input images:

    - The file name should be in the form of **id_timestamp.jpg**. This is done so that we have an indication of the location / user id to which the image belongs and at what time each image is generated. Also, this implies that the images should be a jpeg to maintain consistency
    - Also, to eliminate confusion relating to time zones of the timestamp, we shall have all the timestamps in UTC time zone.

2. **Validation of the input images**: In this part we will check the input images if they have the right **resolution**, **format(jpeg)**, & **tag(names)**. The validated images are stored in a separate directory and used for the object detection. The invalid images are also stored to have a historical record of invalid images.

### Object Detection
3. **TensorFlow Object Detection**: Here we use Python and TensorFlow to detect the objects in each image and the object data for each image is stored in a separate json file. The output json files are stored in separate directories for each location (id).

### Post-Processing
4. **Threat Detector**: Here the detected object data for each id is compared to predefined rules for possible threats. The rules for the threat detection are defined differently for each id. For example, a camera which is placed outdoors should be more tolerant to a few kinds of objects like cars etc. than a camera which is placed indoors. The detected threat information is added to the json of the image and stored in the **final data repository**.
5. **API call to alert service**: If a certain id's image is tagged as having a threat then, we would want to send an API call to an alert service which would send a mail to the user / authorities.
6. **Plotting the chart to analyze alert trends**: We use the data stored in the final repository to plot a chart of the number of alerts that have been found in images of each in a fixed recent time period. This helps us in identifying problems in certain ids (a few ids may have too many or too less alerts indicating a problem with the equipment or positioning of cameras).

Below is a representation of the data pipeline:

![pipeline](https://docs.google.com/drawings/d/e/2PACX-1vQN-KSMWbQwsftzff-TP71U0L5twlKThzp0i3IWxbenakpQFLN_g6zI0DrjbfGf_R3vA2biTkA53T3t/pub?w=2417&h=992)

## Conclusion
While we will be using Amazon S3, Docker, Kubernetes, Python & TensorFlow for various individual elements of the data pipeline, Pachyderm will be the tool which will be helping us in integrating these elements and automating the entire process. In the next few weeks we will be developing the data pipeline outlined in this article.



