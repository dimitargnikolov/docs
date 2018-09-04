# Table of Contents

* [Table of Contents](#table-of-contents)
* [Starting](#starting)
   * [Guided](#guided)
* [Stopping](#stopping)
* [Config and Schema](#config-and-schema)
   * [View Schema](#view-schema)
	  * [All](#all)
	  * [Fields](#fields)
	  * [Dynamic Fields](#dynamic-fields)
	  * [Copy Fields](#copy-fields)
	  * [Field Types](#field-types)
   * [Add Schema Field](#add-schema-field)
   * [Add Copy Field](#add-copy-field)
* [Collections](#collections)
   * [Create](#create)
   * [Delete](#delete)
* [Documents](#documents)
   * [Delete by ID](#delete-by-id)
   * [Delete by Query](#delete-by-query)
   * [Index/Add](#indexadd)
   * [Update](#update)
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

# Config and Schema

A `configSet` includes at a minimum:
1. schema file (named either `managed-schema` or `schema.xml`)
2. `solrconfig.xml`

The schema can be defined either manually or through the Schema API. The latter is preferred, since it allows you to dynamically change the schema. Then solr writes the schema configuration file and we interact with it through the rest API as seen here.

## View Schema

### All
```
$ curl http://localhost:8983/solr/<collection>/schema
```

### Fields
```
$ curl http://localhost:8983/solr/<collection>/schema/fields
```

Or 

```
$ curl http://localhost:8983/solr/<collection>/schema/fields/<field_name>
```

### Dynamic Fields
```
$ curl http://localhost:8983/solr/<collection>/schema/dynamicfields
```

Or 

```
$ curl http://localhost:8983/solr/<collection>/schema/dynamicfields/<field_name>
```

### Copy Fields
```
$ curl http://localhost:8983/solr/<collection>/schema/copyfields
```

### Field Types
```
$ curl http://localhost:8983/solr/<collection>/schema/fieldtypes
```

Or 

```
$ curl http://localhost:8983/solr/<collection>/schema/fieldtypes/<field_type_name>
```

## Add Schema Field
```
$ curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"name", "type":"text_general", "multiValued":false, "stored":true}}' http://localhost:8983/solr/<collection>/schema
```

## Add Copy Field
A copy field can be set up to take all data from all fields and index it into a new field, which we can set up as the default to search against.

```
$ curl -X POST -H 'Content-type:application/json' --data-binary '{"add-copy-field" : {"source":"*","dest":"_text_"}}' http://localhost:8983/solr/<collection>/schema
```

# Collections

## Create
```
$ solr create -c <collection_name> -s 2 -rf 2
```

## Delete
```
$ solr delete -c <collection_name>
```

# Documents

## Delete by ID
Via `post`:
```
$ post -c <collection> -d "<delete><id>TEST1</id></delete>"
```

Via `curl`:
```
$ curl http://localhost:8983/solr/<collection>/update?commit=true -H 'Content-type:application/json' -d '{"delete": {"id": "TEST1"}}'
```

## Delete by Query
Via `post`:
```
$ post -c <collection> -d "<delete><query>*:*</query></delete>"
```

Via `curl`:
```
$ curl http://localhost:8983/solr/<collection>/update?commit=true -H 'Content-type:application/json' -d '{"delete": {"query": "*:*"}}'
```

## Index/Add

Via `post`:
```
$ post -c techproducts $SOLR_HOME/example/exampledocs/*
```

Via `curl`:
```
$ curl http://localhost:8983/solr/techproducts/update?commit=true -H 'Content-type: application/json' -d '[{"id": "TEST1", "cat": ["electronics"], "name": "Test Product 1"}]'
```

## Update
```
$ curl http://localhost:8983/solr/techproducts/update?commit=true -H 'Content-type: application/json' -d '[{"id": "TEST1", "cat": {"add-distinct": ["electronics", "toys"]}, "name": {"set": "Test Product 1 Foundation"}}]'
```

# Query Examples

## Basic Search
```
$ curl "http://localhost:8983/solr/<collection>/select?indent=on&q=*:*"
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
