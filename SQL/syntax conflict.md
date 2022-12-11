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

```EBNF
newline             = /* the Unicode code point U+000A */ .
unicode_char        = /* an arbitrary Unicode code point except newline */ .
```

# Letters and Digits

### influxdb
```EBNF
letter              = ascii_letter | "_" .
ascii_letter        = "A" … "Z" | "a" … "z" .
digit               = "0" … "9" .
```

### postgresql
```EBNF
letter              = ascii_letter | "_" .
ascii_letter        = "A" … "Z" | "a" … "z" .
digit               = "0" … "9" .
dollar              = "$"
```

### Conclusion

influxdb目前不支持`美元引用的字符串常量`和`位置参数`，`$`不作为字母。

> Conflict: ***NO***

# Identifiers

### influxdb
```EBNF
identifier          = unquoted_identifier | quoted_identifier .
unquoted_identifier = ( letter ) { letter | digit } .
quoted_identifier   = `"` unicode_char { unicode_char } `"` .
```

### postgresql
```EBNF
identifier          = unquoted_identifier | quoted_identifier .
unquoted_identifier = ( letter ) { letter | digit | dollar } .
quoted_identifier   = `"` unicode_char { unicode_char } `"` .
```

### Conclusion

influxdb标识符不包含`$`。postgresql标识符中可以包含`$`，但是不推荐使用`$`。

> Conflict: ***NO***

# Keywords

### influxdb
```SQL
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

influxdb关键字不是postgresql的真子集，如MEASUREMENT，MEASUREMENTS，REPLICATION等，未来postgresql扩展时，可能会冲突。

> Conflict: ***YES***

# Symbol

### influxdb

```
+ - * / < > = % ^ & | ( ) , ; * .
```

### postgresql

```
+ - * / < > = ~ ! @ # % ^ & | ` ? $ ( ) [ ] , ; : * .
```

### Conclusion

influxdb的符号集合是postgresql的真子集，无冲突。

> Conflict: ***NO**

# Literal

## String

### influxdb

字符串常量中包含`'`，转义符为`\'`。

```EBNF
string_lit          = `'` { unicode_char } `'` .
```

### postgresql

字符串常量中包含`'`，转义符为`''`和`\'`。

两个只由空白及至少一个新行分隔的字符串常量会被连接在一起，并且将作为一个写在一起的字符串常量来对待

```EBNF
string_lit          = string_line [newline string_line] .
string_line         = `'` { unicode_char } `'` .
```

### Conclusion

`\'`转义符和多行拼接字符串涉及词法解析冲突。

> Conflict: ***YES***

## Integer

### influxdb

```EBNF
int_lit             = ( "1" … "9" ) { digit } .
```

### postgresql

```EBNF
int_lit             = ( "1" … "9" ) { digit } .
```

### Conclusion

postgresql区分32位和64位，根据值域选择。influxdb只支持64位。

> Conflict: ***NO***

## Float

### influxdb

```EBNF
float_lit         = int_lit "." int_lit ["e" ["+"|"-"] int_lit] .
```

### postgresql

```EBNF
float_lit           = (int_lit | float_basic | float_decimal | float_integral) ["e" ["+"|"-"] int_lit] .
float_integral      = int_lit "." [int_lit]
float_decimal       = [int_lit] "." int_lit
float_basic         = int_lit "." int_lit
```
### Conclusion

postgresql中一个不包含小数点和指数的数字常量的值适合类型integer（32 位），它首先被假定为类型integer。否则如果它的值适合类型bigint（64 位），它被假定为类型bigint。再否则它会被取做类型numeric。包含小数点和/或指数的常量总是首先被假定为类型numeric。

> Conflict: ***YES***

## Duration

### influxdb

```EBNF
duration_lit        = int_lit duration_unit .
duration_unit       = "ns" | "u" | "µ" | "ms" | "s" | "m" | "h" | "d" | "w" .
```

| Units | Meaning |
| --- | --- |
| ns | nanoseconds (1 billionth of a second) |
| u or µ | microseconds (1 millionth of a second) |
| ms | milliseconds (1 thousandth of a second) |
| s | second |
| m | minute |
| h | hour |
| d | day |
| w | week |

### postgresql

[日期/时间类型](http://postgres.cn/docs/14/datatype-datetime.html)中`interval`类型有一个附加选项，它可以通过写下面之一的短语来限制存储的fields的集合：

```
YEAR
MONTH
DAY
HOUR
MINUTE
SECOND
YEAR TO MONTH
DAY TO HOUR
DAY TO MINUTE
DAY TO SECOND
HOUR TO MINUTE
HOUR TO SECOND
MINUTE TO SECOND
```

### Conclusion

> Conflict: ***YES***

## Date & Time

### influxdb

```EBNF
time_lit            = "YYYY-MM-DD hh:mm:ss.p" | "YYYY-MM-DD" .
```

### postgresql

```EBNF
time_lit            = "YYYY-MM-DD hh:mm:ss.p" | "YYYY-MM-DD" .
```

### Conclusion

postgresql的日期和时间的输入可以接受几乎任何合理的格式，包括 ISO 8601、SQL-兼容的和其他的形式。influxdb的日期时间格式只支持ISO 8601。

> Conflict: ***NO***

## Boolean

### influxdb

```EBNF
bool_lit            = TRUE | FALSE .
```

### postgresql

```EBNF
bool_lit            = TRUE | FALSE .
```

### Conclusion

postgresql的类型转换支持将一下字符串表示位”真“或”假“状态：

| TRUE | FALSE |
| --- | --- |
| true | false |
| yes | no |
| on | off |
| 1 | 0 |

> Conflict: ***NO***

## Regular Expression

### influxdb

```EBNF
regex_lit           = "/" { unicode_char } "/" .
```

### postgresql

```EBNF
regex_lit          = regex_line [newline regex_line] .
regex_line         = `'` { unicode_char } `'` .
```

### Conclusion

postgresql在[模式匹配](http://postgres.cn/docs/14/functions-matching.html)定义正则表达式为字符串。

> Conflict: ***YES***

# Queries

# Statements

# Clauses

# Exceptions

# Other



