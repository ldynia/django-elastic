# Description

Plain django init project.

# Instructions

```bash
$ docker-compose up
```

# Elastic Search

[tutorial](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html)

## Curl
`curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'`

```
<PATH>
    The API endpoint, which can contain multiple components, such as _cluster/stats or _nodes/stats/jvm. 
<QUERY_STRING>
    Any optional query-string parameters. For example, ?pretty will pretty-print the JSON response to make it easier to read. 
<BODY>
    A JSON-encoded request body (if necessary). 
```

```bash
$ curl -X GET "http://localhost:9200/_cat/nodes?v&pretty"
$ curl -X GET "http://localhost:9200/_cat/health?v=true&pretty"
```

## Interact with ES

```bash
# Create
$ curl -X PUT "http://localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'{ "name": "John Doe" }'
$ curl -X PUT "http://localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'{ "name": "John Doe" }'

# Get all indexes
$ curl "http://localhost:9200/_cat/indices?v=true"
$ curl "http://localhost:9200/_aliases?pretty=true"

# Read
$ curl -X GET "http://localhost:9200/customer/_doc/1?pretty"
$ curl -X GET "http://localhost:9200/customer/_doc/2?pretty"

# Bulk create
$ wget https://raw.githubusercontent.com/elastic/elasticsearch/master/docs/src/test/resources/accounts.json
$ curl https://raw.githubusercontent.com/elastic/elasticsearch/master/docs/src/test/resources/accounts.json --output accounts.json
$ curl -H "Content-Type: application/json" -XPOST "http://localhost:9200/bank/_bulk?pretty&refresh" --data-binary "@accounts.json"
$ curl -XGET "http://localhost:9200/bank/_doc/995?pretty"
$ curl "http://localhost:9200/_cat/indices?v=true"

# Searching
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ],
  "from": 10,
  "size": 1
}
'

# match
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "address": "mill lane" } }
}
'

# match_phrase
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
'

# term
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "term": { "employer": "neteria" } }
}
'

# exclude Idiho
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
'

# between & sort 
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "sort": [
    { "balance": "desc"}
  ],
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
'
# group_by_state
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
'

# average balance per state
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
'
```

https://www.elastic.co/blog/multi-field-search-just-got-better