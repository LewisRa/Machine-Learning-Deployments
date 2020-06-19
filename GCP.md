# Data Science Positions
1. Product Analyst/ Product Data scientist
  - Analysts user data to create reports for product managers
  - Typically use coding as main tool but can use Tableau/Qlik
2. Business Analyst/ Business Inytelligence 
  - Creating insigts and analysis for business data (usually use Tableau and Excel)
3. ML Engineer 
  - custom ML models to fit company
  - very technical
4. Data Engineeer
  - Technical role to build data pipelines
# Recommendation Systems Lab
You are in charge of migrating your company's existing machine learning workload for housing. Recommendations from your on premises had do cluster to the cloud organizations happy with the current model. But the underlying infrastructure on premise is costing them headaches to tune and utilize efficiently. You're CTO wants as little friction as possible from your existing Hadoop on Promise Infrastructure, but has heard of the advantages that cloud solutions offer for auto scaling and serverrless management. 

- Create a Cloud sequel instance and populate the tables. 
- Explore the rentals data using SQL statements using Cloud Shell, 
- Launch DataProc 
- Train and apply a machine learning model Written and Price park to create product recommendations. 
- Explore the inserted rows in Cloud SQL.

Here's what the setup will look like in Step one. We set up Google Cloud Storage and Cloud Sequel and then importer records from GCS into clouds equal. And now that your ratings are in clouds equal in step two, you will run a machine learning training job in Cloud Data. Prock to read those ratings and train the machine learning model in step three, you will run the model in cloud data Prock to create recommendations and save the top five recommendations for each user back into clouds sequence. Lastly, step for your ratings can be delivered back to users. Threw up engine. You want two steps one and four in this lab, because those steps deal mostly with Web programming. In this lab, you concentrate on doing steps two and three. Try out the lab and keep in mind that you have multiple attempts for each lab and you can always come back and practice.


1. Data is stored on Cloud Storage (similar to AWS S3)**Move Storage Off Cluster**
2. Used data and spark job on Data Proc to build recommendation system (AWS EMR) 
  - Resize clusters effortlessly
3. Store recommendations in Cloud SQL (AWS RDS)

#### Google's data center bandwidth/network speen enables the separtion of compute and store (Bisection bandwidth)
hdfs:// --> gs://

# Connect to the database
```
CREATE DATABASE IF NOT EXISTS recommendation_spark;

USE recommendation_spark;

DROP TABLE IF EXISTS Recommendation;
DROP TABLE IF EXISTS Rating;
DROP TABLE IF EXISTS Accommodation;

CREATE TABLE IF NOT EXISTS Accommodation
(
  id varchar(255),
  title varchar(255),
  location varchar(255),
  price int,
  rooms int,
  rating float,
  type varchar(255),
  PRIMARY KEY (ID)
);

CREATE TABLE  IF NOT EXISTS Rating
(
  userId varchar(255),
  accoId varchar(255),
  rating int,
  PRIMARY KEY(accoId, userId),
  FOREIGN KEY (accoId)
    REFERENCES Accommodation(id)
);

CREATE TABLE  IF NOT EXISTS Recommendation
(
  userId varchar(255),
  accoId varchar(255),
  prediction float,
  PRIMARY KEY(userId, accoId),
  FOREIGN KEY (accoId)
    REFERENCES Accommodation(id)
);

SHOW DATABASES;
```

#### Connect using Cloud Shell, by typing
```
gcloud sql connect rentals --user=root --quiet
```

- SHOW DATABASES;
- **Run above query**
- SHOW DATABASES;
- USE recommendation_spark;
- SHOW TABLES;

# Stage Data in Google Cloud Storage
```
echo "Creating bucket: gs://$DEVSHELL_PROJECT_ID"
gsutil mb gs://$DEVSHELL_PROJECT_ID

echo "Copying data to our storage from public dataset"
gsutil cp gs://cloud-training/bdml/v2.0/data/accommodation.csv gs://$DEVSHELL_PROJECT_ID
gsutil cp gs://cloud-training/bdml/v2.0/data/rating.csv gs://$DEVSHELL_PROJECT_ID

echo "Show the files in our bucket"
gsutil ls gs://$DEVSHELL_PROJECT_ID

echo "View some sample data"
gsutil cat gs://$DEVSHELL_PROJECT_ID/accommodation.csv
```

# Loading Data from Google Cloud Storage into Cloud SQL tables
# Explore Cloud SQL data 
# Generating housing recommendations with Machine Learning using Cloud Dataproc
