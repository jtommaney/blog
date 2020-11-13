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
* TPC-H Scale Factor in [ 10,30,100 ]

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
1) There is some variability in query behavior as data size grows.  These are with 8 threads in parallel.   
 - Most queries (15 of 22) run +/- 10% of expected costs.
 - A few including Q17, Q18, and Q22 take 14.x longer to run on 10x more data.  

2) Potential causes for future investigation. 
    - a sub-optimal query plan? 
    - change in query plan at different scale factor? 
    - Some characteristic of sub-query common to all 3 queries is sub-optimal.

![](https://github.com/jtommaney/blog/blob/blog/assets/Scaling_from_10_to_100.png?raw=true)	


Q17, Q18, Q22 queries below:
```
S
select /*Q17*/ sum(l_extendedprice)/7.0 as avg_yearly 
from lineitem, part 
where p_partkey = l_partkey and p_brand = 'Brand#23' and p_container = 'MED BOX' 
and l_quantity < (select 0.2*avg(l_quantity) 
                    from lineitem 
              where l_partkey = p_partkey); 


select /* Q18 */ c_name, c_custkey, o_orderkey, o_orderdate, o_totalprice, sum(l_quantity) 
from customer, orders, lineitem 
where o_orderkey in (select l_orderkey from lineitem 
		      where l_orderkey group by 1 having sum(l_quantity) > 300 ) 
                        and c_custkey = o_custkey and o_orderkey = l_orderkey 
group by c_name, c_custkey, o_orderkey, o_orderdate, o_totalprice 
order by o_totalprice desc, o_orderdate limit 100; 


select /* Q22 */ cntrycode, count(*) as numcust, sum(c_acctbal) as totacctbal 
from (select substring(c_phone,1,2) as cntrycode, c_acctbal 
        from customer 
       where substring(c_phone,1,2) in ('13', '31', '23', '29', '30', '18', '17') 
         and c_acctbal > (select avg(c_acctbal) 
                            from customer 
                           where c_acctbal > 0.00 
                             and substring(c_phone,1,2) in ('13', '31', '23', '29', '30', '18', '17')) 
    and not exists ( select * from orders 
                             where o_custkey = c_custkey)) as custsale 
group by cntrycode order by cntrycode; 
```




	
