# Production Scale Big Data Implementation: Week 2

In the [previous article (Week 1 WriteUp)](https://github.com/kalyansashank/prod-scale-big-data/blob/master/Week%201%20WriteUp.md) we have seen how false positives are a serious problem to security service providers. It was also discussed that we are going to solve this problem by building an automated data pipeline. In this week’s article we are going to get a deeper understanding the first element of our data pipeline: gathering and organizing data.

## Gathering and Organizing Data Using Object Storage

### Object Storage? What are the other types of storage?
The image below explains the three storage systems.

![storagesystems](https://www.emc.com/content/dam/uwaem/production-design-assets/en/sdsaemmodule/images/ecs-object-storage-the-3-pillars-of-modern-enterprise-data-storage.gif)

#### Block Storage
This is the lowest level of data storage wherein the data is directly accessed by the Operating System and byte-level interactions are possible. This is useful for applications which use less data but need constant interactions with the data at a very granular level.

#### File Storage
File storage system involves a intermediary system (network file system) between the underlying local file system and the OS. This allows for a wider compatibility across Operating systems and has advantages like file locking (to prevent inconsistent states of data) and access controls.

#### Object Storage
Object storage is a data storage architecture which treats data as objects and not as a file hierarchy (File storage) or well-defined spaces in a structure (block storage). As we have seen above File Storage and Block Storage involve accessing the data through the operating system. Object storage, on the other hand involves data access at the application level. This means that there is very little scope of granular data interactions. Here, we are trading the ability to interact with granular data for the ability to efficiently handle large amounts of data as random I/O is replaced by sequential I/O. 

Further, in object storage, the metadata of each data file is stored along with the same file unlike the other storage systems. This means that there is much less overhead in accessing the metadata of a file and thus much more ease of use while handling huge amounts of data. It is thus clear to see the advantages of object storage as the size of data to be handled increases. 

#### Why use Object Storage?
**Cost**: The most important reason for using object storage is the low cost associated with it. Currently the cost of setting up a relational database server on Amazon RDS is more than twice that of the cost of setting up comparable compute instances on EC2 with object storage on S3.

**Flexibility**: The ability to store any kind of data without the overhead associated with other kinds of storage makes object storage attractive for big data implementations.

**Scalability**: We have seen above, how sequential I/O and the ability to store metadata on file, give object storage an advantage in handling huge amounts of data.

**Simple to use**: The data in object storage can be accessed and written back in simple ways as the data is accessed and written back only in bulk (there is no random I/O).

#### What are the ways to interact with Object Storage?
Object storage can be accessed in the following ways:
1. **Command Line Interface**: We can use commands like GET, PUT, DELETE
2. **Graphical User Interface**: Examples: S3 browser
3. **Application Program Interface**: Almost every object storage service will have an API that can directly interact with the data from within an application

#### Are there any shortcomings to using object storage?
To make sure that we can manage huge amounts of data using object storage, we have given up some features available in Block/File Storage systems. This has led to shortcomings in following areas:
- Organization: No proper way to organize data
- Practical Data Versioning: There is no simple way to version controlling of data
- Access controls: There is no way to limit the access of certain data to different users of the system
- Compliance /Privacy: Lack of access control leads to problems with privacy

#### Which object storage service are we going to use for the data pipeline?
There are a lot of object storage service providers, but we will choose from the following:
1. Amazon S3
2. Microsoft Azure Blob Storage
3. Google Cloud Storage

Our team will decide on which service to use based on the cost of setting up storage and compute instances and also the cost of data transfer.

#### How are we going to overcome the problems with object storage in our pipeline?
We will be using **Pachyderm** to overcome the problems associated with organization, data versioning, & access control of data in object storages.

#### Pachyderm

Pachyderm is tool that will help us in various stages of building and managing our data pipeline. Particularly in data storage element it will help with the following parts:

1. **Storing and interacting with data (Organization)**: All the data can be put in repositories in pachyderm, but the data will eventually reside in an object store of our choice. With pachyderm, we will get the advantages of object storage but can still interact with the data like we would in a normal file system. The data can be added to a repo using the GUI or ```pachctl```.
2. **Version Controlling**: The data stored in pachyderm is content-addressed by Pachyderm to build version control semantics. This means that Pachyderm acts like "Git" for our data in the object storage and tracks all the changes in history and makes it easier to rollback to previous versions if needed.
3. **Access Controls**: Pachyderm has access control features that let us create and manage various users who can interact with the various repositories on the server. By managing the access to the repositories, we can control the access of pipelines to different users. Activating access controls can also be done by GUI or ```pachctl```.

## Conclusion

In this article we have seen how we are going to setup the object storage element of our data pipeline. In the next article we will explore further elements.