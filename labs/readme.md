
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