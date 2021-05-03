+++
author = "Harvey"
title = "SQL Server"
date = "2017-08-10"
description = "Connecting to a SQL Server database"
tags = ["R"]
+++

Connecting R to a SQL Server can be achieved using the RODBC or RJDBC libraries.  Under CentOS this is a little more challenging than Windows and is generally achieved by installing the freeTDS ODBC driver and using the RODBC library.  I found compilation a little difficult under CentOS 5.7 and so I moved on to JDBC instead.

After installing the JDBC from [https://www.microsoft.com/en-us/download/details.aspx?id=21599](https://www.microsoft.com/en-us/download/details.aspx?id=21599) and unpacking to /etc, it was simple to connect to a database as follows:

```r
drv <- JDBC("com.microsoft.sqlserver.jdbc.SQLServerDriver", "/etc/sqljdbc_3.0/enu/sqljdbc4.jar")
conn <- dbConnect(drv, 'jdbc:sqlserver://server_address', 'user', 'password')
dbListTables(conn)
```