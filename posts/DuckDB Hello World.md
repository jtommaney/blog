**DuckDB is a work-in-progress embedded [in-memory]/parallel/vector/columnar analytics engine. **   

Background reading here:  https://duckdb.org/docs/why_duckdb

**Python Hello World - create 1/2 Billion rows in about 12 seconds, about 1/2 second query **
```
import duckdb
conn = duckdb.connect()
conn.execute("create table t as select mod(range, 100000) c1, 42 c2 from range(500000000);")
print(conn.execute("select c1, count(*) from t where c1 < 5 group by 1").fetchall())
```
**Add parallel execution with this syntax: ** 
```
conn.execute("pragma threads=8")
```

**Create an in-memory database via CLI:**
```
$./build/release/duckdb
DuckDB c0f167461
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
D .timer on
D create table t as select mod(range, 100000) c1, 42 c2 from range(500000000);
Run Time: real 14.281 user 11.257185 sys 3.037032
D select c1, count(*) from t where c1 < 5 group by 1;
┌────┬─────────┐
│ c1 │ count() │
├────┼─────────┤
│ 0  │ 5000    │
│ 1  │ 5000    │
│ 2  │ 5000    │
│ 3  │ 5000    │
│ 4  │ 5000    │
└────┴─────────┘
Run Time: real 0.452 user 0.452294 sys 0.000000
```

DuckDB 0.2.2  --version DuckDB c0f167461  Nov 12, 2020
