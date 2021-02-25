# Data Pipelines with Airflow Project Starter Code

This project will introduce to the core concepts of Apache Airflow. Where we will create our own custom operators to perform tasks such as staging the data, filling the data warehouse, and running checks on the data as the final step.

<!--more-->

[//]: # (Image References)

[image1]: ./images/redshift_start.jpg "Start of the redshift cluster"
[image2]: ./images/create_tables.jpg "Creation of the tables"
[image3]: ./images/etl.jpg "Transformation of data"
[image4]: ./images/redshift_stop.jpg "Stop of the redshift cluster"
[image5]: ./images/star_schema.jpg "Star schema"
[image6]: ./images/cluster_info.jpg "Cluster Information"
[image7]: ./images/cluster_queries.jpg "Queries"

---


#### How to run the program with our own code

## Project Requirements

The requirements for the project are a valid aws account, with accompanying security credentials, as well as a python environment, which satisfies the module requirements given.

You will need to add aws access key and secret information to the dwf.cfg file, under [AWS_ACCESS]. This is not to be comitted to git.

The parameterization of the dwh.cfg file is shown below.
```code
[CLUSTER]
HOST=This parameter will be defined according to the configuration set up
DB_NAME=This parameter will be defined according to the configuration set up
DB_USER=This parameter will be defined according to the configuration set up
DB_PASSWORD=This parameter will be defined according to the configuration set up
DB_PORT=5439
CLUSTER_IDENTIFIER=This parameter will be defined according to the configuration set up
NODE_TYPE=ds2.xlarge
NODE_COUNT=2

[AWS_ACCESS]
AWS_ACCESS_KEY_ID=This parameter will be defined according to the configuration set up
AWS_SECRET_ACCESS_KEY=This parameter will be defined according to the configuration set up
AWS_REGION=us-west-2

[IAM_ROLE]
NAME=dwhRole
POLICY_NAME=AmazonS3ReadOnlyAccess
ARN=arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
REDSHIFT_ARN=This parameter will be defined according to the configuration set up

[S3]
LOG_DATA=s3://udacity-dend/log_data
LOG_JSONPATH=s3://udacity-dend/log_json_path.json
SONG_DATA=s3://udacity-dend/song_data
```


We also have to create our security group which has to be assigned to the default VPC. Because the creation of our cluster has to belong to a VPC.

**NOTE:** _To follow IAC (Infrastructure as Code) practices, and to allow us to easily start and stop the redshift cluster to save costs, we can use the following scripts;_

    redshift_start.py
    redshift_stop.py

The scripts will create/remove the neccessary resources for redshift to run.

For the execution of our own code, we go to the project workspace or to our own terminal in Visual Code.

In the project workspace we can open a terminal and run the following files:

Start the redshift cluster.
```bash
  python redshift_start.py
```
![alt text][image1]

Create the tables.
```bash
  python create_tables.py
```
![alt text][image2]

Data transformations.
```bash
  python etl.py
```
![alt text][image3]

Stop the redshift cluster.
```bash
  python redshift_stop.py
```
![alt text][image4]


---

The summary of the files and folders within repo is provided in the table below:

| File/Folder              | Definition                                                                                                   |
| :----------------------- | :----------------------------------------------------------------------------------------------------------- |
| images/*                 | Folder containing the images of the project.                                                                 |
|                          |                                                                                                              |
| auxiliary.py             | Auxiliary functions for creating the connection to the RedShift cluster.                                     |
| create_tables.py         | Functions to create the fact and dimension tables for the star schema in Redshift.                           |
| dwh.cfg                  | Contains the parameterization offdhcvv z Redshift, IAM role, AWS access and S3.                                      |
| etl.py                   | Contains the queries for loading the S3 data into staging tables on Redshift and then process that data into our analytics tables on Redshift. |
| redshift_start.py        | Contains the necessary processes to create and start up our cluster.                                         |
| redshift_stop.py         | Contains the necessary processes to stop and delete our cluster.                                             |
| sql_queries.py           | Contains the SQL statements that will be implemented for the creation of the tables and for the etl process. |
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
![alt text][image5]


## Preamble

In this project we are going to use two Amazon Web Services, S3 (Data storage) and Redshift (Data warehouse with columnar storage).


## Datasets

Data sources are provided by two public S3 buckets. One bucket contains info about songs and artists, the second has info concerning actions done by users (which song are listening, etc.. ). The objects contained in both buckets are JSON files. The song bucket has all the files under the same directory but the event ones don't, so we need a descriptor file (also a JSON) in order to extract data from the folders by path. We used a descriptor file because we don't have a common prefix on folders.

For this project, you'll be working with two datasets. Here are the s3 links for each:

    * Log data: ```s3://udacity-dend/log_data```
    * Song data: ```s3://udacity-dend/song_data```

The Redshift service is where data will be ingested and transformed, in fact though COPY command we will access to the JSON files inside the buckets and copy their content on our staging tables.


## Configuring the DAG

In the DAG, we add default parameters according to these guidelines

- The DAG does not have dependencies on past runs
- On failure, the task are retried 3 times
- Retries happen every 5 minutes
- Catchup is turned off
- Do not email on retry


## Building the operators

We build four different operators that will stage the data, transform the data, and run checks on data quality.

We utilize Airflow's built-in functionalities as connections and hooks as much as possible and let Airflow do all the heavy-lifting when it is possible.

All of the operators and task instances will run SQL statements against the Redshift database. However, we build flexible, reusable, and configurable operators you can later apply to many kinds of data pipelines with Redshift and with other databases.
