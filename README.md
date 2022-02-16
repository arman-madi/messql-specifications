# MessQL Specifications
The purpose of this repository is to define the specification of a query language called MessQL.

Every log-source contains raw-logs come in multiple arbitrary formats
```
\> cat a.log
s1	Arman	2  
s2	John	3
\> cat b.log
s2	A	B	12.3
s1	C	B	10
```
All log-sources loaded in messq forms a large set of logs 
```
messq --log-source a_log=a.log --log-source b_log=b.log
--> PRINT
-------
s1	Arman	2  
-------
s2	John	3
-------
s2	A	B	12.3
-------
s1	C	B	10
-------
```
MessQL consists of one or more blocks of instructions 
```
messq --log-source a_log=a.log --log-source b_log=b.log
--> // Each line is a block of instructions separated by pipeline (|) 
--> WHERE _log_source = a_log | PRINT
-------
s1	Arman	2  
-------
s2	John	3
-------
--> PRINT
-------
s1	Arman	2  
-------
s2	John	3
-------
s2	A	B	12.3
-------
s1	C	B	10
-------
--> // Blocks can be explecitly shown whithin brackets {} in which every instruction will run over the previous result 
--> // regardless of having them in a single line or multiple lines
--> {
--> // in the following line cut will split the provided field based on first param	and forms new value based on the second param. If the field is not provided _raw_log is the default field
--> SELECT cut(/^\w+\s/, "f2") AS f1 | WHERE f1 = "A"
--> PRINT
-------
s2	A	B	12.3
f1=A
-------
--> }
--> // Here f1 does not exist any more
--> WHERE contains("A") | PRINT
-------
s2	A	B	12.3
-------
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
-------
s1	Arman	2  
-------
s2	John	3
-------
```

SELECT helps to create and add more fields
 ```
 messq --log-source a_log=a.log
 --> SELECT split(_raw_log) AS strcat("c", _i) | PRINT
s1	Arman	2  
c1=s1, c2=Arman, c3=2
-------
s2	John	3
c1=s2, c2=John, c3=3
 ```

 WHERE filters logs based on conditions
 ```
 messq --log-source b=b.log
 --> WHERE len(_raw_log) > 10
 -------
 s2	A	B	12.3
 -------
 ```

 GROUP BY will groups logs based on given fields and run aggregation functions on each group
 ```
messq --log-source --log-source b_log=b.log
--> {
--> SELECT split(_raw_log) AS strcat("c", _i)
--> GROUP BY c3 AGGREGATE strcat(c1, ",") AS stds 
--> PRINT
| stds |
----------
 stds=s2,s1
----------
--> }
```


