# tpch-Spark

This is a test including a simple implementation that runs a TPC-H-like benchmark on **Spark 2.0**.

It builds on the official TPC-H benchmark available at http://tpc.org/tpch/default.asp (`dbgen` to generate dataset ).

## 阶段-1
Based on https://github.com/ssavvides/tpch-spark

### 1.1 install sbt on ec2
```
curl https://bintray.com/sbt/rpm/rpm | sudo tee /etc/yum.repos.d/bintray-sbt-rpm.repo
sudo yum install sbt
```

### 1.2 git clone tpch-spark

```
git clone https://github.com/ssavvides/tpch-spark
```

### 1.3 data generation 
Go into /tpch-spark/dbgen folder and `make`. `2` represents 2 GB data.

```
make
cd /tpch-spark
./dbgen -s 2


# check data
ls -lh *tbl
```

### 1.4 add PATH in .bash_profile
```
cd 
vim .bash_profile

# add
export PATH=$PATH:/root/spark/bin/

# add
export PATH=$PATH:/root/persistent-hdfs/bin/

```

### 1.5 build and run 
`01` represents `query 01`. (you can use 01, 02, ... 22). `--master local` specifies the spark-mode `e.g local, yarn, standalone` etc...


```

# compile using:

sbt package

# run 
spark-submit --class "main.scala.TpchQuery" --master local /root/tpch-spark/target/scala-2.11/spark-tpc-h-queries_2.11-1.0.jar 01
```

### 1.6 check result
go in /dbgen floder

```
ls output
```


### 1.7.1 load data into HDFS

```
hdfs dfs -mkdir /tpch/
hdfs dfs -mkdir /tpch/input/

hdfs dfs -put /root/tpch-spark/dbgen/*.tbl /tpch/input/

hdfs dfs -mkdir /tpch/output/
```


After running the script, you can check the data on HDFS with the following command:

```
hdfs dfs -ls /tpch
```

### 1.7.2 generate and load HDFS directly

在文件中修改 TpchQuery.scala input 和 output 的地址。

```
#Make sure you set the INPUT_DIR and OUTPUT_DIR in TpchQuery 

// read from hdfs
val INPUT_DIR: String = "/dbgen"
    
    
val OUTPUT_DIR: String = "/tpch"

```

## 阶段-2. 从 0 开始

### 2.1 下载TPC-H

```
git clone https://github.com/gregrahn/tpch-kit
```

and then prepare the Makefile - create a copy from makefile.suite

```
$ cd tpch/dbgen
$ cp makefile.suite Makefile
$ vim Makefile
```

The DBGEN tool must be compiled. First rename `makefile.suite` to makefile and change the following settings in it

```
CC = gcc
DATABASE= ORACLE or SQLSERVER
MACHINE = LINUX
WORKLOAD = TPCH
```


### 2.2. Generating data in `dbgen` folder
Right, so let's generate the data using the dbgen `tool` - there's one important parameter 'scale' that influences the amount of data. It's roughly equal to number of GB of raw data, so to generate 10GB of data just do

```
$ time ./dbgen -s 2

real    192m43.897s
user    37m45.398s
sys     19m4.132s
```

```
$ ls -lh *tbl
-rw-r--r-- 1 root root 1.2G Sep 21 15:23 customer.tbl
-rw-r--r-- 1 root root 1.4G Sep 21 15:23 lineitem.tbl
-rw-r--r-- 1 root root 2.2K Sep 21 15:23 nation.tbl
-rw-r--r-- 1 root root 317M Sep 21 15:23 orders.tbl
-rw-r--r-- 1 root root 504K Sep 21 15:23 partsupp.tbl
-rw-r--r-- 1 root root 464K Sep 21 15:23 part.tbl
-rw-r--r-- 1 root root  389 Sep 21 15:23 region.tbl
-rw-r--r-- 1 root root  69M Sep 21 15:23 supplier.tbl

$ pwd
/root/tpch-spark/dbgen
```


###  load into HDFS
After the dataset is generated, they need to be loaded in to Hadoop distributed file system (HDFS).
There is a script for doing that under ./data directory. But first you have to move all the dataset to that directory.

```
cd 
mkdir data
mv /root/tpch-spark/dbgen/*.tbl /root/data   
```


## 其他-记录

### commad record

```
hdfs haadmin -getServiceState namenode1
standby
hdfs haadmin -getServiceState namenode2
active

# 想知道公网ip

# 内网ip
ifconfig ／ hostname

# 替换
sudo hostname ec2-54-209-221-112.compute-1.amazonaws.com
```

### log level
Log4J Levels: ALL > TRACE > DEBUG > INFO > WARN > ERROR > FATAL > OFF

DEBUG – The DEBUG Level designates fine-grained informational events that are most useful to debug an application. 

### Reference
- https://github.com/ssavvides/tpch-spark
- https://github.com/kj-ki/tpc-h-impala
- https://github.com/tvondra/pg_tpch
- https://github.com/gregrahn/tpch-kit