# jdbc driver

* https://www.javatpoint.com/jdbc-driver

## types 
* https://en.wikipedia.org/wiki/Java_Database_Connectivity
- Type 1 that calls native code of the locally available ODBC driver. (Note: In JDBC 4.2, JDBC-ODBC bridge has been removed[9])
- Type 2 that calls database vendor native library on a client side. This code then talks to database over the network.
- Type 3, the pure-java driver that talks with the server-side middleware that then talks to the database.
- Type 4, the pure-java driver that uses database native protocol.
- Extra, internal JDBC driver - a driver embedded with JRE in Java-enabled SQL databases.

## 활용예
```java
Connection conn = DriverManager.getConnection(
     "jdbc:somejdbcvendor:other data needed by some jdbc vendor",
     "myLogin",
     "myPassword");
try {
     /* you use the connection here */
} finally {
    //It's important to close the connection when you are done with it
    try { 
        conn.close();
    } catch (Throwable e) { /* Propagate the original exception
                                instead of this one that you want just logged */ 
        logger.warn("Could not close JDBC Connection",e);
    }
}
```

## JDBC vendors
* https://www.oracle.com/technetwork/java/index-136695.html


## 레퍼런스
* https://github.com/schemacrawler/SchemaCrawler
    * jdbc 를 활용하는 형태

## Hive 의 경우 
* https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-JDBC
* Hive JDBC Connector 2.6.5 for Cloudera Enterprise
    * https://www.cloudera.com/downloads/connectors/hive/jdbc/2-6-5.html