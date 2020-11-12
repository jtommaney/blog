## Profiling DuckDB:  Scale Factor to 300GB, Threads to 16

General test scenario:  create TPC-H schemas at scale factor (SF) 10, 30, 100, and 300.  
SF-100 is 150 million orders, with 600 million line items and about 100GB raw data.  

Use pragma threads = 8; syntax to change then number of threads working on a given query.  

* Threads in [ 1,2,4,8,16 ]
* Concurrent streams in [ 1,2,4 ]
* TPC-H Queries range [ 1...22]
* TPC-H Scale Factor in [ 10,30,100,300 ]
