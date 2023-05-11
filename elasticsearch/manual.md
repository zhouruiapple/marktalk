# 1 Create Index

```json
curl -X PUT "localhost:9200/index01?pretty" -H 'Content-Type: application/json' -d'
{
    "settings": {
        "index": {
            "sort.field": [
                "region",
                "host",
                "app",
                "time"
            ],
            "sort.order": [
                "asc",
                "asc",
                "asc",
                "asc"
            ]
        }
    },
    "mappings": {
        "properties": {
            "region": {
                "type": "keyword"
            },
            "host": {
                "type": "keyword"
            },
            "app": {
                "type": "keyword"
            },
            "time": {
                "type": "long"
            },
            "status": {
                "type": "long"
            },
            "value": {
                "type": "long",
                "index": false
            }
        }
    }
}
'
```

# 2 Insert Index

```json
curl -X POST "localhost:9200/index01/_doc?pretty" -H 'Content-Type: application/json' -d'
{
    "region": "r1",
    "host": "h1",
    "app": "a1",
    "time": 2,
    "status": 1,
    "value": 10
}
'
```

# 3 Search Index

```json
curl -X POST "localhost:9200/index01/_search?request_cache=false&pretty" -H 'Content-Type: application/json' -d'
{
    "size": 0,
    "query": {
        "bool": {
            "filter": [
                {
                    "range": {
                        "time": {
                            "gte": 1
                        }
                    }
                }
            ]
        }
    },
    "aggs": {
        "groupby1": {
            "terms": {
                "field": "region"
            },
            "aggs": {
                "groupby2": {
                    "terms": {
                        "field": "host"
                    },
                    "aggs": {
                        "avg_value": {
                            "avg": {
                                "field": "value"
                            }
                        }
                    }
                }
            }
        }
    }
}
'
```

# 4 Clear Cache

```json
curl -X POST "localhost:9200/_cache/clear?pretty"
```

# 5 Show Settings

```json
curl -X GET "localhost:9200/index01/_settings?pretty"
```

# 6 Show Mapping

```json
curl -X GET "localhost:9200/index01/_mapping?pretty"
```

# 7 Search without Request Cache

```json
curl -X POST "localhost:9200/index01/_search?request_cache=false&pretty" -H 'Content-Type: application/json' -d'
{
    "size": 0,
    "query": {
        "bool": {
            "filter": [
                {
                    "range": {
                        "time.offset": {
                            "gt": 1
                        }
                    }
                }
            ]
        }
    },
    "aggs": {
        "groupby1": {
            "terms": {
                "field": "region",
            },
            "aggs": {
                "groupby2": {
                    "terms": {
                        "field": "host",
                    },
                    "aggs": {
                        "avg_value": {
                            "avg": {
                                "field": "value"
                            }
                        }
                    }
                }
            }
        }
    }
}
'
```

# 8 Search with Request Cache

```json
curl -X POST "localhost:9200/index01/_search?request_cache=true&pretty" -H 'Content-Type: application/json' -d'
{
    "size": 0,
    "query": {
        "bool": {
            "filter": [
                {
                    "range": {
                        "time.offset": {
                            "gt": 1
                        }
                    }
                }
            ]
        }
    },
    "aggs": {
        "groupby1": {
            "terms": {
                "field": "region",
            },
            "aggs": {
                "groupby2": {
                    "terms": {
                        "field": "host",
                    },
                    "aggs": {
                        "avg_value": {
                            "avg": {
                                "field": "value"
                            }
                        }
                    }
                }
            }
        }
    }
}
'
```

# 9 Show Cache Stats

```json
curl -X GET "localhost:9200/_stats/request_cache?pretty"
```

# 19 filter aggregation

```json
curl -X POST "localhost:9200/index01/_search?pretty" -H 'Content-Type: application/json' -d'
{
    "size": 0,
    "query": {
        "bool": {
            "filter": [
                {
                    "range": {
                        "time": {
                            "gte": 1
                        }
                    }
                }
            ]
        }
    },
    "aggs": {
        "groupby1": {
            "terms": {
                "field": "region"
            },
            "aggs": {
                "groupby2": {
                    "terms": {
                        "field": "host"
                    },
                    "aggs": {
                        "avg_value": {
                            "avg": {
                                "field": "value"
                            }
                        }
                    }
                }
            }
        }
    }
}

# Filter Aggregation

```json
curl -X POST "localhost:9200/index01/_search?size=0&request_cache=false&filter_path=aggregations&pretty" -H 'Content-Type: application/json' -d'
{
  "aggs": {
    "avg_price": { "avg": { "field": "value" } },
    "r1": {
      "filter": { "term": { "region": "r1" } },
      "aggs": {
        "avg_price": { "avg": { "field": "value" } }
      }
    }
  }
}
'
```

