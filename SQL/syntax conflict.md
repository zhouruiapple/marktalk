# Abstract

# Notation

Extended Backus-Naur Form (“EBNF”). 

```EBNF
Production  = production_name "=" [ Expression ] "." .
Expression  = Alternative { "|" Alternative } .
Alternative = Term { Term } .
Term        = production_name | token [ "…" token ] | Group | Option | Repetition .
Group       = "(" Expression ")" .
Option      = "[" Expression "]" .
Repetition  = "{" Expression "}" .
```

Notation operator.

```EBNF
|   alternation
()  grouping
[]  option (0 or 1 times)
{}  repetition (0 to n times)
```

# Query Representation

Unicode text encoded in **UTF-8**

# Letters and Digits

### influxdb
```
letter              = ascii_letter | "_" .
ascii_letter        = "A" … "Z" | "a" … "z" .
digit               = "0" … "9" .
```

### postgresql
```
letter              = ascii_letter | "_" .
ascii_letter        = "A" … "Z" | "a" … "z" .
digit               = "0" … "9" .
dollar              = "$"
```

### Conclusion

influxdb不支持“**美元引用的字符串常量**”和“**位置参数**”，$不作为字母。

> Conflict: **NO**


# Identifiers

### influxdb
```
identifier          = unquoted_identifier | quoted_identifier .
unquoted_identifier = ( letter ) { letter | digit } .
quoted_identifier   = `"` unicode_char { unicode_char } `"` .
```

### postgresql
```
identifier          = unquoted_identifier | quoted_identifier .
unquoted_identifier = ( letter ) { letter | digit | dollar } .
quoted_identifier   = `"` unicode_char { unicode_char } `"` .
```

### Conclusion

influxdb标识符不包含$。

> Conflict: **NO**

# Keywords

### influxdb
```
ALL           ALTER         ANY           AS            ASC           BEGIN
BY            CREATE        CONTINUOUS    DATABASE      DATABASES     DEFAULT
DELETE        DESC          DESTINATIONS  DIAGNOSTICS   DISTINCT      DROP
DURATION      END           EVERY         EXPLAIN       FIELD         FOR
FROM          GRANT         GRANTS        GROUP         GROUPS        IN
INF           INSERT        INTO          KEY           KEYS          KILL
LIMIT         SHOW          MEASUREMENT   MEASUREMENTS  NAME          OFFSET
ON            ORDER         PASSWORD      POLICY        POLICIES      PRIVILEGES
QUERIES       QUERY         READ          REPLICATION   RESAMPLE      RETENTION
REVOKE        SELECT        SERIES        SET           SHARD         SHARDS
SLIMIT        SOFFSET       STATS         SUBSCRIPTION  SUBSCRIPTIONS TAG
TO            USER          USERS         VALUES        WHERE         WITH
WRITE
```

### postgresql

[附录 C. SQL关键词](http://postgres.cn/docs/14/sql-keywords-appendix.html)

### Conclusion

influxdb关键字不是postgresql的真子集，如MEASUREMENT，MEASUREMENTS，REPLICATION等。

> Conflict: **YES**

# Literals

# Queries

# Statements

# Clauses

# Exceptions

# Other



