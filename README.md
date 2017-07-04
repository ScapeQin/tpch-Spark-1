# tpch-Spark

This is a test including a simple implementation that runs a TPC-H-like benchmark on Spark.

It builds on the official TPC-H benchmark available at http://tpc.org/tpch/default.asp (`dbgen` to generate dataset ).

## 1.  首先，TPC-H 是什么？



## 2. Setup TPC-H benchmark

### 2.1 下载最新版TPC-H
First, download the TPC-H benchmark from http://tpc.org/tpch/default.asp and extract it to a directory

```
$ wget http://tpc.org/tpch/spec/tpch_2_17_2.tgz
$ mkdir tpch
$ tar -xzf tpch_2_17_2.tgz -C tpch
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

### 2.2 使用本repo 自带的旧版TPC-H
Then you can run

```
make
```

## 3. Generating data
Right, so let's generate the data using the dbgen tool - there's one important parameter 'scale' that influences the amount of data. It's roughly equal to number of GB of raw data, so to generate 10GB of data just do

```
$ ./dbgen -s 10
```

```
[root@imysql tpch]# time ./dbgen -s 50
TPC-H Population Generator (Version 2.9.0)
Copyright Transaction Processing Performance Council 1994 - 2008

real    192m43.897s
user    37m45.398s
sys     19m4.132s
```

```
[root@imysql tpch]# ls -lh *tbl
-rw-r--r-- 1 root root 1.2G Sep 21 15:23 customer.tbl
-rw-r--r-- 1 root root 1.4G Sep 21 15:23 lineitem.tbl
-rw-r--r-- 1 root root 2.2K Sep 21 15:23 nation.tbl
-rw-r--r-- 1 root root 317M Sep 21 15:23 orders.tbl
-rw-r--r-- 1 root root 504K Sep 21 15:23 partsupp.tbl
-rw-r--r-- 1 root root 464K Sep 21 15:23 part.tbl
-rw-r--r-- 1 root root  389 Sep 21 15:23 region.tbl
-rw-r--r-- 1 root root  69M Sep 21 15:23 supplier.tbl
```
dbgen参数 -s 的作用是指定生成测试数据的仓库数，建议基准值设定在100以上，在我的测试环境中，一般都设定为1000。

##  load into HDFS
After the dataset is generated, they need to be loaded in to Hadoop distributed file system (HDFS).
There is a script for doing that under ./data directory. But first you have to move all the dataset to that directory.
```
mv ../../../tpch_2_17_0/dbgen/*.tbl ./data   
```

Then you can upload them to HDFS by execute the following command:

```
$./tpch_prepare_data.sh
```

```
/usr/bin/hadoop fs -mkdir /tpch/ 

/usr/bin/hadoop fs -mkdir /tpch/customer
/usr/bin/hadoop fs -mkdir /tpch/lineitem
/usr/bin/hadoop fs -mkdir /tpch/nation
/usr/bin/hadoop fs -mkdir /tpch/orders
/usr/bin/hadoop fs -mkdir /tpch/part
/usr/bin/hadoop fs -mkdir /tpch/partsupp
/usr/bin/hadoop fs -mkdir /tpch/region
/usr/bin/hadoop fs -mkdir /tpch/supplier

/usr/bin/hadoop fs -copyFromLocal customer.tbl /tpch/customer/
/usr/bin/hadoop fs -copyFromLocal lineitem.tbl /tpch/lineitem/
/usr/bin/hadoop fs -copyFromLocal nation.tbl /tpch/nation/
/usr/bin/hadoop fs -copyFromLocal orders.tbl /tpch/orders/
/usr/bin/hadoop fs -copyFromLocal part.tbl /tpch/part/
/usr/bin/hadoop fs -copyFromLocal partsupp.tbl /tpch/partsupp/
/usr/bin/hadoop fs -copyFromLocal region.tbl /tpch/region/
/usr/bin/hadoop fs -copyFromLocal supplier.tbl /tpch/supplier/

```
查看导入情况：

After running the script, you can check the data on HDFS with the following command:

```
$hadoop fs -ls /tpch
```


TPC-H queries implemented in Spark using the DataFrames API. Tested under Spark 2.0.0

## Running

First compile using:

```
sbt package
```
Make sure you set the INPUT_DIR and OUTPUT_DIR in TpchQuery class before compiling to point to the location the of the input data and where the output should be saved.

You can then run a query using:

```
spark-submit --class "main.scala.TpchQuery" --master MASTER target/scala-2.11/spark-tpc-h-queries_2.11-1.0.jar ##
```

where `##` is the number of the query to run `e.g 1, 2, ..., 22` and `MASTER` specifies the spark-mode `e.g local, yarn, standalone` etc...



# 其他-记录

## 命令行记录

[root@host1 20376-hdfs-NAMENODE]# hdfs haadmin -getServiceState namenode1
standby
[root@host1 20376-hdfs-NAMENODE]# hdfs haadmin -getServiceState namenode2
active

## Reference
- https://github.com/ssavvides/tpch-spark
- https://github.com/kj-ki/tpc-h-impala
- https://github.com/tvondra/pg_tpch
- https://github.com/gregrahn/tpch-kit