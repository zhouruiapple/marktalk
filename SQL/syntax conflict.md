# Abstract

以postgresql作为`标准SQL`，分析influxdb的`InfluxQL`查询语言语法上与`标准SQL`的差异和冲突。

语法冲突分析遵循`3`个关键规则：

- `标准SQL`中存在，`InfluxQL`中不存在的语法，不会冲突；
- `InfluxQL`中存在，`标准SQL`中不存在的语法，当`标准SQL`扩展时会冲突；
- 在`标准SQL`和`InfluxQL`同时存在的语法，当语义不同时会冲突；

# Notation

使用Extended Backus-Naur Form (“EBNF”)来描述语法. 

```EBNF
Production  = production_name "=" [ expr ] "." .
expr  = Alternative { "|" Alternative } .
Alternative = Term { Term } .
Term        = production_name | token [ "…" token ] | Group | Option | Repetition .
Group       = "(" expr ")" .
Option      = "[" expr "]" .
Repetition  = "{" expr "}" .
```

EBNF操作符，

```EBNF
|   alternation
()  grouping
[]  option (0 or 1 times)
{}  repetition (0 to n times)
```

# Query Representation

使用`UTF-8`对查询文本进行编码，文本由一下两个部分组成，

```EBNF
newline             = /* the Unicode code point U+000A */ .
unicode_char        = /* an arbitrary Unicode code point except newline */ .
```

# Letters and Digits

`influxdb`

```EBNF
letter              = ascii_letter | "_" .
ascii_letter        = "A" … "Z" | "a" … "z" .
digit               = "0" … "9" .
```

`postgresql`

```EBNF
letter              = ascii_letter | "_" .
ascii_letter        = "A" … "Z" | "a" … "z" .
digit               = "0" … "9" .
dollar              = "$"
```

`Conclusion`

- influxdb目前不支持`美元引用的字符串常量`和`位置参数`，`$`不作为字母。

> Conflict: ***NO***

# Identifiers

`influxdb`

```EBNF
identifier          = unquoted_identifier | quoted_identifier .
unquoted_identifier = ( letter ) { letter | digit } .
quoted_identifier   = `"` unicode_char { unicode_char } `"` .
```

`postgresql`

```EBNF
identifier          = unquoted_identifier | quoted_identifier .
unquoted_identifier = ( letter ) { letter | digit | dollar } .
quoted_identifier   = `"` unicode_char { unicode_char } `"` .
```

`Conclusion`

- influxdb标识符不包含`$`。postgresql标识符中可以包含`$`，但是不推荐使用`$`。

> Conflict: ***NO***

# Keywords

`influxdb`

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

`postgresql`

