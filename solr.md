# Starting
```
$ solr start -c -p 8983 -s example/cloud/node1/solr
$ solr start -c -p 7574 -s example/cloud/node2/solr -z localhost:9983
```

# Stopping
```
$ solr stop -all
```

# Collections

Create: 
```
$ solr create -c <collection_name> -s 2 -rf 2
```

Delete:
```
$ solr delete -c <collection_name>
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

For documents containing both `music` and `electronics`, we would have to query with `+music` and `+electronics` and encode the `+` to `%2B` (since `+` denotes a space in a URL):

```
$ curl "http://localhost:8983/solr/techproducts/select?q=%2Belectronics%20%2Bmusic"
```

Documents containing `electronics` but not `music`:

```
$ curl "http://localhost:8983/solr/techproducts/select?q=%2Belectronics+-music"
```

1. Download generic Linux tarball, untar, define MYSQL_HOME, update PATH.
2. Run
```
$ mysqld --initialize --basedir=$MYSQL_HOME --datadir=$MYSQL_HOME/data
```

