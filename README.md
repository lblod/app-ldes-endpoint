# app-ldes-endpoint

This app allows you to publish your data as linked data fragments.

# pre-requisites
- [Docker Compose (at least v3.7)](https://docs.docker.com/compose/)
- The host that can access the internet

The application hasn't been tested on any windows operating system.

# What's included?
This repository has two setups. The base of these setups resides in the standard docker-compose.yml.

* *docker-compose.yml* This provides you with the backend components.
* *docker-compose.dev.yml* Provides changes for a good local development setup.
  - publishes the backend services on port 80 directly, so you can navigate the application easily.
  - Note: make sure there are not already services running on these port

# Running
This section contains general information on running and maintaining an installation.

## Running the dev setup

  Execute the following:

      # Clone this repository
      git clone https://github.com/lblod/app-ldes-endpoint

      # Move into the directory
      cd app-ldes-endpoint

      # Start the system
      docker-compose -f docker-compose.yml -f docker-compose.dev.yml up

We don't recommend this setup for production.

## Running the regular setup

  ```
  docker-compose up -d
  ```
  This is likely the way to go for production. The `-d' flag makes sure the application runs in background mode.
  The application starts without publishing any ports. As a best practice, Docker Compose recommends tailoring your deployment configuration in a [docker-compose.override.yml file](https://docs.docker.com/compose/extends/).

To monitor the logs, `docker-compose logs -ft --tail=100` is your friend.

# REST API
By default, this application doesn't contain any data, so you'll have to first add some data before being able to retrieve it.
We'll (briefly) specification the API in this section. Please refer to the examples to see how it would work in practice.

## **GET** /:folder/:subfolder?/:nodeId

This endpoint allows you to query a specific node represented in an RDF format to your liking.
Using the HTTP Accept header, you can provide which representation of the data you would like to receive.
Possible `Accept` values: `application/ld+json` (default) and text/turtle`

Typically, the view node of the dataset is located in /:folder/1, while the other nodes are additionally stored in subfolders.

## **POST** /:folder
Allows you to add a new resource to a dataset located in a folder.
The post body can contain the resource in any RDF format of your liking.
The post body format should be supplied using the Content-Type HTTP header.
Possible options for `Content-Type`: `application/ld+json` or `text/turtle`.

**This endpoint expects the following query parameters:**
| Param  | description | required
|---|---|---|
| resource | the id of the resource which is to be added | yes |
| fragmenter | the type of fragmenter to use, defaults to "time-fragmenter". The other option is "prefix-tree-fragmenter". | no |


# Examples
We assume the service is running on `http://localhost:80/`
### Adding John Lennon as a resource
Using the REST client of your choice and taking the following parameters into account, you can add a resource to the stream:
```
POST http://localhost/example-stream?resource=https://example.org/person/80325
Content-Type: application/ld+json
```
With body
```
  {
    "@context": "https://json-ld.org/contexts/person.jsonld",
    "@id": "https://example.org/person/80325",
    "@type": "Person",
    "name": "John Lennon",
    "born": "2040-10-09",
    "spouse": "http://dbpedia.org/resource/Cynthia_Lennon"
  }
```
This will result in the following response:
`Http status: 201`
and body
```
{
    "message": "ok",
    "membersInPage": 1
}
```
You can check the state of the published resource by going to the first page of the stream: `GET http://localhost/example-stream/1`.
```
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
            "@id": "http://data.lblod.info/streams/lpdc/example-stream",
            "@type": [
                "ldes:EventStream",
                "tree:Collection"
            ],
            "member": [
                {
                    "@id": "http://mu.semte.ch/services/ldes-time-fragmenter/versioned/a446969f-a44d-42f2-9b1e-c96e97c50f24",
                    "@type": "http://xmlns.com/foaf/0.1/Person",
                    "isVersionOf": "https://example.org/person/80325",
                    "schema:birthDate": {
                        "@type": "http://www.w3.org/2001/XMLSchema#date",
                        "@value": "2040-10-09"
                    },
                    "schema:spouse": {
                        "@id": "http://dbpedia.org/resource/Cynthia_Lennon"
                    },
                    "generatedAtTime": "2022-10-27T10:43:18.191Z",
                    "http://xmlns.com/foaf/0.1/name": "John Lennon"
                },
                {
                    "@id": "http://mu.semte.ch/services/ldes-time-fragmenter/versioned/eb843414-1f35-4b6d-9d73-29ce4884d5b2",
                    "@type": "http://xmlns.com/foaf/0.1/Person",
                    "isVersionOf": "https://example.org/person/80325",
                    "schema:birthDate": "1940-10-09",
                    "schema:spouse": "http://dbpedia.org/resource/Cynthia_Lennon",
                    "generatedAtTime": "2022-10-27T10:53:35.197Z",
                    "http://xmlns.com/foaf/0.1/name": "John Lennon"
                }
            ],
            "view": {
                "@id": "http://localhost/example-stream/1",
                "@type": "tree:Node"
            }
        },
        {
            "@id": "http://dbpedia.org/resource/Cynthia_Lennon"
        }
    ]
}
```
More information on navigating pages in LDES, may be found in the spec. See appendix for this.
### Updating John Lennon as a resource.
It turns out John Lennon was published with the wrong birth date. We need to update the publication.
Let's update this and use the other supported format as an example.
We'll need to publish the *full* resource again, LDES takes care of versioning.

Using the REST client of your choice and taking the following parameters into account, you can add a resource to the stream:
```
POST http://localhost/example-stream?resource=https://example.org/person/80325
Content-Type: text/turtle
```
With body
```
<https://example.org/person/80325> a <http://xmlns.com/foaf/0.1/Person>;
        <http://xmlns.com/foaf/0.1/name> "John Lennon";
        <http://schema.org/birthDate> "1940-10-09"^^<http://www.w3.org/2001/XMLSchema#date>;
        <http://schema.org/spouse> <http://dbpedia.org/resource/Cynthia_Lennon>.

```
This will result in the following response:
`Http status: 201`
and body
```
{
    "message": "ok",
    "membersInPage": 2
}
```
Again, you can check the state of the published resource by going to the first page of the stream: `GET http://localhost/example-stream/1`.
Let's ask for `text/turtle` this time,
```
<http://data.lblod.info/streams/lpdc/example-stream> a <http://w3id.org/ldes#EventStream>, <https://w3id.org/tree#Collection>;
    <https://w3id.org/tree#view> <http://localhost/example-stream/1>.
<http://localhost/example-stream/1> a <https://w3id.org/tree#Node>.
<http://data.lblod.info/streams/lpdc/example-stream> <https://w3id.org/tree#member> <http://mu.semte.ch/services/ldes-time-fragmenter/versioned/a446969f-a44d-42f2-9b1e-c96e97c50f24>.
<http://mu.semte.ch/services/ldes-time-fragmenter/versioned/a446969f-a44d-42f2-9b1e-c96e97c50f24> a <http://xmlns.com/foaf/0.1/Person>;
    <http://xmlns.com/foaf/0.1/name> "John Lennon";
    <http://schema.org/birthDate> "2040-10-09"^^<http://www.w3.org/2001/XMLSchema#date>;
    <http://schema.org/spouse> <http://dbpedia.org/resource/Cynthia_Lennon>;
    <http://purl.org/dc/terms/isVersionOf> <https://example.org/person/80325>;
    <http://www.w3.org/ns/prov#generatedAtTime> "2022-10-27T10:43:18.191Z"^^<http://www.w3.org/2001/XMLSchema#dateTime>.
<http://data.lblod.info/streams/lpdc/example-stream> <https://w3id.org/tree#member> <http://mu.semte.ch/services/ldes-time-fragmenter/versioned/eb843414-1f35-4b6d-9d73-29ce4884d5b2>.
<http://mu.semte.ch/services/ldes-time-fragmenter/versioned/eb843414-1f35-4b6d-9d73-29ce4884d5b2> a <http://xmlns.com/foaf/0.1/Person>;
    <http://xmlns.com/foaf/0.1/name> "John Lennon";
    <http://schema.org/birthDate> "1940-10-09"^^<http://www.w3.org/2001/XMLSchema#date>;
    <http://schema.org/spouse> <http://dbpedia.org/resource/Cynthia_Lennon>;
    <http://purl.org/dc/terms/isVersionOf> <https://example.org/person/80325>;
    <http://www.w3.org/ns/prov#generatedAtTime> "2022-10-27T10:53:35.197Z"^^<http://www.w3.org/2001/XMLSchema#dateTime>.
```
Clients knowing the LDES-spec, will automatically deduce this correction and update whatever they need to update accordingly.
In a nutshell: you see two versions of the same resource, one published later than the other. Hence the client can assume the most recent one is the correct one.

### Adding a mandataris
Let's add some extra data: an EredienstMandataris. According to the model, this resource (amongst others) combines a mandaat and a person.
Let's suppose we had to create the person linked to the mandataries. This would require publishing the person in a separate call, so we end up with versioned resources that can be updated later.

Using the REST client of your choice and taking the following parameters into account, you can add a resource to the stream:
```
POST http://localhost/example-stream?resource=http://vendor.dns.entry/id/mandatarissen/6357E15D25AAED09E56A02FB
Content-Type: text/turtle
```
With body
```
<http://vendor.dns.entry/id/mandatarissen/6357E15D25AAED09E56A02FB>  <http://data.vlaanderen.be/ns/mandaat#start>  "2022-10-23T22:00:00Z"^^<http://www.w3.org/2001/XMLSchema#dateTime> ;
  <http://www.w3.org/ns/org#holds>  <http://data.lblod.info/id/mandaten/1b1769f4aa44de6d4b413a441bdeb1d7> ;
<http://data.vlaanderen.be/ns/mandaat#isBestuurlijkeAliasVan>  <http://vendor.dns.entry/id/personen/2b49b50f-4b52-47ba-af19-f5a6d381e694> ;
<http://www.w3.org/1999/02/22-rdf-syntax-ns#type>  <http://data.lblod.info/vocabularies/erediensten/EredienstMandataris> .
```
And for the person -just because we can- as JSON-LD,
```
POST http://localhost/example-stream?resource=http://vendor.dns.entry/id/personen/2b49b50f-4b52-47ba-af19-f5a6d381e694
Content-Type: application/ld+json
```
With body
```
{
  "@context": {
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "xsd": "http://www.w3.org/2001/XMLSchema#"
  },
  "@graph": [
    {
      "@id": "http://vendor.dns.entry/id/personen/2b49b50f-4b52-47ba-af19-f5a6d381e694",
      "@type": "http://www.w3.org/ns/person#Person",
      "http://data.vlaanderen.be/ns/persoon#gebruikteVoornaam": "Jane",
      "http://data.vlaanderen.be/ns/persoon#geslacht": {
        "@id": "http://publications.europa.eu/resource/authority/human-sex/FEMALE"
      },
      "http://xmlns.com/foaf/0.1/familyName": "Doe"
    }
  ]
}
```
### Removing a person
Let's say we want to remove a person; this will be handled as a soft delete according to the spec by typing the identifier with a `https://www.w3.org/ns/activitystreams#Tombstone`.
```
POST http://localhost/example-stream?resource=https://example.org/person/80325
Content-Type: application/ld+json
```
With body
```
  {
    "@id": "https://example.org/person/80325",
    "@type": "https://www.w3.org/ns/activitystreams#Tombstone"
  }
```

# Side track: RDF formats
This section gives a bit more info about the discussed RDF formats and their content-type

## Turtle (ttl)
### File extension
"`.ttl` "
### Content-type
```text/turtle```
### Example file
"`turtle
<https://example.org/person/80325> a <http://xmlns.com/foaf/0.1/Person>;
        <http://xmlns.com/foaf/0.1/name> "John Lennon";
        <http://schema.org/birthDate> "1940-10-09";
        <http://schema.org/spouse> "http://dbpedia.org/resource/Cynthia_Lennon".

```
### W3C Spec sheet
https://www.w3.org/TR/turtle/

## JSON-LD (tt)
### File extension
```.jsonld```
### Content-type
```application/ld+json```
### Example file
"`json
  {
    "@context": "https://json-ld.org/contexts/person.jsonld",
    "@id": "https://example.org/person/80325",
    "@type": "Person",
    "name": "John Lennon",
    "born": "1940-10-09",
    "spouse": "http://dbpedia.org/resource/Cynthia_Lennon"
  }
```
### W3C Spec sheet
https://www.w3.org/TR/json-ld11/

### Playground environment
https://json-ld.org/playground/
This playground allows you to check your JSON-LD for errors and additionally has some examples to help you get familiar with the format. Tip: use the table or visualized view tab in the results to see if the JSON-LD was mapped correctly.

### @id & @type
@id is a special key. Special keys are prepended by the @ symbol. @id is telling us who the following data is about (the subject). While @type tells us what type the subject is (type person). Translated to turtle, we get the following:

```
<https://example.org/person/80325> a <http://xmlns.com/foaf/0.1/Person>;
```

<small>note: `a` is the shorthand of `rdf:type`</small>

### @context
note: This service prefers having the context inline. So avoiding external links.

JSON-LD should always come with a @context. The context allows regular JSON to translate to RDF easily. A context is either inline or an external link. In this case, the context points to the following URL `https://json-ld.org/contexts/person.jsonld`.

When opening `https://json-ld.org/contexts/person.jsonld`, we find a page with the context. This page describes how the key-value pairs translate to RDF. An example is the "born" key. When looking up this key in the context file, we see this translates to  `http://schema.org/birthDate` with the value being of type `XSD:date`.

### Gotcha
Make sure all items are correctly spelled. If no mapping is found in the @context for a particular key, it will NOT show in the results (unless there is a top-level @id specified in the @context)

Example: If you make a typo and "born" is spelt "boorn" instead, it will not show in the output results as there only exists a mapping for "born" in the @context, and no default namespace (top-level @id) has been specified.

# Further reading
## used services
-  https://github.com/redpencilio/fragmentation-producer-service
## Linked data event streams
- https://semiceu.github.io/LinkedDataEventStreams/
  - The page itself contains many links to the standards it builds upon.
## models
  - https://data.vlaanderen.be/doc/applicatieprofiel/mandatendatabank/
    - Extension: EredienstMandatarissen (TODO; publish) https://app.diagrams.net/#G1F2AWVdZbTR57WrDd7nX0ZEFo3BJAq6xQ
  - https://data.vlaanderen.be/doc/applicatieprofiel/besluit-subsidie/
