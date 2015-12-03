## HELLO HIVE

### Get Sample Data

[root@sandbox dev]# wget https://s3.amazonaws.com/hw-sandbox/tutorial1/NYSE-2000-2001.tsv.gz
[root@sandbox dev]# hadoop fs -put NYSE-2000-2001.tsv.gz /tmp

### Create Text Table

    CREATE TABLE `nyse_stocks`(
      `exchange` string,
      `stock_symbol` string,
      `date` string,
      `stock_price_open` double,
      `stock_price_high` double,
      `stock_price_low` double,
      `stock_price_close` double,
      `stock_volume` bigint,
      `stock_price_adj_close` double)
    ROW FORMAT DELIMITED
      FIELDS TERMINATED BY '\t'
    STORED AS INPUTFORMAT
      'org.apache.hadoop.mapred.TextInputFormat';

### Create ORC Table

    CREATE TABLE `nyse_stocks_orc`(
      `exchange` string,
      `stock_symbol` string,
      `date` string,
      `stock_price_open` double,
      `stock_price_high` double,
      `stock_price_low` double,
      `stock_price_close` double,
      `stock_volume` bigint,
      `stock_price_adj_close` double) STORED AS orc tblproperties ("orc.compress"="NONE");

    CREATE TABLE `nyse_stocks2`(
      `exchange` string,
      `stock_symbol` string,
      `date` string,
      `stock_price_open` double,
      `stock_price_high` double,
      `stock_price_low` double,
      `stock_price_close` double,
      `stock_volume` bigint,
      `stock_price_adj_close` double); 

### Load data from HDFS file

    LOAD DATA INPATH '/tmp/NYSE-2000-2001.tsv.gz' OVERWRITE INTO TABLE nyse_stocks2;

    Loading data to table default.nyse_stocks2
    Table default.nyse_stocks2 stats: [numFiles=1, numRows=0, totalSize=11325505, rawDataSize=0]
    OK
    Time taken: 0.826 seconds


### Describe metadata of table and preview data

    describe formatted nyse_stocks2;
	select * from nyse_stocks2 limit 5;

      

### Copy data from staging TEXT table to ORC table
  

      INSERT INTO TABLE nyse_stocks_orc select * from nyse_stocks;

### JSON STEPS


    mkdir /user/hue/tweets
     
    upload tweets.txt into /user/hue/tweets
     
    DROP TABLE IF EXISTS JSON_STRING_TABLE;
    CREATE EXTERNAL TABLE JSON_STRING_TABLE (JSON STRING)
    LOCATION '/user/hue/tweets/';
    
    SELECT DISTINCT get_json_object(JSON, '$.user.name') AS usrname,
      get_json_object(JSON, '$.user.userlocation') AS usrlocation,
    get_json_object(JSON, '$.tweetmessage') AS text
    FROM JSON_STRING_TABLE;
    
    SELECT name, screenname, tweetmessage FROM JSON_STRING_TABLE a 
    LATERAL VIEW json_tuple(a.json, 'user', 'tweetmessage') b AS usr, tweetmessage
    LATERAL VIEW json_tuple(usr, 'name', 'screenname') c AS name, screenname;
    
    
###JsonSerDe
    
    DROP TABLE IF EXISTS tweets;
     
    CREATE EXTERNAL TABLE tweets (
      createddate string,
      geolocation string,
      tweetmessage string,
      user struct<geoenabled:boolean, id:int, name:string, screenname:string, userlocation:string>)
    ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
    LOCATION '/user/hue/tweets/';
     
    SELECT DISTINCT tweetmessage, user.name, createddate 
    FROM tweets WHERE user.name = 'Hortonworks'
    ORDER BY createddate;

 
###Custom UDF

#### HDP 2.3 ONLY, user is a reserved word, must use set command below with 

upload JAR file into /tmp/udfs on hdfs

    SET hive.support.sql11.reserved.keywords=false;
    
    SELECT "California" as State;
    
    CREATE TEMPORARY FUNCTION getRegionUS AS 'com.hortonworks.hive.SimpleUDFgetRegionUS'
    USING JAR 'hdfs:///tmp/udfs/HiveSimpleUdf-1.0-SNAPSHOT.jar';
    
    LIST JARS;
    
    SELECT "California" as State, getRegionUS("California") as Region;
    
    DROP FUNCTION getRegionsUS;
    
    CREATE FUNCTION getRegionUS AS 'com.hortonworks.hive.SimpleUDFgetRegionUS' 
    USING JAR 'hdfs:///tmp/udfs/HiveSimpleUdf-1.0-SNAPSHOT.jar';
    
    LIST JARS;
    
    SELECT DISTINCT getRegionUS(split(user.userlocation, ",")[0]) FROM TWEETS;

> Written with [StackEdit](https://stackedit.io/).





