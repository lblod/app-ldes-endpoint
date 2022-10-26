# 1. app-ldes-endpoint

This app allows you to publish your data as linked data fragments. 

# 2. Used services
1. https://github.com/redpencilio/fragmentation-producer-service


# 3. REST API

## 3.1 **GET** /:folder/:subfolder?/:nodeId

this endpoint allows you to query a specific node represented in an RDF format to your liking. Using the HTTP Accept header, you can provide which representation of the data you would like to receive. Typically, the view node of the dataset is located in /:folder/1 while the other nodes are additionally stored in subfolders.

### Request body
 N/A


### Response body

  <h5> 200 OK </h5>
<small> x</small>

  ```javascript
  // This is an example. Content-type: application/ld+json
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
Allows you to add a new resource to a dataset located in folder. The post body can containing the resource in any RDF format off your liking. The post body format should be supplied using the Content-Type HTTP header. 

**This endpoints expects the following query parameters:**
| Param  | description | required
|---|---|---|
| resource | the id of the resource which is to be added | yes | 
| fragmenter | the type of fragmenter to use, defaults to "time-fragmenter". The other option is "prefix-tree-fragmenter". | no |


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
This playground allows you to check your json-ld for errors and additionaly has some examples to help you get familiar with the format. Tip: use the table or visualized view tab in the results to see if the json-ld was mapped correctly. 

### @id & @type
@id is a special key. Special keys are prepended by the @ symbol. @id is teling us who the following data is about (the subject). While @type tells us what type the subject is (type person). Translated to turtle we get the following:

```
<https://example.org/person/80325> a <http://xmlns.com/foaf/0.1/Person>;
```

<small>note: `a` is the shorthand of `rdf:type`</small>

### @context
note: This service prefers having the context inline. So avoiding external links.

JSON-LD should always come with a @context. The context allows regular json to easily translate to RDF. A context is either inline or an external link. In this case the context points to the following url `https://json-ld.org/contexts/person.jsonld`. 

When opening `https://json-ld.org/contexts/person.jsonld` we find a page with the context. This page describes how the key value pairs translate to RDF. An example being the "born" key. When looking up this key in the context file we see this translates to  `http://schema.org/birthDate` with the value being of type `xsd:date`. 

### Gotcha
Make sure all items are correctly spelled. If no mapping is found in the @context for a certain key, it will NOT show in the results (unless there is a top-level @id specified in the @context) 

Example: If you make a typo and "born" is spelled "boorn" instead, it will not show in the output results as there only exists a mapping for "born" in the @context and no default namespace (top-level @id) has been specified.

# 5. Examples

## POST a mandataris in turtle format

POST `/mandataris?resource=600728030CCED4000800026A` <br>
Content-type: `text/turtle`

### Request body
```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>


<http://data.lblod.info/id/mandatarissen/600728030CCED4000800026A>	rdf:type	<http://data.vlaanderen.be/ns/mandaat#Mandataris> ;
	<http://mu.semte.ch/vocabularies/core/uuid>	"600728030CCED4000800026A" ;
	<http://data.vlaanderen.be/ns/mandaat#start>	"2021-01-01T00:00:00Z"^^xsd:dateTime ;
	<http://data.vlaanderen.be/ns/mandaat#isBestuurlijkeAliasVan>	<http://data.lblod.info/id/personen/2b49b50f-4b52-47ba-af19-f5a6d381e694> ;
	<http://data.vlaanderen.be/ns/mandaat#rangorde>	"Tweede schepen" ;
	<http://data.vlaanderen.be/ns/mandaat#status> <http://data.lblod.info/id/bestuurseenheden/e1ca6edd-55e1-4288-92a5-53f4cf71946a> .

<http://data.lblod.info/id/personen/2b49b50f-4b52-47ba-af19-f5a6d381e694> a <http://www.w3.org/ns/person#Person>;
   <http://data.vlaanderen.be/ns/persoon#gebruikteVoornaam> "Jane";
   <http://data.vlaanderen.be/ns/persoon#geslacht> <http://publications.europa.eu/resource/authority/human-sex/FEMALE>;
   <http://mu.semte.ch/vocabularies/core/uuid> "2b49b50f-4b52-47ba-af19-f5a6d381e694";
   <http://xmlns.com/foaf/0.1/familyName> "Doe" .
```

### Response body
```
{
    "message": "ok",
    "membersInPage": 1
}
```

## POST a mandataris in JSON-LD format

POST `/mandataris?resource=600728030CCED4000800026A` <br>
Content-type: `application/ld+json`

