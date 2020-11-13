## DuckDB Profiling 1: Scaling Data and Threads

General test scenario:  create TPC-H schemas at scale factor (SF) 10, 30, 100, and 300.  
SF-100 is 150 million orders, with 600 million line items and about 100GB raw data.  

Use pragma threads = 8; syntax to change then number of threads working on a given query.  

* Threads in [ 1,2,4,8,16 ]
* TPC-H Queries range [ 1...22]
* TPC-H Scale Factor in [ 10,30,100,300 ]

## Analysis on setting up this test scenario
1) This is all "index free", just using the power of columnar vector scans in parallel.  Very simple to use, create table -> run query.
2) Load is extremely fast.  At 100GB load is about 25 minutes. 
3) Data was loaded by day to enable data skipping. For example a filter on l_shipdate for a given month can ignore remaining 83 months.  


Below are TPC-H queries 1-22 running at SF 10, 30, and 100, using 1, 2, 4, 8, or 16 threads. 

![](https://github.com/jtommaney/blog/blob/blog/assets/DuckDB_SF10_Scaling.png?raw=true) 

![](https://github.com/jtommaney/blog/blob/blog/assets/DuckDB_SF30_Scaling.png?raw=true) 

![](https://github.com/jtommaney/blog/blob/blog/assets/DuckDB_SF100_Scaling.png?raw=true) 

## Analysis:
1) Consistent query execution across scale factors.  A query that runs at 10GB of data will also run at 100GB.
   What this means:  no "gotchas" where the query works fine until the data grows.  
2) Examples of "Pretty Good" Linear Scaling.  Most queriest get faster with more threads,and cleanest examples are Q2, Q5, Q10, and Q21.  


’’’
SELECT /* Q5 */ n1.n_name, 
       Sum(l_extendedprice * ( 1 - l_discount )) AS revenue 
FROM   customer, 
       orders, 
       lineitem, 
       supplier, 
       nation n1, 
       region r1, 
       nation n2, 
       region r2 
WHERE  c_custkey = o_custkey 
       AND l_orderkey = o_orderkey 
       AND l_suppkey = s_suppkey 
       AND c_nationkey = s_nationkey 
       AND s_nationkey = n1.n_nationkey 
       AND n1.n_regionkey = r1.r_regionkey 
       AND r1.r_name = 'ASIA' 
       AND c_nationkey = n2.n_nationkey 
       AND n2.n_regionkey = r2.r_regionkey 
       AND r2.r_name = 'ASIA' 
       AND o_orderdate >= '1994-01-01' 
       AND o_orderdate < '1995-01-01' 
GROUP  BY n1.n_name 
ORDER  BY revenue DESC; 
’’’