[附录 C. SQL关键词](http://postgres.cn/docs/14/sql-keywords-appendix.html)

`Conclusion`

- influxdb关键字不是postgresql的真子集，如MEASUREMENT，MEASUREMENTS，REPLICATION等，未来postgresql扩展时，可能会冲突。

> Conflict: ***YES***

# Symbol

`influxdb`

```
+ - * / < > = % ^ & | ( ) , ; * .
```

`postgresql`

```
+ - * / < > = ~ ! @ # % ^ & | ` ? $ ( ) [ ] , ; : * .
```

`Conclusion`

- influxdb的符号集合是postgresql的真子集，无冲突。

> Conflict: ***NO***

# Expression

`influxdb`

```EBNF
binary_op        = "+" | "-" | "*" | "/" | "%" | "&" | "|" | "^" | "AND" |
                   "OR" | "=" | "!=" | "<>" | "<" | "<=" | ">" | ">=" .
expr             = unary_expr { binary_op unary_expr } .
unary_expr       = "(" expr ")" | var_ref | time_lit | string_lit | int_lit | float_lit | bool_lit | duration_lit | regex_lit .
```

`postgresql`

***TODO***

`Conclusion`

- 未找到postgresql官方定义，经验上语法应该和influxdb一样，主要的区别在于`binary_op`的数量大于influxdb。

> Conflict: ***NO***

# Literal

## String

`influxdb`

字符串常量中包含`'`，转义符为`\'`。

```EBNF
string_lit          = `'` { unicode_char } `'` .
```

`postgresql`

字符串常量中包含`'`，转义符为`''`和`\'`。

两个只由空白及至少一个新行分隔的字符串常量会被连接在一起，并且将作为一个写在一起的字符串常量来对待

```EBNF
string_lit          = string_line [newline string_line] .
string_line         = `'` { unicode_char } `'` .
```

`Conclusion`

- `\'`转义符和多行拼接字符串词法冲突。

> Conflict: ***YES***

## Integer

`influxdb`

```EBNF
int_lit             = ( "1" … "9" ) { digit } .
```

`postgresql`

```EBNF
int_lit             = ( "1" … "9" ) { digit } .
```

`Conclusion`

> Conflict: ***NO***

## Float

`influxdb`

```EBNF
float_lit         = int_lit "." int_lit ["e" ["+"|"-"] int_lit] .
```

`postgresql`

```EBNF
float_lit           = (int_lit | float_basic | float_decimal | float_integral) ["e" ["+"|"-"] int_lit] .
float_integral      = int_lit "." [int_lit]
float_decimal       = [int_lit] "." int_lit
float_basic         = int_lit "." int_lit
```

`Conclusion`

- postgresql中一个不包含小数点和指数的数字常量的值适合类型integer（32 位），它首先被假定为类型integer。否则如果它的值适合类型bigint（64 位），它被假定为类型bigint。再否则它会被取做类型numeric。包含小数点和/或指数的常量总是首先被假定为类型numeric。

> Conflict: ***YES***

## Duration

`influxdb`

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

`postgresql`

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

`Conclusion`

> Conflict: ***YES***

## Date & Time

`influxdb`

```EBNF
time_lit            = "YYYY-MM-DD hh:mm:ss.p" | "YYYY-MM-DD" .
```

`postgresql`

```EBNF
time_lit            = "YYYY-MM-DD hh:mm:ss.p" | "YYYY-MM-DD" .
```

`Conclusion`

- postgresql的日期和时间的输入可以接受几乎任何合理的格式，包括 ISO 8601、SQL-兼容的和其他的形式。influxdb的日期时间格式只支持RFC 3339。

> Conflict: ***NO***

## Boolean

`influxdb`

```EBNF
bool_lit            = TRUE | FALSE .
```

`postgresql`

```EBNF
bool_lit            = TRUE | FALSE .
```

`Conclusion`

- postgresql的类型转换支持将一下字符串表示位”真“或”假“状态：

| TRUE | FALSE |
| --- | --- |
| true | false |
| yes | no |
| on | off |
| 1 | 0 |

> Conflict: ***NO***

## Regular Expression

`influxdb`

```EBNF
regex_lit           = "/" { unicode_char } "/" .
```

`postgresql`

```EBNF
regex_lit          = regex_line [newline regex_line] .
regex_line         = `'` { unicode_char } `'` .
```

`Conclusion`

- postgresql在[模式匹配](http://postgres.cn/docs/14/functions-matching.html)定义正则表达式为字符串。

> Conflict: ***YES***

# Queries

`influxdb`

```EBNF
query               = statement { ";" statement } .
statement           = alter_retention_policy_stmt |
                      create_continuous_query_stmt |
                      create_database_stmt |
                      create_retention_policy_stmt |
                      create_subscription_stmt |
                      create_user_stmt |
                      delete_stmt |
                      drop_continuous_query_stmt |
                      drop_database_stmt |
                      drop_measurement_stmt |
                      drop_retention_policy_stmt |
                      drop_series_stmt |
                      drop_shard_stmt |
                      drop_subscription_stmt |
                      drop_user_stmt |
                      explain_stmt |
                      explain_analyze_stmt |
                      grant_stmt |
                      kill_query_statement |
                      revoke_stmt |
                      select_stmt |
                      show_continuous_queries_stmt |
                      show_databases_stmt |
                      show_diagnostics_stmt |
                      show_field_key_cardinality_stmt |
                      show_field_keys_stmt |
                      show_grants_stmt |
                      show_measurement_cardinality_stmt |
                      show_measurement_exact_cardinality_stmt |
                      show_measurements_stmt |
                      show_queries_stmt |
                      show_retention_policies_stmt |
                      show_series_cardinality_stmt |
                      show_series_exact_cardinality_stmt |
                      show_series_stmt |
                      show_shard_groups_stmt |
                      show_shards_stmt |
                      show_stats_stmt |
                      show_subscriptions_stmt |
                      show_tag_key_cardinality_stmt |
                      show_tag_key_exact_cardinality_stmt |
                      show_tag_keys_stmt |
                      show_tag_values_stmt |
                      show_tag_values_cardinality_stmt |
                      show_users_stmt .
```

`postgresql`

参考支持的[SQL 命令](http://postgres.cn/docs/14/sql-commands.html)
```EBNF
query               = statement { ";" statement } .
statement           = /* SQL 命令 */ .
```

`Conclusion`

查询语句对应的`Statement`或`Clause`冲突时其必然冲突。

> Conclusion: ***YES***

# Statements

## SELECT

`influxdb`

```EBNF
select_stmt = 
SELECT select_list_clause
    [ into_clause ]
    from_clause
    [ into_clause ]
    [ where_clause ]
    [ group_by_clause ] 
    [ order_by_clause ]
    [ limit_clause ]
    [ offset_clause ]
    [ slimit_clause ]
    [ soffset_clause ]
    [ timezone_clause ] .
```

`postgresql`

```EBNF
select_stmt = [ WITH [ RECURSIVE ] with_query [, ...] ]
SELECT select_list_clause
    [ into_clause ]
    [ from_clause ]
    [ where_clause ]
    [ group_by_clause ]
    [ HAVING condition ]
    [ WINDOW window_name AS ( window_definition ) [, ...] ]
    [ { UNION | INTERSECT | EXCEPT } [ ALL | DISTINCT ] select ]
    [ order_by_clause ]
    [ limit_clause ]
    [ offset_clause ]
    [ FETCH { FIRST | NEXT } [ count ] { ROW | ROWS } { ONLY | WITH TIES } ]
    [ FOR { UPDATE | NO KEY UPDATE | SHARE | KEY SHARE } [ OF table_name [, ...] ] [ NOWAIT | SKIP LOCKED ] [...] ]
```

`Conclusion`

- influxdb的子句集合不是postgresql的子句集合的子集，当postgresql扩展时可能会存在冲突。

> Conflict: ***YES***

## INSERT

`influxdb`

```EBNF
insert_stmt     = <measurement>[,<tag_key>=<tag_value>[,<tag_key>=<tag_value>]] <field_key>=<field_value>[,<field_key>=<field_value>] [<timestamp>]
measurement     = { unicode_char }
tag_set         = { unicode_char }
field_set       = { unicode_char }
timestamp       = { unicode_char }
```

`postgresql`

```EBNF
insert_stmt     = [ WITH [ RECURSIVE ] with_query [, ...] ]
INSERT INTO table_name [ AS alias ] [ ( column_name [, ...] ) ]
    [ OVERRIDING { SYSTEM | USER } VALUE ]
    { DEFAULT VALUES | VALUES ( { expr | DEFAULT } [, ...] ) [, ...] | query }
    [ ON CONFLICT [ conflict_target ] conflict_action ]
    [ RETURNING * | output_expr [ [ AS ] output_name ] [, ...] ] .
conflict_target = ( { index_column_name | ( index_expr ) } [ COLLATE collation ] [ opclass ] [, ...] ) [ WHERE index_predicate ]
    ON CONSTRAINT constraint_name
conflict_action = DO NOTHING
    DO UPDATE SET { column_name = { expr | DEFAULT } | ( column_name [, ...] ) = [ ROW ] ( { expr | DEFAULT } [, ...] ) | ( column_name [, ...] ) = ( sub-SELECT ) } [, ...] [ WHERE condition ]
```

`Conclusion`

- influxdb中`INSERT`是自定义`Line Protocol`语法，`Lex`将该语法的解析成4个部分，Measurement，Tag set，Field set，Timestamp。随后对这四个部分分别以`key-value`的形式解析，使用的是一套独立的词法和语法解析。

> Conflict: ***NO***

## EXPLAIN

`influxdb`

```EBNF
explain_stmt    = EXPLAIN [option] select_stmt .
option          = ANALYZE
```

`postgresql`

```EBNF
explain_stmt    = EXPLAIN [ options ] select_stmt .
options         = option {, option}
option          = ANALYZE [bool_lit] | VERBOSE [bool_lit] | COSTS [bool_lit] | SETTINGS [bool_lit] | BUFFERS [bool_lit] | WAL [bool_lit] | TIMING [bool_lit] | SUMMARY [bool_lit] | FORMAT [bool_lit]
```

`Conclusion`

> Conflict: ***NO***

# Clauses

## SELECT_LIST_CLAUSE

`influxdb`

```EBNF
select_list_clause     = [ "*" | expr_as_list ] .
expr_as_list           = expr [ AS identify ] {, expr [ AS identify ]} .
```

`postgresql`

```EBNF
select_list_clause     = [ ALL | DISTINCT [ ON ( expr_list ) ] ] [ "*" | expr_as_list ] .
expr_list              = expr {, expr} .
expr_as_list           = expr [ [ AS ] identify ] {, expr [ [ AS ] identify ]} .
```

`Conclusion`

> conflict: ***NO***

## INTO_CLAUSE

`influxdb`

```EBNF
into_clause     = INTO ( table_name | back_ref ).
back_ref        = ( identify ".:MEASUREMENT" ) |
                   ( identify "." [ identify ] ".:MEASUREMENT" ) .
table_name      = identify {"." identify} .
```

`postgresql`

```EBNF
into_clause     = INTO [ TEMPORARY | TEMP | UNLOGGED ] [ TABLE ] table_name .
table_name      = identify {"." identify} .
```

`Conclusion`

> Conflict: ***NO***

## FROM_CLAUSE

`influxdb`

```EBNF
from_clause     = FROM from_items .
from_items      = from_item {, from_item} .
from_item       = item_table | item_subquery | item_cte | item_join .
item_join       = from_item join_type from_item [ ON expr ] .
item_table      = table_name [ AS identify ] .
item_subquery   = "(" select_stmt ")" [ AS identify ] .
join_type       = FULL [ OUTER ] JOIN .
table_name      = identify {"." identify} .
```

`postgresql`

```EBNF
from_clause     = FROM from_items .
from_items      = from_item {, from_item} .
from_item       = item_table | item_subquery | item_cte | item_join .
item_join       = from_item [ NATURAL ] join_type from_item [ ( ON expr ) | ( USING "(" identify {"," identify} ")" ) ] .
item_table      = [ ONLY ] table_name [ "*" ] [ [ AS ] identify [ "(" identify {"," identify} ")" ] ]
    [ TABLESAMPLE identify "(" expr {"," expr} ")" [ REPEATABLE "(" expr ")" ] ] .
item_subquery   = [ LATERAL ] "(" select_stmt ")" [ AS ] identify [ "(" identify {"," identify} ")" ] ] .
item_cte        = identify [ [ AS ] identify [ "(" identify {"," identify} ")" ] ] .
join_type       = ( [ INNER ] JOIN ) | ( LEFT [ OUTER ] JOIN ) | ( RIGHT [ OUTER ] JOIN ) | ( FULL [ OUTER ] JOIN ) | ( CROSS JOIN ) .
table_name      = identify {"." identify} .
```

`Conclusion`

- postgresql支持函数调用出现在FROM子句中，由于influxdb不支持且复杂，目前不分析。
- from_items在postgresql中将多个源表进行笛卡尔积运算，在influxdb中降多个源表进行集合并运算，语法相同但是语义冲突。

> Conflict: ***YES***

## WHERE_CLAUSE

`influxdb`

```EBNF
where_clause    = WHERE expr .
```

`postgresql`

```EBNF
where_clause    = WHERE expr .
```

`Conclusion`

> Conflict: ***NO***

## GROUP_BY_CLAUSE

`influxdb`

```EBNF
group_by_clause     = GROUP BY grouping_element [ fill "(" fill_option ")" ] .
fill_option         = "null" | "none" | "previous" | int_lit | float_lit | "linear" .
grouping_element    = identify {, identify}
```

`postgresql`

```EBNF
group_by_clause     = GROUP BY grouping_element
grouping_element    = identify {, identify}
```

`Conclusion`

> Conflict: ***NO***

## ORDER_BY_CLAUSE

`influxdb`

```EBNF
order_by_clause = ORDER BY sort_fields .
sort_fields     = sort_field { "," sort_field } .
sort_field      = identify [ ASC | DESC ] .
```

`postgresql`

```EBNF
order_by_clause = ORDER BY sort_fields .
sort_fields     = sort_field { "," sort_field } .
sort_field      = identify [ ASC | DESC | USING operator ] [ NULL { FIRST | LAST } ].
operator        = "<" | ">" | "<=" | ">=" | "=" .
```

`Conclusion`

> Conflict: ***NO***

## LIMIT_CLAUSE

`influxdb`

```EBNF
limit_clause    = LIMIT int_lit .
```

`postgresql`

```EBNF
limit_clause    = LIMIT ( int_lit | ALL ).
```

`Conclusion`

> Conflict: ***NO***

## OFFSET_CLAUSE

`influxdb`

```EBNF
offset_clause   = OFFSET int_lit .
```

`postgresql`

```EBNF
offset_clause   = OFFSET int_lit ( ROW | ROWS ) FETCH ( FIRST | NEXT ) [ int_lit ] ( ROW | ROWS ) ( ONLY | WITH TIES ) .
```

`Conclusion`

> Conflict: ***NO***

## SLIMIT_CLAUSE

`influxdb`

```EBNF
slimit_clause    = SLIMIT int_lit .
```

`postgresql`

***Not supported***

`Conclusion`

> Conflict: ***YES***

## SOFFSET_CLAUSE

`influxdb`

```EBNF
soffset_clause   = SOFFSET int_lit .
```

`postgresql`

***Not supported***

`Conclusion`

> Conflict: ***YES***

## TIMEZONE_CLAUSE

`influxdb`

```EBNF
timezone_clause = tz(string_lit) .
```

`postgresql`

***Not supported***

`Conclusion`

> Conflict: ***YES***