### Request body
```
{
  "@context": {
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "xsd": "http://www.w3.org/2001/XMLSchema#"
  },
  "@graph": [
    {
      "@id": "http://data.lblod.info/id/mandatarissen/600728030CCED4000800026A",
      "rdf:type": {
        "@id": "http://data.vlaanderen.be/ns/mandaat#Mandataris"
      },
      "http://mu.semte.ch/vocabularies/core/uuid": "600728030CCED4000800026A",
      "http://data.vlaanderen.be/ns/mandaat#start": {
        "@value": "2021-01-01T00:00:00Z",
        "@type": "xsd:dateTime"
      },
      "http://data.vlaanderen.be/ns/mandaat#isBestuurlijkeAliasVan": {
        "@id": "http://data.lblod.info/id/personen/2b49b50f-4b52-47ba-af19-f5a6d381e694"
      },
      "http://data.vlaanderen.be/ns/mandaat#rangorde": "Tweede schepen",
      "http://data.vlaanderen.be/ns/mandaat#status": {
        "@id": "http://data.lblod.info/id/bestuurseenheden/e1ca6edd-55e1-4288-92a5-53f4cf71946a"
      }
    },
    {
      "@id": "http://data.lblod.info/id/personen/2b49b50f-4b52-47ba-af19-f5a6d381e694",
      "@type": "http://www.w3.org/ns/person#Person",
      "http://data.vlaanderen.be/ns/persoon#gebruikteVoornaam": "Jane",
      "http://data.vlaanderen.be/ns/persoon#geslacht": {
        "@id": "http://publications.europa.eu/resource/authority/human-sex/FEMALE"
      },
      "http://mu.semte.ch/vocabularies/core/uuid": "2b49b50f-4b52-47ba-af19-f5a6d381e694",
      "http://xmlns.com/foaf/0.1/familyName": "Doe"
    }
  ]
}
```

### Response body
```
{
    "message": "ok",
    "membersInPage": 1
}
```

## GET First fragment in folder mandataris

GET /:folder/:subfolder?/:nodeId

GET `/mandataris/1` <br>
Accept: `text/turtle`

### Request body
N/A

### Response body
```
<http://data.lblod.info/streams/lpdc/mandataris> a <http://w3id.org/ldes#EventStream>, <https://w3id.org/tree#Collection>;
    <https://w3id.org/tree#view> <http://localhost:80/mandataris/1>.
<http://localhost:80/mandataris/1> a <https://w3id.org/tree#Node>.
<http://data.lblod.info/streams/lpdc/mandataris> <https://w3id.org/tree#member> <http://mu.semte.ch/services/ldes-time-fragmenter/versioned/63066d9d-f454-4f50-8fa6-d8c5121b77bc>.
<http://mu.semte.ch/services/ldes-time-fragmenter/versioned/63066d9d-f454-4f50-8fa6-d8c5121b77bc> <http://purl.org/dc/terms/isVersionOf> <http://localhost:80/mandataris/600728030CCED4000800026A>;
    <http://www.w3.org/ns/prov#generatedAtTime> "2022-10-26T09:26:56.652Z"^^<http://www.w3.org/2001/XMLSchema#dateTime>.
<http://data.lblod.info/streams/lpdc/mandataris> <https://w3id.org/tree#member> <http://mu.semte.ch/services/ldes-time-fragmenter/versioned/251a54f4-9cc2-48a9-93cd-9a0de7273b98>.
<http://mu.semte.ch/services/ldes-time-fragmenter/versioned/251a54f4-9cc2-48a9-93cd-9a0de7273b98> <http://purl.org/dc/terms/isVersionOf> <http://localhost:80/mandataris/600728030CCED4000800026A>;
    <http://www.w3.org/ns/prov#generatedAtTime> "2022-10-26T09:28:33.025Z"^^<http://www.w3.org/2001/XMLSchema#dateTime>.
<http://data.lblod.info/streams/lpdc/mandataris> <https://w3id.org/tree#member> <http://mu.semte.ch/services/ldes-time-fragmenter/versioned/2865aa9d-1e11-42b0-9ac1-c6fcbb51819b>.
<http://data.lblod.info/id/mandatarissen/600728030CCED4000800026A> a <http://data.vlaanderen.be/ns/mandaat#Mandataris>;
    <http://mu.semte.ch/vocabularies/core/uuid> "600728030CCED4000800026A";
    <http://data.vlaanderen.be/ns/mandaat#start> "2021-01-01T00:00:00Z"^^<http://www.w3.org/2001/XMLSchema#dateTime>;
    <http://data.vlaanderen.be/ns/mandaat#isBestuurlijkeAliasVan> <http://data.lblod.info/id/personen/2b49b50f-4b52-47ba-af19-f5a6d381e694>;
    <http://data.vlaanderen.be/ns/mandaat#rangorde> "Tweede schepen";
    <http://data.vlaanderen.be/ns/mandaat#status> <http://data.lblod.info/id/bestuurseenheden/e1ca6edd-55e1-4288-92a5-53f4cf71946a>.
<http://data.lblod.info/id/personen/2b49b50f-4b52-47ba-af19-f5a6d381e694> a <http://www.w3.org/ns/person#Person>;
    <http://mu.semte.ch/vocabularies/core/uuid> "2b49b50f-4b52-47ba-af19-f5a6d381e694";
    <http://data.vlaanderen.be/ns/persoon#gebruikteVoornaam> "Jane";
    <http://data.vlaanderen.be/ns/persoon#geslacht> <http://publications.europa.eu/resource/authority/human-sex/FEMALE>;
    <http://xmlns.com/foaf/0.1/familyName> "Doe".
<http://mu.semte.ch/services/ldes-time-fragmenter/versioned/2865aa9d-1e11-42b0-9ac1-c6fcbb51819b> <http://purl.org/dc/terms/isVersionOf> <http://localhost:80/mandataris/600728030CCED4000800026A>;
    <http://www.w3.org/ns/prov#generatedAtTime> "2022-10-26T09:28:42.794Z"^^<http://www.w3.org/2001/XMLSchema#dateTime>.

```

