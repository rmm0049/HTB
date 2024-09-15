# SQLMap Overview

SQLMap is a free and open-source penetration testing tool written in Python that automates the process of detecting and exploiting SQL injection (SQLi) flaws.

ex)

`python sqlmap.py -u 'http://inlanefreight.htb/page.php?id=5'`

It comes with a powerful detection engine, numerous features, and a broad range of options and switches for fine-tuning the many aspects of it, such as:

- Target Connection
- Enumeration
- Injection detection
- Database content retrieval
- File system access
- fingerprinting
...

##### Installation

```
sudo apt install sqlmap
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev
python sqlmap.py
```

#### Supported SQL Injection Types

SQLMap is the only penetration testing tool that can properly detect and exploit all known SQLi types. We see the types of SQL injections supported by SQLMap with the `sqlmap -hh` command

The technique characters `BEUSTQ` refers to the following:
- `B`: Boolean-based blind
- `E`: Error-based
- `U`: Union query-based
- `S`: Stacked queries
- `T`: Time-based blind
- `Q`: Inline queries

##### Boolean-based blind

ex)

```sql
AND 1=1
```

This type is exploited through the differentiation of `TRUE` from `FALSE` query results, effectively retireving 1 byte of info per request. The differentiation is based on comparing server responses to determine whether the SQL query returned `TRUE` or `FALSE`.  This ranges from fuzzy comparisons of raw response content, HTTP codes, page titles, filtered text and other factors

- `TRUE` results are generally based on responses having none or marginal differences to the regular server reponse
- `FALSE` based on having substantial differences from regular server response
- `Boolean-based blind SQLi` is the most common type in web applications

#### Error-based SQL Injection

Ex)

```sql
AND GTID_SUBSET(@@version, 0)
```

If the Database Management System (DBMS) erros are being returned as part of the server response for any database-related problems, then there is a probability that they can be used to carry the results for requested queries.

#### UNION query-based

Example of UNION query-based SQl Injection:

```sql
UNION ALL SELECT 1,@@version,3
```

With the usage of `UNION`, it is generally possible to extend the original (`vulnerable`) query with the injected statements' results. This way, if the original query results are rendered as part of the response, the attacker can get additional results from the injected statements within the page response itself. This type of SQLi is considered the *fastest*, as in the ideal scenario, the attacker would be able to pull the content of the whole database table of interest with a single request

#### Stacked queries

Ex)

```sql
; DROP TABLE users
```

Stacking SQL queries, also known as the "piggy-backing", is the form of injecting additional SQL statements after the vulnerable one. In case that there is a requirement for running non-query statements (e.g.  `INSERT`, `UPDATE`, or `DELETE`), stacking must be supported by the vulnerable platform.


