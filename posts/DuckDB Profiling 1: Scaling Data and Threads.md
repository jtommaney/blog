## DuckDB Profiling 1:TLDR:
* Excellent usability, performance, and scaling across many query, data, and parallelism conditions.
* Opportunities to improve linear scaling, and to better handle data growth for specific queries. 
* DuckDB continues to improve quickly, these notes are with DuckDB 0.2.2

## DuckDB Profiling 1: Scaling Data and Threads

General test scenario:  create TPC-H schemas at scale factor (SF) 10, 30, and 100.  
SF-100 is 150 million orders, with 600 million line items and about 100GB raw data.  

Use pragma threads = 8; syntax to change then number of threads working on a given query.  

* Threads in [ 1,2,4,8,16 ]
* TPC-H Queries range [ 1...22]
* TPC-H Scale Factor in [ 10,30,100,300 ]

## Notes on this test scenario
1) This is all "index free", just using the power of columnar vector scans in parallel.  Very simple to use, create table -> run query.
2) Load is extremely fast.  At 100GB load is about 25 minutes. 
3) Data was loaded by day to enable data skipping. For example a filter on l_shipdate for a given month can ignore remaining 83 months.
4) Overall at least 12x faster than a tuned MySQL instance at SF10. Q1 for example is 176.5 seconds with MySQL, and 1.26 with DuckDB.  


Below are TPC-H queries 1-22 running at SF 10, 30, and 100, using 1, 2, 4, 8, or 16 threads. 

![](https://github.com/jtommaney/blog/blob/blog/assets/DuckDB_SF10_Scaling.png?raw=true) 

![](https://github.com/jtommaney/blog/blob/blog/assets/DuckDB_SF30_Scaling.png?raw=true) 

![](https://github.com/jtommaney/blog/blob/blog/assets/DuckDB_SF100_Scaling.png?raw=true) 

## Thread-Scaling Analysis - What happens as parallelism changes:
1) Generally consistent behaviors across scale factors.  A query that runs at 10GB of data will also run at 100GB.
2) Examples of "Pretty Good" Linear Scaling include Q1, Q3,Q4 and Q7. Query 1 is 3.77x faster at SF100 with 4 threads, and 7.3x faster with 8 threads. 
3) The next set of 9 queries averages 2.48x faster with 4 threads, and 3.23x faster with 8 threads (diminishing returns).
4) The remaining 9 queries have limited scaling with additional threads.  They average 1.39x faster with 4 threads and 1.42x faster with 8 threads.

![](https://github.com/jtommaney/blog/blob/blog/assets/speedup_at_sf100.png?raw=true)



## Data-Scaling Analysis - What happens as data size changes:  
1) There is significant variability in query behavior as data size grows.  These are with 8 threads in parallel.   
 - Positive example:  Q1 (no joins) runs in 6.59 seconds at SF10 and 12.49 seconds at SF100 
 - Negative example:  Q5 (8 table join) runs in 0.18 seconds at SF10 and 17.53 seconds at SF100
 - Negative example:  Q17 (3 table join including subquery) runs in 0.39 seconds at SF10 and 51.92 seconds at SF100 

2) Potential causes for future investigation:
    - a sub-optimal plan with an N-squared compontent  
    - change in query plan

![](https://github.com/jtommaney/blog/blob/blog/assets/Scaling_from_10_to_100.png?raw=true)	


’’’
SELECT /* Q5 */ n1.n_name, 
       Sum(l_extendedprice * ( 1 - l_discount )) AS revenue 
  FROM customer, 
       orders, 
       lineitem, 
       supplier, 
       nation n1, 
       region r1, 
       nation n2, 
       region r2 
 WHERE c_custkey = o_custkey 
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
’’’
SELECT sum(l_extendedprice)/7.0 as avg_yearly 
  FROM lineitem, 
       part 
 WHERE p_partkey = l_partkey 
       AND p_brand = 'Brand#23' 
       AND p_container = 'MED BOX' 
       AND l_quantity < (SELECT 0.2*avg(l_quantity) 
                           FROM lineitem 
                          WHERE l_partkey = p_partkey); 
’’’




	
