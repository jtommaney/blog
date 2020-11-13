## DuckDB Profiling 1: Scaling Data and Threads

General test scenario:  create TPC-H schemas at scale factor (SF) 10, 30, 100, and 300.  
SF-100 is 150 million orders, with 600 million line items and about 100GB raw data.  

Use pragma threads = 8; syntax to change then number of threads working on a given query.  

* Threads in [ 1,2,4,8,16 ]
* TPC-H Queries range [ 1...22]
* TPC-H Scale Factor in [ 10,30,100,300 ]

## Notes on setting up this test scenario
1) This is all "index free", just using the power of columnar vector scans in parallel.  Very simple to use, create table -> run query.
2) Load is extremely fast.  At 100GB load is about 25 minutes. 
3) Data was loaded by day to enable data skipping. For example a filter on l_shipdate for a given month can ignore remaining 83 months.
4) Overall at least 12x faster than a tuned MySQL instance at SF10. Q1 for example is 176.5 seconds with MySQL, and 1.26 with DuckDB.  


Below are TPC-H queries 1-22 running at SF 10, 30, and 100, using 1, 2, 4, 8, or 16 threads. 

![](https://github.com/jtommaney/blog/blob/blog/assets/DuckDB_SF10_Scaling.png?raw=true) 

![](https://github.com/jtommaney/blog/blob/blog/assets/DuckDB_SF30_Scaling.png?raw=true) 

![](https://github.com/jtommaney/blog/blob/blog/assets/DuckDB_SF100_Scaling.png?raw=true) 

## Analysis:
1) Generally consistent behaviors across scale factors.  A query that runs at 10GB of data will also run at 100GB.
   What this means:  few "gotchas" where the query works fine until the data grows.  
2) Examples of "Pretty Good" Linear Scaling include Q1, Q3,Q4 and Q7. Query 1 is 3.77x faster at SF100 with 4 threads, and 7.3x faster with 8 threads. 
3) The next set of 9 queries averages 2.48x faster with 4 threads, and 3.23x faster with 8 threads (diminishing returns).
4) The remaining 9 queries have limited scaling with additional threads.  They average 1.39x faster with 4 threads and 1.42x faster with 8 threads.



	
