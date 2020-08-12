---
title: 'Rundeck 3.3.1 - MySQL connection'
description: "Rundeck"
date: "2020-08-12"
---

Rundeck 3.3.1 updated its 'mysql-connector-java' dependancy to version 8.0.21 which renamed the &nbsp; `com.mysql.jdbc.Driver` &nbsp; classname to &nbsp; `com.mysql.cj.jdbc.Driver`.  
  
Reconfigure Rundeck accordingly:

```shell
/etc/rundeck/rundeck-config.properties:
...
- dataSource.url = jdbc:mysql://localhost/rundeck?autoReconnect=true&useSSL=false
+ dataSource.url = jdbc:mysql://localhost/rundeck?serverTimezone=Europe/Amsterdam
...
- dataSource.driverClassName = com.mysql.jdbc.Driver
+ dataSource.driverClassName = com.mysql.cj.jdbc.Driver
+ dataSource.properties.autoReconnect = true
+ dataSource.properties.useSSL = false
...
```
