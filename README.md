# 1. app-ldes-endpoint

This app allows you to publish your data as linked data fragments. 

# 2. Used services
1. https://github.com/redpencilio/fragmentation-producer-service


# 3. REST API

## 3.1 **GET** /:folder/:subfolder?/:nodeId

this endpoint allows you to query a specific node represented in an RDF format to your liking. Using the HTTP Accept header, you can provide which representation of the data you would like to receive. Typically, the view node of the dataset is located in /:folder/1 while the other nodes are additionally stored in subfolders.

### 3.1.1 Request body
 N/A


### 3.1.2  Response body

  <h5> 200 OK </h5>

  ```javascript
{
    "@context": {
        "member": {
            "@id": "https://w3id.org/tree#member",
            "@type": "@id"
        },
        "relation": {
            "@id": "https://w3id.org/tree#relation",
            "@type": "@id"
        },
        "view": {
            "@id": "https://w3id.org/tree#view",
            "@type": "@id"
        },
        "generatedAtTime": {
            "@id": "http://www.w3.org/ns/prov#generatedAtTime",
            "@type": "http://www.w3.org/2001/XMLSchema#dateTime"
        },
        "isVersionOf": {
            "@id": "http://purl.org/dc/terms/isVersionOf",
            "@type": "@id"
        },
        "tree": "https://w3id.org/tree#",
        "ex": "http://example.org/",
        "ldes": "http://w3id.org/ldes#",
        "schema": "http://schema.org/",
        "uuid": "http://mu.semte.ch/vocabularies/core/uuid",
        "node": {
            "@id": "tree:node",
            "@type": "@id"
        },
        "path": {
            "@id": "tree:path",
            "@type": "@id"
        }
    },
    "@graph": [
        {
            "@id": "http://data.lblod.info/streams/lpdc/lpdc",
            "@type": [
                "ldes:EventStream",
                "tree:Collection"
            ],
            "member": {
                "@id": "http://mu.semte.ch/services/ldes-time-fragmenter/versioned/fa670909-8644-42c6-aa74-8f4378cc909a",
                "isVersionOf": {
                    "@id": "http://localhost:80/lpdc/1",
                    "@type": "tree:Node"
                },
                "generatedAtTime": "2022-10-25T13:57:30.799Z"
            },
            "view": "http://localhost:80/lpdc/1"
        },
        {
            "@id": "http://localhost:80/lpdc/1",
            "@type": "tree:Node"
        },
        {
            "@id": "http://mu.semte.ch/services/ldes-time-fragmenter/versioned/fa670909-8644-42c6-aa74-8f4378cc909a",
            "isVersionOf": {
                "@id": "http://localhost:80/lpdc/1",
                "@type": "tree:Node"
            },
            "generatedAtTime": "2022-10-25T13:57:30.799Z"
        },
        {
            "@id": "https://example.org/movies/800005",
            "@type": "https://example.org/Movie",
            "uuid": "01323132&",
            "https://example.org/genres": "Comedy|Romance|Action",
            "https://example.org/name": "My new movie"
        }
    ]
}
  ```

## 3.2 **POST** /:folder
allows you to add a new resource to a dataset located in folder. The post body can containing the resource in any RDF format to your liking. The post body format should be supplied using the Content-Type HTTP header. This endpoints expects the following query parameters:
- resource: <resource>: the id of the resource which is to be added
- fragmenter: <fragmenter_type>: the type of fragmenter to use, defaults to 
- time-fragmenter. The other option is prefix-tree-fragmenter.

### Request body
 N/A


### Response body

  <h5> 201 Created </h5>

  ```javascript
  {
      "message": "ok",
      "membersInPage": 1
  }
  ```

# 4. RDF formats
This section describes the existing RDF formats and their content-type

## 4.1 Turtle (ttl)
### File extention
```.ttl```
### Content-type
```text/turtle```
### Example file
```turtle
<https://example.org/person/80325> a <http://xmlns.com/foaf/0.1/Person>;
        <http://xmlns.com/foaf/0.1/name> "John Lennon";
        <http://schema.org/birthDate> "1940-10-09";
        <http://schema.org/spouse> "http://dbpedia.org/resource/Cynthia_Lennon".

```
### W3C Spec sheat
https://www.w3.org/TR/turtle/

## 4.2 JSON-LD (tt)
### File extention
```.jsonld```
### Content-type
```application/ld+json```
### Example file
```json
  {
    "@context": "https://json-ld.org/contexts/person.jsonld",
    "@id": "https://example.org/person/80325",
    "@type": "Person",
    "name": "John Lennon",
    "born": "1940-10-09",
    "spouse": "http://dbpedia.org/resource/Cynthia_Lennon"
  }
```
### W3C Spec sheat
https://www.w3.org/TR/json-ld11/

### Playground environment
https://json-ld.org/playground/
