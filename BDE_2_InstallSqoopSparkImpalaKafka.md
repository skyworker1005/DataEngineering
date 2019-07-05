# BDE_2_InstallSqoopSparkImpalaKafka.md
    20190704-DataEngineering (Con't) 
    ToC
        ### [Install Spark2]
        ### [Install Sqoop, Impala, Kafka]

## Install Sqoop, Spark, Impala and Kafka
    
### [Install Spark2]
    link: https://www.cloudera.com/documentation/spark2/latest/topics/spark2.html
    CDS Powered by Apache Spark Overview
    CDS(custom service descriptor)

    1. To download the CDS Powered by Apache Spark service descriptor, in the Version Information table in CDS Versions Available for Download, click the service descriptor link for the version you want to install.

    Available CDS Versions
    Version	        Custom Service Descriptor	        Parcel Repository
    2.4 Release 2	SPARK2_ON_YARN-2.4.0.cloudera2.jar	http://archive.cloudera.com/spark2/parcels/2.4.0.cloudera2/

    2. Log on to the Cloudera Manager Server host, and copy the CDS Powered by Apache Spark service descriptor in the location configured for service descriptor files.
    * scp  -i ./SKT.pem ./SPARK2_ON_YARN-2.4.0.cloudera2.jar centos@13.209.93.133:/home/centos

    3. Set the file ownership of the service descriptor to cloudera-scm:cloudera-scm with permission 644.

    [root@cm csd]# pwd
    /opt/cloudera/csd
    [root@cm csd]# cp /home/centos/SPARK2_ON_YARN-2.4.0.cloudera2.jar ./
    [root@cm csd]# ll
    total 20
    -rw-r--r-- 1 root root 19066 Jul  4 00:49 SPARK2_ON_YARN-2.4.0.cloudera2.jar
    [root@cm csd]# chown cloudera-scm:cloudera-scm SPARK2_ON_YARN-2.4.0.cloudera2.jar 
    [root@cm csd]# ll
    total 20
    -rw-r--r-- 1 cloudera-scm cloudera-scm 19066 Jul  4 00:49 SPARK2_ON_YARN-2.4.0.cloudera2.jar
    [root@cm csd]# 
    [root@cm csd]# chmod 644 SPARK2_ON_YARN-2.4.0.cloudera2.jar 
    [root@cm csd]# ll
    total 20
    -rw-r--r-- 1 cloudera-scm cloudera-scm 19066 Jul  4 00:49 SPARK2_ON_YARN-2.4.0.cloudera2.jar
    [root@cm csd]# 

    4. Restart the Cloudera Manager Server with the following command:
    command: systemctl restart cloudera-scm-server

    5. Download the CDS Powered by Apache Spark parcel, distribute the parcel to the hosts in your cluster, and activate the parcel. See Managing Parcels.
    cm 메뉴 옆에 있는 Parcel menu 에서 spark2 에 대한 설정 진행 


### [Install Sqoop, Impala, Kafka]
    Install them in CM! 
    

(end of file)
