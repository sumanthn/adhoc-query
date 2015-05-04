Adhoc query on structured data
===================
Fire SQL queries on accesslog data; check out factdatagenerator for more on generating access log
storage - ORC compressed with snappy
Query Engine - Presto(https://prestodb.io/)
Unlike the other data processing & query processing engines, this is a different challenge since the data cannot be modelled as required for the queries , since queries are unknown.

----------


Data-storage
-------------

    create  table factdata(
    accessUrl string,
    responseStatusCode int,
    responseTime int,
    accessTimestamp string,
    requestVerb string,
    requestSize int,
    dataExchangeSize int,
    serverIp string,
    clientIp string,
    clientId string,
    sessionId string,
    userAgentDevice string,
    UserAgentType string,
    userAgentFamily string,
    userAgentOSFamily string,
    userAgentVersion string,
    userAgentOSVersion string,
    city string,
    country string,
    region string,
    minOfDay int,
    hourOfDay int,
    dayOfWeek int,
    monthOfYear int
   

    ) 
    PARTITIONED BY (day string)
    stored as orc tblproperties ("orc.compress"="SNAPPY", "orc.stripe.size"="67108864","orc.row.index.stride"="50000") ;

Generate the raw data using factgenerator and place in a temporary storage say /stage

**Load data into Hive**

    load data inpath '/stage/fdata_20150401_20150402_5000.orc' into table factdata  PARTITION (day='20150401');

....

**Querying**
Start hive meta server
Setup presto with hive catalog
Query from presto-cli or use *presto-jdbc*
Presto supports ANSI -SQL

Results
On a 4 node server [4 cores, 16G & 4T each]backed by hdfs & query engine as presto
5000 tps files for 1 month data ~12 Billion records

|   Results  |             |          |          |        |          |                       |
|:----------:|-------------|----------|----------|--------|----------|-----------------------|
| Query Type | Projections | Group By | Order By | Filter | Timespan | Response time minutes |
|            | 1           |        1 |        1 | 0      |   3 days |                     1 |
|            | 4           |        2 |        2 | 2      |   1 week |                     3 |
|            | 5           |        5 |        5 | 0      |   1 week |                   5.4 |
|            | 3           | 0        | 3        | 3      | 2 weeks  | 2                     |

**Learnings**

 - Read as less as you can - ORC is your friend in reading only columns required
 - Read sequentially , no index but do skip over the files
in ORC using min & max for each stripe or skip over partitions
 - Read in parallel
 - Try to do as much as possible in-memory


 
