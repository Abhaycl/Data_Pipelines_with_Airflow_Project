# Data Pipelines with Airflow Project Starter Code

This project will introduce to the core concepts of Apache Airflow. Where we will create our own custom operators to perform tasks such as staging the data, filling the data warehouse, and running checks on the data as the final step.

<!--more-->

[//]: # (Image References)

[image01]: ./images/user1.jpg "User creation"
[image02]: ./images/user2.jpg "User creation"
[image03]: ./images/user3.jpg "User creation"
[image04]: ./images/user4.jpg "User creation"
[image05]: ./images/user5.jpg "User creation"
[image06]: ./images/role1.jpg "Role creation"
[image07]: ./images/role2.jpg "Role creation"
[image08]: ./images/role3.jpg "Role creation"
[image09]: ./images/role4.jpg "Role creation"
[image10]: ./images/securitygroup1.jpg "Security Group Configuration"
[image11]: ./images/securitygroup2.jpg "Security Group Configuration"
[image12]: ./images/cluster1.jpg "Redshift cluster creation"
[image13]: ./images/cluster2.jpg "Redshift cluster creation"
[image14]: ./images/cluster3.jpg "Redshift cluster creation"
[image15]: ./images/cluster4.jpg "Redshift cluster creation"
[image16]: ./images/cluster5.jpg "Redshift cluster creation"
[image17]: ./images/cluster6.jpg "Redshift cluster creation"
[image18]: ./images/apache.jpg "Apache Airflow"
[image19]: ./images/star_schema.jpg "Star Schema"
[image20]: ./images/dag.png "DAG dependencies"


---


#### How to run the program with our own code

## Project Requirements

The requirements for the project are a valid aws account, with accompanying security credentials, as well as a python environment, which satisfies the module requirements given.

- An IAM user must be created.

![alt text][image01]
![alt text][image02]
![alt text][image03]
![alt text][image04]
![alt text][image05]


- A role must be created for redshift.

![alt text][image06]
![alt text][image07]
![alt text][image08]
![alt text][image09]


- A default VPC has to be created in which the security group has to be configured.

![alt text][image10]
![alt text][image11]


- This time we create a redshift cluster on AWS.

![alt text][image12]
![alt text][image13]
![alt text][image14]
![alt text][image15]
![alt text][image16]
![alt text][image17]


- We need to perform an installation of the Apache airflow software and run the following command to run the application:

```bash
   /opt/airflow/start.sh
```

![alt text][image18]


---

The summary of the files and folders within repo is provided in the table below:

| File/Folder              | Definition                                                                                                   |
| :----------------------- | :----------------------------------------------------------------------------------------------------------- |
| dags/*                   | Folder containing the code of our DAG for the Airflow UI.                                                    |
| images/*                 | Folder containing the images of the project.                                                                 |
| plugins/helpers/*        | Folder containing the class for the SQL transformations.                                                     |
| plugins/operators/*      | Folder containing all operator templates.                                                                    |
|                          |                                                                                                              |
| create_tables.py         | Functions to create the fact and dimension tables for the star schema in Redshift.                           |
|                          |                                                                                                              |
| README.md                | Contains the project documentation.                                                                          |
| README.pdf               | Contains the project documentation in PDF format.                                                            |


---

**Steps to complete the project:**

1. To complete the project, you will need to create your own custom operators to perform tasks such as staging the data, filling the data warehouse, and running checks on the data as the final step.


## [Rubric Points](https://review.udacity.com/#!/rubrics/2478/view)
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
## Scenario

A music streaming company, Sparkify, has decided that it is time to introduce more automation and monitoring to their data warehouse ETL pipelines and come to the conclusion that the best tool to achieve this is Apache Airflow.

They have decided to bring you into the project and expect you to create high grade data pipelines that are dynamic and built from reusable tasks, can be monitored, and allow easy backfills. They have also noted that the data quality plays a big part when analyses are executed on top the data warehouse and want to run tests against their datasets after the ETL steps have been executed to catch any discrepancies in the datasets.

The source data resides in S3 and needs to be processed in Sparkify's data warehouse in Amazon Redshift. The source datasets consist of JSON logs that tell about user activity in the application and JSON metadata about the songs the users listen to.


## Schema definition

To represent this context a star schema has been used.

The songplays table is the core of this schema, is it our fact table and it contains foreign keys to four tables:

    * start_time REFERENCES time(start_time)
    * user_id REFERENCES users(user_id)
    * song_id REFERENCES songs(song_id)
    * artist_id REFERENCES artists(artist_id)

There are also two staging tables; One for event dataset and one for song dataset.
![alt text][image19]


## Preamble

In this project we are going to use two Amazon Web Services, S3 (Data storage) and Redshift (Data warehouse with columnar storage).


## Datasets

Data sources are provided by two public S3 buckets. One bucket contains info about songs and artists, the second has info concerning actions done by users (which song are listening, etc.. ). The objects contained in both buckets are JSON files. The song bucket has all the files under the same directory but the event ones don't, so we need a descriptor file (also a JSON) in order to extract data from the folders by path. We used a descriptor file because we don't have a common prefix on folders.

For this project, you'll be working with two datasets. Here are the s3 links for each:

- Log data: ```s3://udacity-dend/log_data```
- Song data: ```s3://udacity-dend/song_data```

The Redshift service is where data will be ingested and transformed, in fact though COPY command we will access to the JSON files inside the buckets and copy their content on our staging tables.


## Configuring the DAG

In the DAG, we add default parameters according to these guidelines

- The DAG does not have dependencies on past runs
- On failure, the task are retried 3 times
- Retries happen every 5 minutes
- Catchup is turned off
- Do not email on retry

We configure the task dependencies so that the graph view follows the flow shown in the image below.
![alt text][image20]


## Building the operators

We build four different operators that will stage the data, transform the data, and run checks on data quality.

We utilize Airflow's built-in functionalities as connections and hooks as much as possible and let Airflow do all the heavy-lifting when it is possible.

All of the operators and task instances will run SQL statements against the Redshift database. However, we build flexible, reusable, and configurable operators you can later apply to many kinds of data pipelines with Redshift and with other databases.


## Stage Operator

The stage operator is expected to be able to load any JSON formatted files from S3 to Amazon Redshift. The operator creates and runs a SQL COPY statement based on the parameters provided. The operator's parameters should specify where in S3 the file is loaded and what is the target table.

The parameters should be used to distinguish between JSON file. Another important requirement of the stage operator is containing a templated field that allows it to load timestamped files from S3 based on the execution time and run backfills.


## Fact and Dimension Operators

With dimension and fact operators, we utilize the provided SQL helper class to run data transformations. Most of the logic is within the SQL transformations and the operator is expected to take as input a SQL statement and target database on which to run the query against. We can also define a target table that will contain the results of the transformation.

Dimension loads are often done with the truncate-insert pattern where the target table is emptied before the load. Thus, you could also have a parameter that allows switching between insert modes when loading dimensions. Fact tables are usually so massive that they should only allow append type functionality.


## Data Quality Operator

The final operator to create is the data quality operator, which is used to run checks on the data itself. The operator's main functionality is to receive one or more SQL based test cases along with the expected results and execute the tests. For each the test, the test result and expected result needs to be checked and if there is no match, the operator should raise an exception and the task should retry and fail eventually.

For example one test could be a SQL statement that checks if certain column contains NULL values by counting all the rows that have NULL in the column. We do not want to have any NULLs so expected result would be 0 and the test would compare the SQL statement's outcome to the expected result.