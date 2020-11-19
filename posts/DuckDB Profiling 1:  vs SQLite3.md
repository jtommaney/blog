## DuckDB Profiling Summary
* We are hiring if you are interested in working on DuckDB with Alibaba!  
  Looking for C++ w/ DBMS tech, Seattle/Sunnyvale areas (after work-from-home restrictions).
* DuckDB has excellent usability and performance across many query, data, and parallelism conditions.
* Up to 80x faster than SQLite3. DBT3 benchmark at 10 GB:  SQLite3 - 2573.88 Seconds, DuckDB with 8 threads - 32.15 seconds

## DuckDB Profiling 1: General Setup
General test scenario:  create DBT-3 schemas at scale factor (SF) 10.  
SF-10 is 15 million orders, with 60 million line items and about 10GB raw data.  

```
DuckDB used double instead of integer for columns used in aggregation - current software pessimistically handles integer math with HugeInt internally.  

Create SQLite schema with indexes on these columns:
n_nationkey, n_regionkey 
r_regionkey 
c_custkey, c_nationkey
p_partkey
s_suppkey,s_nationkey
ps_partkey, ps_suppkey
o_orderkey, o_custkey, o_orderdate
l_orderkey,l_partkey,
l_suppkey,l_shipdate,l_commitdate,l_receiptdate
[ l_partkey,  l_suppkey] and [ l_suppkey,  l_partkey]
[ps_partkey, ps_suppkey] and [ps_suppkey, ps_partkey]
```
## DuckDB Profiling 1: Comparison with SQLite3
```
|  Loading  10GB  |  SQLite3 2417 Seconds |  
                  |  DuckDB   165 Seconds |

|  Memory Footprint  |  SQLite3 28.0g Resident Memory  |
                     |  DuckDB  11.3g Resident Memory  |

|  Total Query Seconds  |  SQLite3          2573.8 Seconds  |
                        |  DuckDB 1 thread   103.4 Seconds  |
                        |  DuckDB 8 threads   32.2 Seconds  |
100GB (instead of 10GB) |  DuckDB 8 threads  421.7 Seconds  |
```
Per-Query speedup ( x times faster ):
![](https://github.com/jtommaney/blog/blob/blog/assets/DuckDB_Speedup.png?raw=true) 


Individual timings

![](https://github.com/jtommaney/blog/blob/blog/assets/DuckDB_SQLite.png?raw=true) 
