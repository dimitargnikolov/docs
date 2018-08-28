# Table of Contents

* [Table of Contents](#table-of-contents)
* [Starting](#starting)
   * [Guided](#guided)
* [Stopping](#stopping)
* [Config](#config)
   * [Add Schema Field](#add-schema-field)
   * [Add Copy Field](#add-copy-field)
* [Collections](#collections)
   * [Create](#create)
   * [Delete](#delete)
	  * [Collection](#collection)
	  * [Document by ID](#document-by-id)
	  * [Documents by Query](#documents-by-query)
* [Indexing](#indexing)
   * [In Bulk](#in-bulk)
* [Query Examples](#query-examples)
   * [Basic Search](#basic-search)
   * [Field Search](#field-search)
   * [Phrase Search](#phrase-search)
   * [Combined Searches](#combined-searches)
	  * [AND](#and)
	  * [NOT](#not)
   * [Faceted Search](#faceted-search)
	  * [Range Facets](#range-facets)
	  * [Pivot Facets](#pivot-facets)

# Starting
```
$ solr start -c -p 8983 -s example/cloud/node1/solr
$ solr start -c -p 7574 -s example/cloud/node2/solr -z localhost:9983
```

## Guided
```
$ solr start -e cloud
```

# Stopping
```
$ solr stop -all
```

# Config

A `configSet` includes at a minimum:
1. schema file (named either `managed-schema` or `schema.xml`)
2. `solrconfig.xml`

## Add Schema Field
```
$ curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"name", "type":"text_general", "multiValued":false, "stored":true}}' http://localhost:8983/solr/films/schema
```

## Add Copy Field
A copy field can be set up to take all data from all fields and index it into a new field, which we can set up as the default to search against.

```
$ curl -X POST -H 'Content-type:application/json' --data-binary '{"add-copy-field" : {"source":"*","dest":"_text_"}}' http://localhost:8983/solr/films/schema
```

# Collections

## Create
```
$ solr create -c <collection_name> -s 2 -rf 2
```

## Delete

### Collection
```
$ solr delete -c <collection_name>
```

### Document by ID
Via `post`:
```
$ post -c techproducts -d "<delete><id>TEST1</id></delete>"
```

Via `curl`:
```
$ curl http://localhost:8983/solr/techproducts/update?commit=true -H 'Content-type:application/json' -d '{"delete": {"id": "TEST1"}}'
```

### Documents by Query
Via `post`:
```
$ post -c techproducts -d "<delete><query>*:*</query></delete>"
```

Via `curl`:
```
$ curl http://localhost:8983/solr/techproducts/update?commit=true -H 'Content-type:application/json' -d '{"delete": {"query": "*:*"}}'
```

# Index/Add Document

## In Bulk
Via `post`:
```
$ post -c techproducts $SOLR_HOME/example/exampledocs/*
```

Via `curl`:
```
$ curl http://localhost:8983/solr/techproducts/update?commit=true -H 'Content-type: application/json' -d '[{"id": "TEST1", "cat": ["electronics"], "name": "Test Product 1"}]'
```

# Update Document

## Atomic Update
```
$ curl http://localhost:8983/solr/techproducts/update?commit=true -H 'Content-type: application/json' -d '[{"id": "TEST1", "cat": {"add-distinct": ["electronics", "toys"]}, "name": {"set": "Test Product 1 Foundation"}}]'
```

# Query Examples

## Basic Search
```
$ curl "http://localhost:8983/solr/techproducts/select?indent=on&q=*:*"
$ curl "http://localhost:8983/solr/techproducts/select?q=foundation"
```

## Field Search
```
$ curl "http://localhost:8983/solr/techproducts/select?q=cat:electronics"
```

## Phrase Search
```
$ curl "http://localhost:8983/solr/techproducts/select?q=\"CAS+latency\""
```

## Combined Searches

### AND
For documents containing both `music` and `electronics`, we would have to query with `+music` and `+electronics` and encode the `+` to `%2B` (since `+` denotes a space in a URL):

```
$ curl "http://localhost:8983/solr/techproducts/select?q=%2Belectronics%20%2Bmusic"
```

### NOT
Documents containing `electronics` but not `music`:

```
$ curl "http://localhost:8983/solr/techproducts/select?q=%2Belectronics+-music"
```

## Faceted Search
```
$ curl "http://localhost:8983/solr/films/select?q=*:*&rows=0&facet=true&facet.field=genre_str"
```

### Range Facets
```
$ curl "http://localhost:8983/solr/films/select?q=*:*&rows=0&facet=true&facet.range=initial_release_date&facet.range.start=NOW-20YEAR&facet.range.end=NOW&facet.range.gap=%2B1YEAR"
```
### Pivot Facets
```
$ curl "http://localhost:8983/solr/films/select?q=*:*&rows=0&facet=on&facet.pivot=genre_str,directed_by_str"
```
