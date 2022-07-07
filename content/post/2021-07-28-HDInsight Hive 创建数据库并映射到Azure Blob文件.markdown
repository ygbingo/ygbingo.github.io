---
layout:     post
title:      Azure Blob CSV to HDInsight
subtitle:   HDInsight Hive 创建数据库并映射到Azure Blob文件
date:       2021-07-28
author:     Bingo
header-img: img/post-bg-HDInsight.png
catalog: true
tags:
    - Azure
    - Hive
    - HDInsight
---

> 最近需要搭建一套数据仓库，已经有了csv的数据源文件，怎么能直接把它映射到hdfs中呢？

# 准备
- [Azure Blob](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction)容器
- [Azure HDInsight](https://docs.microsoft.com/en-us/azure/hdinsight/hdinsight-overview)集群

# 创建blob目录
1. 创建HDInsight集群时指定已有的blob容器，保证在HDInsight中能访问到对应的blob
2. 在Azure Blob中创建一个容器，例如：hd-container
3. ssh链接到HDInsight，假设HDInsight地址为bingo-hdinsight.azurehdinsight.cn
    ```shell
    ssh sshuser@****-ssh.azureinsight.cn
    ```
4. 创建hadoop文件目录
    ```shell
    hadoop fs -mkdir wasbs://hd-container@bingo-hdinsight.blob.core.chinacloudapi.cn/
    ```
5. 查看该路径下文件内容
    ```shell
    hadoop fs -ls wasbs://hd-container@bingo-hdinsight.blob.core.chinacloudapi.cn/
    ```
6. 添加hive的访问权限（防止hive访问path时出现permission denied）
    ```shell
    hdfs dfs -chown hive wasbs://hd-container@bingo-hdinsight.blob.core.chinacloudapi.cn/
    ```

# 创建hive分区表
1. 在HDInsight的中控台登录Ambari
    - 初始用户名
        - username: admin
        - password: ******
    ![](https://docs.microsoft.com/zh-cn/azure/hdinsight/hadoop/media/apache-hadoop-linux-create-cluster-get-started-portal/hdinsight-linux-get-started-open-cluster-dashboard.png)

2. 进入Hive View 2.0控制台
3. 创建一个database用来存表
    ```sql
    CREATE DATABASE bingoDB;
    ```
4. 创建一个按`date`分区的表
    ```sql
    CREATE EXTERNAL TABLE bingodb.`bingotable_ex`(
    `timestamp` string COMMENT 'timestamp in dm-log',
    `requestcontent` string COMMENT 'request content',
    `responsecontent` string COMMENT 'response content',
    `answerfeed` string COMMENT 'whitch answerfeed',
    `fromusername` string COMMENT 'username',
    `workflowname` string COMMENT 'whitch workflow',
    `sessionid` string COMMENT 'session id')
    PARTITIONED BY (`date` string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
    STORED AS TEXTFILE LOCATION 'wasbs://hd-container@bingo-hdinsight.blob.core.chinacloudapi.cn/'
    TBLPROPERTIES('skip.header.line.count'='1');
    ```
    - CREATE EXTERNAL TABLE: 创建一个外部表，这样当表被删除时，数据文件还是能被保留；
    - PARTITIONED BY: 用来分区的字段；
    - ROW FORMAT DELIMITED FIELDS TERMINATED BY：数据文件中字段用来分割字段的符号，比如我的数据文件格式为csv，中间用","分割；
    - STORED AS TEXTFILE LOCATION：数据文件的存储位置，即上面hadoop创建的hdfs文件路径；
    - TBLPROPERTIES('skip.header.line.count'='1'): 数据文件的首行为标题，载入时自动跳过文件首行

# 载入数据文件

1. 方法一：把文件通过ssh传输的方式传到hdinsight集群中，然后执行hql语句
    ```sql
    LOAD DATA LOCAL INPATH "/home/sshuser/data/data1.csv" TO bingodb.`bingotable_ex` WHERE `date`="2021-02-01";
    ```
2. 方法二：在blob的hd-container容器中创建一个文件夹```date=2021-02-02```，对应表的分区为"2021-02-02"，然后把csv文件直接导入该表格中，通过hql查看结果
    ```sql
    SELECT * FROM bingodb.`bingotable_ex` WHERE `date`="2021-02-02";
    ```