# tpch-Spark

This is a test including a simple implementation that runs a TPC-H-like benchmark on Spark.

It builds on the official TPC-H benchmark available at http://tpc.org/tpch/default.asp (`dbgen` to generate dataset ).

## 1.  首先，TPC-H 是什么？



## 2. Setup
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
$ nano Makefile
```
and modify it so that it contains this (around line 110)

```
CC=gcc
DATABASE=ORACLE
MACHINE=LINUX
WORKLOAD=TPCH
```
and compile it using make as usual. Now you should have dbgen and qgen tools that generate data and queries.

The DBGEN tool must be compiled. First rename `makefile.suite` to makefile and change the following settings in it
```
CC = gcc
DATABASE= SQLSERVER
MACHINE = LINUX
WORKLOAD = TPCH
```
Then you can run

```
make
```

## 3. Generating data
Right, so let's generate the data using the dbgen tool - there's one important parameter 'scale' that influences the amount of data. It's roughly equal to number of GB of raw data, so to generate 10GB of data just do

```
$ ./dbgen -s 10
```