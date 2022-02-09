# MessQL Specifications
The purpose of this repository is to define the specification of a query language called MessQL.

Every log-source contains raw-logs come in multiple arbitrary formtats
```
\> cat a.log
a1	a2	a3
1	2	3	4
\> cat b.log
b1	2
x1	2	c3	4
```
Adding a log-source means all of its logs will be considered as a part of whole logs
```
messq --log-source a_log=a.log --log-source b_log=b.log
--> PRINT
a1	a2	a3
1	2	3	4
b1	2
x1	2	c3	4
```
MessQL consist of one or more blocks of instructions to dig into the log-sources and extract useful information.
```
messq --log-source a_log=a.log --log-source b_log=b.log
--> // Each line is a block of instructions separated by pipeline (|) 
--> WHERE _log_source = a_log | PRINT
a1	a2	a3
1	2	3	4
--> PRINT
a1	a2	a3
1	2	3	4
b1	2
x1	2	c3	4
--> // Blocks can be explecitly shown whithin brackets {} in which every instruction will run over the previous result 
--> // regardless of having them in a single line or multiple lines
--> {
--> SELECT regex(/^\w+\s/, _raw_log) AS f1 | WHERE f1 = "x1"
--> PRINT
x1	2	c3	4 
f1=x1
--> }
--> // Here f1 does not exist any more
--> PRINT
a1	a2	a3
1	2	3	4
b1	2
x1	2	c3	4
--> EXIT
```

# Nouns
log-source: 
query

# Instructions

LOGSRC is used to add a log-source to the running block

```
messq 
--> LOGSRC a_log = a.log | PRINT
a1	a2	a3
1	2	3	4
```

SELECT helps to create and add more fields
 ```
 messq --log-source a_log=a.log
 --> SELECT split(_raw_log) AS strcat("c", _i) | PRINT
a1	a2	a3
c1=a1, c2=a2, c3=a3
-------
1	2	3	4
c1=1, c2=2, c3=3, c4=4
 ```

 WHERE filters logs based on conditions
 ```
 messq --log-source b=b.log
 --> WHERE _raw_log.contains("b1")
 b1	2
 ```

 GROUP BY will groups logs based on given fields and run aggregation functions on each group
 ```
messq --log-source a_log=a.log --log-source b_log=b.log
--> SELECT split(_raw_log) AS strcat("c", _i)
--> GROUP BY c4 AGGREGATE strcat(c1) AS agg
| agg |
----------
 a1b1
 1x1
```


