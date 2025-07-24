# Elasticsearch Query 

## Query Types

### 1. Match Queries (Full-Text Search)
- Designed for searching analyzed text fields.
- Finds documents matching a text phrase or terms, with support for language analysis.
- **Variants:**
  - `match`: Basic full-text search.
  - `match_phrase`: Exact phrase search.
  - `multi_match`: Search across multiple fields.

```json
{
  "query": {
    "match": {
      "description": "server error"
    }
  }
}
```

---

### 2. Term Queries (Exact Match)
- Used for exact value matching (not analyzed).
- Ideal for keyword, numeric, or boolean fields.
- **Variants:**
  - `term`: Single value.
  - `terms`: Multiple values.

```json
{
  "query": {
    "term": {
      "status": "SUCCESS"
    }
  }
}
```

---

### 3. Bool Queries (Boolean Combinations)
- Combine multiple queries using boolean logic.
- **Operators:**
  - `must`: All conditions must match (AND).
  - `should`: At least one should match (OR).
  - `must_not`: Exclude documents.
  - `filter`: Like `must`, but does not affect scoring.

```json
{
  "query": {
    "bool": {
      "must": [
        { "term": { "status": "ERROR" }},
        { "range": { "timestamp": { "gte": "now-1d/d" }}}
      ],
      "must_not": [
        { "term": { "isHidden": true }}
      ]
    }
  }
}
```

---

### 4. Range Queries
- Query numeric, date, or string ranges.
- Supports `gt`, `gte`, `lt`, `lte` for flexible range filtering.

```json
{
  "query": {
    "range": {
      "timestamp": {
        "gte": "2024-01-01",
        "lt": "2024-02-01"
      }
    }
  }
}
```

---

### 5. Exists Queries
- Find documents where a field exists (or is missing).

```json
{
  "query": {
    "exists": {
      "field": "user_id"
    }
  }
}
```

---

### 6. Prefix, Wildcard, and Regexp Queries
- Pattern-based text matching.
- **Variants:**
  - `prefix`: Terms starting with a prefix.
  - `wildcard`: Use `*` or `?` for flexible matching.
  - `regexp`: Regular expression patterns.

```json
{
  "query": {
    "wildcard": {
      "username": "adm*n"
    }
  }
}
```

---

### 7. Fuzzy Query
- Finds terms similar to the query term (handles typos and misspellings).

```json
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "jon",
        "fuzziness": 2
      }
    }
  }
}
```

---

### 8. Query String and Simple Query String
- Allows complex search syntax in a single string.
- Supports logical operators, wildcards, and grouping.

```json
{
  "query": {
    "query_string": {
      "query": "(error OR failure) AND NOT isHidden:true"
    }
  }
}
```

---

### 9. Function Score Query
- Modify document scores using custom functions (e.g., boost recent documents).
- Useful for advanced ranking and personalization.

```json
{
  "query": {
    "function_score": {
      "query": { "match": { "title": "elasticsearch" } },
      "boost": 5,
      "functions": [
        {
          "filter": { "term": { "is_recent": true } },
          "weight": 2
        },
        {
          "gauss": {
            "created_at": {
              "origin": "now",
              "scale": "10d",
              "offset": "5d",
              "decay": 0.5
            }
          }
        }
      ],
      "score_mode": "sum",
      "boost_mode": "multiply"
    }
  }
}
```

---

### 10. Nested Query
- Query nested objects or arrays of objects inside documents.
- Required for fields mapped as `nested` type.

```json
{
  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "bool": {
          "must": [
            { "match": { "comments.author": "john" } },
            { "range": { "comments.likes": { "gte": 10 } } }
          ]
        }
      },
      "score_mode": "avg"
    }
  }
}
```

---

## Advanced Query Types

### 11. Aggregations
- Used to summarize, group, and analyze data (counts, averages, min/max, histograms, etc.).
- Can be combined with queries for filtered analytics.

```json
{
  "size": 0,
  "aggs": {
    "status_count": {
      "terms": { "field": "status.keyword" }
    }
  }
}
```

---

### 12. Highlighting
- Highlight matching terms in search results for better UX.

```json
{
  "query": {
    "match": { "content": "elasticsearch" }
  },
  "highlight": {
    "fields": {
      "content": {}
    }
  }
}
```

---

### 13. Script Queries
- Use custom scripts for advanced filtering or scoring.
- Useful for dynamic calculations or business logic.

```json
{
  "query": {
    "script": {
      "script": {
        "source": "doc['price'].value > params.min_price",
        "params": { "min_price": 100 }
      }
    }
  }
}
```

---

### 14. Geo Queries
- Search and filter by geographic location (points, shapes, distance).

```json
{
  "query": {
    "geo_distance": {
      "distance": "10km",
      "location": {
        "lat": 40.7128,
        "lon": -74.0060
      }
    }
  }
}
```

---

### 15. Percolate Query
- Register queries and match incoming documents against them (reverse search).

```json
{
  "query": {
    "percolate": {
      "field": "query",
      "document": {
        "message": "Elasticsearch error detected"
      }
    }
  }
}
```

---

## Document Operations: Add, Update, Delete

Elasticsearch is not just for searching—you can also add, update, and delete documents using its RESTful API. Here’s how:

### 1. Add (Index) a Document
- Use the `POST` or `PUT` method on the index endpoint.
- If you use `POST`, Elasticsearch auto-generates an ID. With `PUT`, you specify the ID.

**Auto-generated ID:**
```bash
POST /my-index/_doc
{
  "user": "alice",
  "message": "Hello, world!"
}
```

