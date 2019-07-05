# BDE_3_TestCluster.md
    20190704-DataEngineering (Con't) 
    ToC
        ## Let’s test out our cluster 
        [0. scp SQL files]
        [1. Create user “training” in linux and in hdfs]
        [2. In MySQL create the sample tables that will be used for the rest of the test]
        [3. Extract tables authors and posts from the database and create Hive tables]
        [4. Create and run a Hive/Impala query]
        [5. Export the data from above query to MySQL]
        
## Let’s test out our cluster 

### [0. scp SQL files]
    authors-23-04-2019-02-34-beta.sql.zip
    posts23-04-2019-02-44.sql.zip

    # scp  -i ./SKT.pem ./authors-23-04-2019-02-34-beta.sql.zip centos@13.209.93.133:/home/centos
    # scp  -i ./SKT.pem ./posts23-04-2019-02-44.sql.zip centos@13.209.93.133:/home/centos
    [root@cm centos]# ls -al *.sql
    -rwxrwxrwx 1 root root   892780 Apr 23 02:34 authors-23-04-2019-02-34-beta.sql
    -rwxrwxrwx 1 root root 52680682 Apr 23 02:44 posts23-04-2019-02-44.sql
    # mysql -u root -p
    CREATE DATABASE test DEFAULT CHARACTER SET utf8  DEFAULT COLLATE utf8_general_ci;
    GRANT ALL ON test.* TO 'training'@'%' IDENTIFIED BY 'Admin123!';
    # mysql -u training -p
    # source authors-23-04-2019-02-34-beta.sql
    # source posts23-04-2019-02-44.sql
    
### [1. Create user “training” in linux and in hdfs]
    useradd training -G hadoop 
    passwd training 
    hdfs dfs -mkdir /user/training
    hdfs dfs -chown training:hadoop /user/training

### [2. In MySQL create the sample tables that will be used for the rest of the test]
    mysql -u root -p
    CREATE DATABASE test DEFAULT CHARACTER SET utf8  DEFAULT COLLATE utf8_general_ci;
    GRANT ALL ON *.* TO 'training'@'%' IDENTIFIED BY 'Admin123!';

    mysql -u training -p
    source authors-23-04-2019-02-34-beta.sql
    source posts23-04-2019-02-44.sql

### [3. Extract tables authors and posts from the database and create Hive tables]
    #check 
    beeline -u jdbc:hive2://cm.skplanet.com:10000 -n hive

    authors, posts

    sqoop import --connect jdbc:mysql://cm.skplanet.com:3306/test \
    --username training  \
    -P training   \
    --table authors     \
    --target-dir /user/training   \
    --fields-terminated-by ","    \
    --hive-import    \
    --create-hive-table    \
    --hive-table authors

### [4. Create and run a Hive/Impala query]

#### [sqoop]
    sqoop import --connect jdbc:mysql://cm:3306/test --username training --password Hadoop123! --table authors  --driver com.mysql.jdbc.Driver --target-dir /user/training/authors --hive-import --hive-table test.authors 
    sqoop import --connect jdbc:mysql://cm:3306/test --username training --password Hadoop123! --table posts  --driver com.mysql.jdbc.Driver --target-dir /user/training/posts --hive-import --hive-table test.posts

    CREATE External TABLE `authors`(
      `id` int,
      `first_name` string,
      `last_name` string,
      `email` string,
      `birthdate` string,
      `added` string)
    COMMENT 'Imported by sqoop on 2019/07/04 05:42:59'
    ROW FORMAT SERDE
      'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
    WITH SERDEPROPERTIES (
      'field.delim'='\u0001',
      'line.delim'='\n',
      'serialization.format'='\u0001')
    STORED AS INPUTFORMAT
      'org.apache.hadoop.mapred.TextInputFormat'
    OUTPUTFORMAT
      'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
    LOCATION
      'hdfs://master1.skplanet.com:8020/user/training/authors'
    TBLPROPERTIES (
      'COLUMN_STATS_ACCURATE'='true',
      'numFiles'='4',
      'totalSize'='760827',
      'transient_lastDdlTime'='1562218982')

#### [impala]
    1) shell실행
     명령어 : impala-shell 
    2) 접속후 하이브 테이블 메타정보 갱신
      INVALIDATE METADATA 
    3) 테이블 데이터 확인
     show databases;
     use test;
     select * from authors limit 10;
     
### [5. Export the data from above query to MySQL]

#### [export sqoop]
 
###### hive query to hdfs 
        INSERT OVERWRITE DIRECTORY '/user/training/results'
        select a.id, a.fname, a.Lname, count(a.pid)
        from (
            select authors.id as id, authors.first_name as fname, authors.last_name as Lname, posts.id as pid
          from authors, posts
          where authors.id = posts.author_id
        ) a
        group by a.id, a.fname, a.Lname;

###### create mysql table (test.results)
        create table test.results (
          id varchar(100)
          , fname varchar(100)
          , Lname varchar(100)
          , num_posts int
        );

##### hdfs to mysql (sqoop)
        sqoop export --connect jdbc:mysql://cm:3306/test --username training --password Hadoop123! --driver com.mysql.jdbc.Driver --table test.results --export-dir /user/training/results --input-fields-terminated-by '\0001'


(end of file)
