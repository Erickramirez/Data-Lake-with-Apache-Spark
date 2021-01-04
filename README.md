# Data-Lake-with-Apache-Spark
This project has as output a Data Lake solution. It building an ETL pipeline that extracts their data from S3, processes them using Spark, and loads the data back into S3 as a set of dimensional tables. This will allow their analytics team to continue finding insights in what songs their users are listening to.

Sparkify is a fictional popular digital media service created by Udacity, similar to Spotify or 
Pandora; many users use their services every day.
   
 ### Prerequisites
The environment needed for this project:
1. [Python 3.6](https://www.python.org/downloads/release/python-360/)
2. [PySpark](https://spark.apache.org/docs/latest/api/python/index.html)
3. [AWS account](https://aws.amazon.com/)
    - [IAM User](https://console.aws.amazon.com/iam) with `Programmatic access` and permission to write on S3.
    - [S3 bucket](https://aws.amazon.com/es/s3/)
4. [Apache Spark](https://spark.apache.org/), you can use one of the following configurations:
    - Stand along installation, check [Apache Spark documentation](https://spark.apache.org/downloads.html) 
    - [Amazon EMR Cluster](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-gs-launch-sample-cluster.html)

### Explanation of the files in the repository
1. **etl.py:** reads data from S3, processes that data using Spark, and writes them back to S3.
2. **dl.cfg:** configuration file that contains AWS credentials: 
```
[CREDENTIALS]
AWS_ACCESS_KEY_ID = <your aws key>
AWS_SECRET_ACCESS_KEY = <your aws secret>
```
5. **images:** folder that contains the images used in this file.


### Instructions to run the project
1. clone the github repository: `git clone https://github.com/Erickramirez/Sparkify_Data-Lake-with-Apache-Spark.git`
2. verify the Prerequisites
4. Execute ETL process using the command `python etl.py` 

## About the Data Warehouse solution
### Datasets
1. **Song data:** `s3://udacity-dend/song_data`  it is a subset of real data from 
[Million Song Dataset](http://millionsongdataset.com/) it is in JSON format: 
    ```
    {
        "num_songs": 1,
        "artist_id": "ARJIE2Y1187B994AB7",
        "artist_latitude": null,
        "artist_longitude": null,
        "artist_location": "",
        "artist_name": "Line Renaud",
        "song_id": "SOUPIRU12A6D4FA1E1",
        "title": "Der Kleine Dompfaff",
        "duration": 152.92036,
        "year": 0
    }
    ```
2. **Log data:** `s3://udacity-dend/log_data` consists of log files in JSON format 
generated by this [event simulator](https://github.com/Interana/eventsim) based on the songs in the dataset above. 
It has the following structure:  ![log-data](/images/log-data.png)

### Database Schema
It will be necesary to copy the JSON file in the S3 buckets into the staging tables. 
After this staging tables, the data is loaded into dimension and fact tables. 
This table definition is in `etl.py`
1. **Dimension tables**
    - **users:** users in the app - user_id, first_name, last_name, gender, level
    - **songs:** songs in music database - song_id, title, artist_id, year, duration
    - **artists:** artists in music database - artist_id, name, location, lattitude, longitude
    - **time:** timestamps of records in songplays broken down into specific units - start_time, hour, day, week, month, year, weekday
2. **Fact table**
    - **songplays:** records in event data associated with song plays i.e. records with page `NextSong` - 
songplay_id, start_time, user_id, level, song_id, artist_id, session_id, location, user_agent

Each of the five tables are written to parquet files in a separate analytics directory on S3. 
Each table has its own folder within the directory. 
`Songs` table files are partitioned by year and then artist. 
`Time` table files are partitioned by year and month. 
`Songplays` table files are partitioned by year and month.
    
### ETL Pipeline
The data is extracted from JSON files in s3 (s3a://udacity-dend/) for the explained `Datasets`. It is necessary to transform the data into the `Database Schema`; it is performed with Spark using PySpark. Some of the transformations are related only to select some columns and remove the duplicated data, and in other cases, it is necessary to modify the type; for instance, the `ts` in `log-data` dataset is in Unix format, and that will need to be converted to a date-time format.  The files generated are in parquet format.
The etl for both dataset are in the methods: `process_song_data` and `process_log_data` in the file `etl.py`.