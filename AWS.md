# AWS
> A full sevice cloud platform

* Online hosting service
* Deploy web applications
* Deplou databases


## EC2/ECS
>  Remote VM
1. Repository or Elastic Container Registry (ECR)： where you store the app image created using docker
2. Cluster: where AWS runs containers.
3. Task Definition: where you tells AWS how to create your containers. 
4. Service: a collection of containers that will run on EC2 instances (ECS container instances) 
5. Task — This is a running container with the settings defined in the Task Definition.
6. EC2 - is simply a remote (virtual) machine. which is docker host

In simple words,ECS is a manager while EC2 instances are just like employees. All the employees (EC2) under this manager(ECS) can perform "Docker" tasks and the manager also understands "docker" pretty well. So,whenever you need "docker" resources, you show up to the Manager. Manager already has status from every employee(EC2) decides which one should perform the task.

* EC2 All you get is operating system, you have to manully install other apps
* Elastic Bennstalk is pre-package platform(PAAS), All you do is deploy your code

## S3 
* Simple Storage Service
* A bucket is a container for objects stored in Amazon S
* Objects are the fundamental entities stored in Amazon S3. Objects consist of object data and metadata.
* A key is the unique identifier for an object within a bucket，Every object in a bucket has exactly one key.
* Amazon S3 offers 3 storage classes. 
 1. STANDARD for general-purpose storage of frequently accessed data, 
 2. STANDARD_IA for long-lived, but less frequently accessed data, 
 3. GLACIER for long-term archive.
* The main difference between Amazon EC2 and S3 is that EC2 is a computing service that allows companies to run servers in the cloud. While S3 is an object storage service used to store and retrieve data from AWS through the Internet. S3 is like a giant hard drive in the cloud, while EC2 offers CPU and RAM in addition to storage

## AWS Dynomodb 
> Managed NoSQL database service

Tables – Similar to other database systems, DynamoDB stores data in tables. A table is a collection of data. For example, see the example table called People that you could use to store personal contact information about friends, family, or anyone else of interest. You could also have a Cars table to store information about vehicles that people drive.

Items – Each table contains zero or more items. An item is a group of attributes that is uniquely identifiable among all of the other items. In a People table, each item represents a person. For a Cars table, each item represents one vehicle. Items in DynamoDB are similar in many ways to rows, records, or tuples in other database systems. In DynamoDB, there is no limit to the number of items you can store in a table.

Attributes – Each item is composed of one or more attributes. An attribute is a fundamental data element, something that does not need to be broken down any further. For example, an item in a People table contains attributes called PersonID, LastName, FirstName, and so on. For a Department table, an item might have attributes such as DepartmentID, Name, Manager, and so on. Attributes in DynamoDB are similar in many ways to fields or columns in other database systems.


Primary key: Each item in the table has a unique identifier, or primary key, that distinguishes the item from all of the others in the table. DynamoDB supports two different kinds of primary keys: A simple primary key, composed of one attribute known as the partition key.Referred to as a composite primary key, this type of key is composed of two attributes

## RDS
> Relational Database Service

Create an Instance
1. Select an DB engine:  launch database
2. Choose use case: 
3. Specify DB detailsL password, username

Configure database
1. Configure security inbound rule: Edit Source address
2. Test Connectivty between Workbench and AWS: Edit the hostname of local workbench with endpoint from aws

# Summary

* What container do you deploy your application,how to container the application? Docker?
Docker

* other aws server?
EC2, S3, ECS

* In which scenario do you use S3 and DynamoDB?
Amazon DynamoDB stores structured data, indexed by primary key, and allows low latency read and write access to items ranging from 1 byte up to 64KB. Amazon S3 stores unstructured blobs and suited for storing large objects up to 5 TB. In order to optimize your costs across AWS services, large objects or infrequently accessed data sets should be stored in Amazon S3, while smaller data elements or file pointers (possibly to Amazon S3 objects) are best saved in Amazon DynamoDB.


* what it ec2
Developers can create instances of virtual machines and easily configure the capacity scaling of instances using the EC2 web interface.


* How Do You Setup AWS EC2?
EC2 setup involves creating an Amazon Machine Image (AMI), which includes an operating system, apps, and configurations. That AMI is loaded to the Amazon Simple Storage Service (S3), and it’s registered with EC2, at which point users can launch virtual machines as needed

* AWS EC2 Benefits
1. EC2 reduces the time to boot new servers
2. Scaling capacity based on changes to computing requirements
3. Complete control of servers
4. Flexibility with operating systems
5. Built-in security 


*  Dynomodb basic conception/creat/ configuration
* How do you map object from application to DynamoDB? How do you configure?
*  What container do you deploy your application?



