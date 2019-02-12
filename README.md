# recap-hive

##### Introduction
* Hive is a datawarehouse application.
* Used to run batch queries on structured data.
* Can handle large data sets reliably.
* Allows to run MapReduce jobs without learnig java code.
* Hive is good for structured data like tables. Unstructured data such as Json may require some manipulation before it is usable in Hive.
* Hive is commonly used for batch queries and we can write batch scripts.
* Once Hive query is completed, the results will be either submitted to the command line or UI or user can store into another table or as .csv file for later use. These exported results can be loaded back into HDFS.

##### DATA TYPES
* ALL Java Primitive Types
* Array<datatype>
* Map<primitive,datattype>
* Struct<columnName,datatype,.....>

##### Databases and Tables
* We can create multiple databaases inside hive.
* while crating table if we don't specify a database, it will use **default** database.
* Hive itself doesn't contain data, all data is stored in HDFS.
* When we create database/table, all the meta data is stored in hive meta store.
* Hive is case-insenstivie
* **Creating a database**
    ```
    create database cmdb 
    LOCATION '/home/neerajm/hive/cmdb.db'
    COMMENT 'contains change management database';
    ```
* show databases;
* use database;
* **Create table**

```
Create table employee(firstName String,lastName String, dob Date,height Float);

```

* Create table parameters
 ```
 [FIELDS|ROWS] TERMINATED BY <delimeter> eg comma,space,tab,newline,dollarsign,hashes,semicolon etc
 ```
 
 ```
 STORED AS <FILEFORMAT>
 - TEXTFILE
 - SEQUENCEFILE
 - RCFILE (Flipped Matrix )
 - ORC  (Same feature of RC, but with additional headers and footers for metadata and indexing)(Due to ordered nature hive can skip records by reading meta data, thus increasing speed)
 - PARQUET
 ```
 
 ```
 OUTPUTFORMAT`
 ```
 ```
 TBLPROPERTIES ('key'='value',...)
 ```
* show tables; or show tables in <database_name>
* describe <table_name>
* show create table <table_name>

###### EXTERNAL TABLE
* External tables are linked directly to an external file.
* These are read only tables.
* Data is loaded into table simultaneously with create command itself.
```
CREATE TABLE CMDB.instance(instance_id INT,instance_name STRING,date_created DATE)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
STORED AS TEXTFILE
LOCATION '/home/neerajm/instance.csv'
```
###### PARTIONING
* Partitioning divides data into logically organised chunks.
* After creating table structure, we load data seprately for each partition.
* Partioned column is not part of table column, a directory is created for each unique partition value.
* This may result in fast processing time.
```
CREATE TABLE CMDB.instance(instance_id INT,instance_name STRING,date_created DATE)
PARTITIONED BY (region STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
STORED AS TEXTFILE
LOCATION '/home/neerajm/instance.csv'
```

###### BUCKETING
* Bucketing divides data into evenly-sized chunks by hashing.
```
CREATE TABLE CMDB.instance(instance_id INT,instance_name STRING,date_created DATE)
CLUSTERED BY (region STRING) INTO 10 BUCKETS;
```

###### LOADING DATA INTO TABLES
```
LOAD DATA LOCAL INPATH
'/home/neerajm/hive/input/data/instance.csv'
INTO TABLE CMDB.instance;
```
* Loading Partition Data
```
LOAD DATA LOCAL INPATH
'/home/neerajm/hive/input/data/instance.csv'
INTO TABLE CMDB.instance PARTITION(region='EMEA');
```

###### DROP/DELETE
* DROP TABLE;
  * **Note that dropping a table does not necessarily delete your data files. Your data is typically stored on your HDFS, and will remain there unaltered.This is true even if you delete an external table**
* DROP DATABASE; // TO DELETE EMPTY DATABASE
###### ALTER TABLE
* Can rename table name
* Can change column name

```
ALTER TABLE CMDB.INSTANCE RENAME TO CMDB.LOCAL_INSTANCE;
ALTER TABLE CMDB.INSTANCE CHANGE COLUMN instance_name name String;
```

###### QUERYING DATA
* Similar to SQL
* EXPLAIN ouput query plan for the MapReduce jobs

###### CREATE TABLE
* CTAS save query into a table.
```
CREATE TABLE EMEA_INSTANCE AS SELECT
.....
```

###### etc

* QUIT; exit hive shell
* hive -e to run inline query
```
hive -e 'Select *  from employee' > '/home/neerajm/emp_data/emp.csv'
```

##### SerDe
* A SerDe allows Hive to read in data from a table, and write it back out to HDFS in any custom format
* IO interface to seriaize/decerialize data into hive tables.
* Built-in SerDes
     *   Avro (Hive 0.9.1 and later)
     *   ORC (Hive 0.11 and later)
     *   RegEx
     *   Thrift
     *   Parquet (Hive 0.13 and later)
     *   CSV (Hive 0.14 and later)
     *   JsonSerDe (Hive 0.12 and later in hcatalog-core)

```hdfs dfs -put sample.csv /tmp/serdes/```
* CSV file
```
drop table if exists sample;
create external table sample(id int,first_name string,last_name string,email string,gender string,ip_address string)
  row format serde 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
  stored as textfile
location '/tmp/serdes/';
```
* TSV file
```
drop table if exists sample;
create external table sample(id int,first_name string,last_name string,email string,gender string,ip_address string)
  row format serde 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
with serdeproperties (
  "separatorChar" = "\t"
  )
  stored as textfile
location '/tmp/serdes/';
```

##### Map Join
* It allows a table to be loaded into memory so that a (very fast) join could be performed entirely within a mapper without having to use a Map/Reduce step.

``` SELECT /*+ MAPJOIN(c) */ * FROM orders o JOIN cities c ON (o.city_id = c.id); ```

``` hive.auto.convert.join ```
``` Hive will automatically use mapjoins for any tables smaller than hive.mapjoin.smalltable.filesize (default is 25MB) ```
* Mapjoins have a limitation in that the same table or alias cannot be used to join on different columns in the same query.
