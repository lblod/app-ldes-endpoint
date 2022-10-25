# 1. app-ldes-endpoint

This app allows you to publish your data as linked data fragments. 

## 1.1 Used services
1. https://github.com/redpencilio/fragmentation-producer-service


## 2. REST API

### **GET** /:folder/:subfolder?/:nodeId

this endpoint allows you to query a specific node represented in an RDF format to your liking. Using the HTTP Accept header, you can provide which representation of the data you would like to receive. Typically, the view node of the dataset is located in /:folder/1 while the other nodes are additionally stored in subfolders.

#### Request body
 N/A


#### Response

  <h5> 201 Created </h5>

  ```javascript
  {
      "data": {
          "type": "public-service",
          "id": "{NewPublicServiceId}",
          "uri": "http://data.lblod.info/id/public-services/{NewPublicServiceId}"
      }
  }
  ```

### **POST** /:folder
allows you to add a new resource to a dataset located in folder. The post body can containing the resource in any RDF format to your liking. The post body format should be supplied using the Content-Type HTTP header. This endpoints expects the following query parameters:
- resource: <resource>: the id of the resource which is to be added
- fragmenter: <fragmenter_type>: the type of fragmenter to use, defaults to 
- time-fragmenter. The other option is prefix-tree-fragmenter.

#### Request body
 N/A


#### Response

  <h5> 201 Created </h5>

  ```javascript
  {
      "data": {
          "type": "public-service",
          "id": "{NewPublicServiceId}",
          "uri": "http://data.lblod.info/id/public-services/{NewPublicServiceId}"
      }
  }
  ```

## RDF formats
This section describes the existing RDF formats and their content-type

### Turtle (ttl)
#### File extention
```.ttl```
#### Content-type
```text/turtle```
#### Example file
```turtle
<https://example.org/person/80325> a <http://xmlns.com/foaf/0.1/Person>;
        <http://xmlns.com/foaf/0.1/name> "John Lennon";
        <http://schema.org/birthDate> "1940-10-09";
        <http://schema.org/spouse> "http://dbpedia.org/resource/Cynthia_Lennon".

```
#### W3C Spec sheat
https://www.w3.org/TR/turtle/

### JSON-LD (tt)
#### File extention
```.jsonld```
#### Content-type
```application/ld+json```
#### Example file
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
#### W3C Spec sheat
https://www.w3.org/TR/json-ld11/

#### Playground environment
https://json-ld.org/playground/