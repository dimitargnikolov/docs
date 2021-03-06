# Useful resources
1. https://www.w3.org/TR/rdf11-primer
2. https://www.w3.org/TR/ldp-primer/
3. https://wiki.duraspace.org/display/FEDORA474/First+Steps
4. https://wiki.duraspace.org/display/FEDORA474/The+Fedora+4+object+model
5. https://wiki.duraspace.org/display/FEDORA474/RESTful+HTTP+API
6. http://fedora.info/definitions/v4/2016/10/18/repository

# Fedora start/stop

## Start
```
daemon.sh start
```

## Monitor logs
```
tail -f $CATALINA_HOME/logs/catalina-daemon.out
```

## Stop
```
daemon.sh stop
```

# Use cases

## Read

1. Read the contents (RDF metadata) of a container node.
  ```
curl -v -X GET \
-H "Accept: application/ld+json;profile=\"http://www.w3.org/ns/json-ld#flattened\"" \
http://localhost:8080/fcrepo-webapp-4.7.4/rest/ \
| python -m json.tool
  ```
2. Read the contents of a resource node.
  ```
curl -v -X GET
http://localhost:8080/fcrepo-webapp-4.7.4/rest/img1
  ```

## Create nodes with custom names or metadata

1. Create a container node with user-specified name, under a user-specified parent node.
  ```
curl -v -X PUT http://localhost:8080/fcrepo-webapp-4.7.4/rest/albums
curl -v -X PUT http://localhost:8080/fcrepo-webapp-4.7.4/rest/albums/a1
  ```

2. Create a container node with custom metadata.
  ```
curl -v -X PUT -H "Content-Type: text/turtle" -d "" "http://localhost:8080/rest/node/to/update"
curl -v -X PUT http://localhost:8080/fcrepo-webapp-4.7.4/rest/albums/a1/im1c
  ```

3. Create a resource node with user-specified name, under a user-specified parent node.
  ```
curl -v -X PUT \
--upload-file test.png \
http://localhost:8080/fcrepo-webapp-4.7.4/rest/albums/a1/im1c/im1
  ```
  
4. Create a resource node with custom metadata. Is that possible or do you need to wrap the resource node in a container node?
  ```
  ```

## Create nodes with auto-assigned names

```
curl -v -X POST http://localhost:8080/fcrepo-webapp-4.7.4/rest/
```

## Delete a node
```
curl -v -X DELETE http://localhost:8080/fcrepo-webapp-4.7.4/rest/image1
```