**Custom ID:**
```bash
PUT /my-index/_doc/1
{
  "user": "bob",
  "message": "Custom ID example"
}
```

### 2. Update a Document
- Use the `_update` endpoint with a script or partial document.

```bash
POST /my-index/_doc/1/_update
{
  "doc": {
    "message": "Updated message"
  }
}
```

### 3. Delete a Document
- Use the `DELETE` method with the document ID.

```bash
DELETE /my-index/_doc/1
```

### 4. Bulk Operations
- Use the `_bulk` endpoint to add, update, or delete multiple documents in one request.

```bash
POST /my-index/_bulk
{ "index": { "_id": "1" } }
{ "user": "alice", "message": "Bulk add" }
{ "delete": { "_id": "2" } }
{ "update": { "_id": "3" } }
{ "doc": { "message": "Bulk update" } }
```

---

## Pagination in Elasticsearch

Efficiently retrieving large result sets requires proper pagination. Elasticsearch supports several pagination methods:

### 1. from/size (Basic Pagination)
- Use `from` to skip a number of results and `size` to limit the number returned.
- Best for small result sets (not recommended for deep pagination).

```json
{
  "from": 10,
  "size": 10,
  "query": {
    "match_all": {}
  }
}
```

### 2. search_after (Deep Pagination)
- Use for deep pagination with sorted results.
- Requires a unique, sequential sort key (e.g., timestamp + _id).
- Pass the last sort values from the previous page in `search_after`.

```json
{
  "size": 10,
  "query": {
    "match_all": {}
  },
  "sort": [
    { "timestamp": "asc" },
    { "_id": "asc" }
  ],
  "search_after": [1680000000000, "doc_id_10"]
}
```

### 3. Scroll API (Large Exports)
- Use for exporting or processing large numbers of documents.
- Not intended for real-time user-facing pagination.

```bash
POST /my-index/_search?scroll=1m
{
  "size": 100,
  "query": {
    "match_all": {}
  }
}
```
- Use the returned `_scroll_id` to fetch the next batch:
```bash
POST /_search/scroll
{
  "scroll": "1m",
  "scroll_id": "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAA..."
}
```

### Best Practices
- Use `from/size` for shallow pagination (first few pages).
- Use `search_after` for deep pagination (large offsets).
- Use `scroll` for batch processing or exports, not for user interfaces.

---

## Query Execution Methods

- **Search API (`_search`)**: General-purpose search endpoint.
- **Count API (`_count`)**: Returns the count of matching documents.
- **Multi-search (`_msearch`)**: Run multiple queries in a single request.
- **Scroll API**: Efficiently retrieve large result sets in batches.
- **Search After**: Pagination using a sort key for deep pagination.
- **Aggregations**: Summarize and analyze data (counts, averages, histograms, etc.).

---

## Summary Table

| Query Type      | Use Case                          | Example Field Type        |
| --------------- | --------------------------------- | ------------------------- |
| Match           | Full-text search with analysis    | text                      |
| Term            | Exact value match                 | keyword, boolean, numeric |
| Bool            | Combine queries with AND, OR, NOT | Any                       |
| Range           | Numeric/date range queries        | date, numeric             |
| Exists          | Check if field exists             | Any                       |
| Prefix/Wildcard | Pattern matching                  | keyword, text             |
| Fuzzy           | Approximate matching (typos)      | text                      |
| Query String    | Complex search expressions        | text                      |
| Function Score  | Customize scoring                 | Any                       |
| Nested          | Query nested JSON objects         | nested objects            |

---


## Real-World Example: Log Search

Fetch the first 20 error logs for a service, with aggregation and highlighting:

```json
{
  "from": 0,
  "size": 20,
  "query": {
    "bool": {
      "must": [
        { "match": { "log_level": "error" }},
        { "range": { "@timestamp": { "gte": "now-7d/d" }}}
      ],
      "filter": [
        { "term": { "service": "api-gateway" }}
      ]
    }
  },
  "aggs": {
    "top_errors": {
      "terms": { "field": "error_code.keyword", "size": 5 }
    }
  },
  "highlight": {
    "fields": { "message": {} }
  }
}
```

Fetch the next 20 results (page 2):

```json
{
  "from": 20,
  "size": 20,
  "query": {
    "bool": {
      "must": [
        { "match": { "log_level": "error" }},
        { "range": { "@timestamp": { "gte": "now-7d/d" }}}
      ],
      "filter": [
        { "term": { "service": "api-gateway" }}
      ]
    }
  },
  "aggs": {
    "top_errors": {
      "terms": { "field": "error_code.keyword", "size": 5 }
    }
  },
  "highlight": {
    "fields": { "message": {} }
  }
}
```

**Note:**
Using `from`/`size` for pagination can result in missing or duplicate results if new logs are added between queries, because the result set can shift as new documents are indexed. For real-time log pagination, use `search_after` with a stable sort (such as `@timestamp` and `_id`).

Example: Paginate logs using `search_after` (fetch next page after the last result):

```json
{
  "size": 20,
  "query": {
    "bool": {
      "must": [
        { "match": { "log_level": "error" }},
        { "range": { "@timestamp": { "gte": "now-7d/d" }}}
      ],
      "filter": [
        { "term": { "service": "api-gateway" }}
      ]
    }
  },
  "sort": [
    { "@timestamp": "asc" },
    { "_id": "asc" }
  ],
  "search_after": ["2024-06-01T12:00:00.000Z", "doc_id_20"]
}
```

---

## Further Reading
- [Elasticsearch Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)
- [Elasticsearch Aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)
- [Elasticsearch Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html)
- [Elasticsearch Performance Tuning](https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-search-speed.html) 