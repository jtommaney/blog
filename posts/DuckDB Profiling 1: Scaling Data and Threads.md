## DuckDB Profiling 1: Scaling Data and Threads

General test scenario:  create TPC-H schemas at scale factor (SF) 10, 30, 100, and 300.  
SF-100 is 150 million orders, with 600 million line items and about 100GB raw data.  

Use pragma threads = 8; syntax to change then number of threads working on a given query.  

* Threads in [ 1,2,4,8,16 ]
* TPC-H Queries range [ 1...22]
* TPC-H Scale Factor in [ 10,30,100,300 ]

Below are TPC-H queries 1-22 running at SF 10, 30, and 100, using 1, 2, 4, 8, or 16 threads. 

![](https://github.com/jtommaney/blog/blob/blog/assets/DuckDB_SF10_TPCH.png?raw=true) 

![](https://github.com/jtommaney/blog/blob/blog/assets/DuckDB_SF30_TPCH.png?raw=true) 

![](https://github.com/jtommaney/blog/blob/blog/assets/DuckDB_SF100_TPCH.png?raw=true) 
