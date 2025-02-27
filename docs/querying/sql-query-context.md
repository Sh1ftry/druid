---
id: sql-query-context
title: "SQL query context"
sidebar_label: "SQL query context"
---

<!--
  ~ Licensed to the Apache Software Foundation (ASF) under one
  ~ or more contributor license agreements.  See the NOTICE file
  ~ distributed with this work for additional information
  ~ regarding copyright ownership.  The ASF licenses this file
  ~ to you under the Apache License, Version 2.0 (the
  ~ "License"); you may not use this file except in compliance
  ~ with the License.  You may obtain a copy of the License at
  ~
  ~   http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing,
  ~ software distributed under the License is distributed on an
  ~ "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
  ~ KIND, either express or implied.  See the License for the
  ~ specific language governing permissions and limitations
  ~ under the License.
  -->

> Apache Druid supports two query languages: Druid SQL and [native queries](querying.md).
> This document describes the SQL language.

Druid supports query context parameters which affect SQL planning.
See [Query context](query-context.md) for general query context parameters for all query types.

## SQL query context parameters

Configure Druid SQL query planning using the parameters in the table below.

|Parameter|Description|Default value|
|---------|-----------|-------------|
|`sqlQueryId`|Unique identifier given to this SQL query. For HTTP client, it will be returned in `X-Druid-SQL-Query-Id` header.<br/><br/>To specify a unique identifier for SQL query, use `sqlQueryId` instead of [`queryId`](query-context.md). Setting `queryId` for a SQL request has no effect. All native queries underlying SQL use an auto-generated `queryId`.|auto-generated|
|`sqlTimeZone`|Sets the time zone for this connection, which will affect how time functions and timestamp literals behave. Should be a time zone name like "America/Los_Angeles" or offset like "-08:00".|druid.sql.planner.sqlTimeZone on the Broker (default: UTC)|
|`sqlStringifyArrays`|When set to true, result columns which return array values will be serialized into a JSON string in the response instead of as an array (default: true, except for JDBC connections, where it is always false)|
|`useApproximateCountDistinct`|Whether to use an approximate cardinality algorithm for `COUNT(DISTINCT foo)`.|druid.sql.planner.useApproximateCountDistinct on the Broker (default: true)|
|`useGroupingSetForExactDistinct`|Whether to use grouping sets to execute queries with multiple exact distinct aggregations.|druid.sql.planner.useGroupingSetForExactDistinct on the Broker (default: false)|
|`useApproximateTopN`|Whether to use approximate [TopN queries](topnquery.md) when a SQL query could be expressed as such. If false, exact [GroupBy queries](groupbyquery.md) will be used instead.|druid.sql.planner.useApproximateTopN on the Broker (default: true)|
|`enableTimeBoundaryPlanning`|If true, SQL queries will get converted to TimeBoundary queries wherever possible. TimeBoundary queries are very efficient for min-max calculation on __time column in a datasource |druid.query.default.context.enableTimeBoundaryPlanning on the Broker (default: false)|

## Setting the query context
The query context parameters can be specified as a "context" object in the [JSON API](sql-api.md) or as a [JDBC connection properties object](sql-jdbc.md).
See examples for each option below.

### Example using JSON API

```
{
  "query" : "SELECT COUNT(*) FROM data_source WHERE foo = 'bar' AND __time > TIMESTAMP '2000-01-01 00:00:00'",
  "context" : {
    "sqlTimeZone" : "America/Los_Angeles"
  }
}
```

### Example using JDBC

```java
String url = "jdbc:avatica:remote:url=http://localhost:8082/druid/v2/sql/avatica/";

// Set any query context parameters you need here.
Properties connectionProperties = new Properties();
connectionProperties.setProperty("sqlTimeZone", "America/Los_Angeles");
connectionProperties.setProperty("useCache", "false");

try (Connection connection = DriverManager.getConnection(url, connectionProperties)) {
  // create and execute statements, process result sets, etc
}
```

