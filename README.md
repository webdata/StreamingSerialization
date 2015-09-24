## Abstract Serialization
The definition above describes an **abstract** data model for a timestamped graph because it is based on an RDF Dataset, which is an abstract entity. The serialization of such timestamped graphs is an additional issue. A timestamped graph *could* be serialized in an existing format for RDF Datasets, e.g. [Trig](http://www.w3.org/TR/2014/REC-trig-20140225/). However, the Trig syntax does not provide abbreviations for the special cases that arise for a timestamped graph. Therefore, we must introduce a new concrete syntax for timestamped graphs. There are several syntactic categories.
####  'n' is a blank node. 
The intention is that this blank node will not be shared with any other RDF graphs or RDF Datasets. Therefore, it can be represented implicitly, meaning processors will generate a new blank node for each timestamped graph they receive. As an illustration, the serialization could take the form of a space-separated tuple surrounded by parentheses: (g p t). For example, a serialization of a timestamped graph containing two triples would appear as follows
```
(
  {
       _:b foaf:name "Alice" .
       _:b foaf:mbox <mailto:alice@work.example.org> .
  }
  <http://streamreasoning.org/timeVocab#startsAtTime>
  "2012-11-02T19:14:23+01:00"^^xsd:dateTime
)
```
When we say "a timestamped graph contains two triples", that is shorthand for "a timestamped graph whose named graph contains two triples"
This syntax could potentially be abbreviated further by making some assumptions, e.g.
* the predicate is always an absolute IRI
* the timestamp is by default of datatype xsd:dateTime
With these assumptions, most of the delimiting characters become unnecessary, and the timestamped graph could be serialized so:
```
(
  {
       _:b foaf:name "Alice" .
       _:b foaf:mbox <mailto:alice@work.example.org> .
  }
  http://streamreasoning.org/timeVocab#startsAtTime
  2012-11-02T19:14:23+01:00
)  
```
#### 'n' is an IRI and the named graph is included explicitly. 
As an illustration, the serialization could take the form of a space-separated qudruple surrounded by parentheses: (n g p t). For example:
```
(
 <http://example.org/alice>
  {
       _:b foaf:name "Alice" .
       _:b foaf:mbox <mailto:alice@work.example.org> .
  }
  <http://streamreasoning.org/timeVocab#startsAtTime>
  "2012-11-02T19:14:23+01:00"^^xsd:dateTime
)
```
This could potentially be abbreviated further by making some assumptions in addition to those from the previous case, e.g.
* the name of the named graph is always an absolute IRI
With these assumptions, most of the delimiting characters become unnecessary, and the timestamped graph could be serialized so:
```
(
http://example.org/alice
  {
       _:b foaf:name "Alice" .
       _:b foaf:mbox <mailto:alice@work.example.org> .
  }
  http://streamreasoning.org/timeVocab#startsAtTime
  2012-11-02T19:14:23+01:00
)  
```

####  'n' is an IRI and the named graph is obtained by dereferencing that IRI. 
As an illustration, the serialization could take the form of a space-separated tuple surrounded by parentheses: (n p t). For example:
```
(
 <http://example.org/alice>
 <http://streamreasoning.org/timeVocab#startsAtTime>
 "2012-11-02T19:14:23+01:00"^^xsd:dateTime
)
```
This could potentially be abbreviated further by making the assumptions as in the previous cases.
With these assumptions, most of the delimiting characters become unnecessary, and the timestamped graph could be serialized so:
```
(
http://example.org/alice
http://streamreasoning.org/timeVocab#startsAtTime
2012-11-02T19:14:23+01:00
)  
```
Although there may be further abbreviations possible for these special cases, it is also desirable to be able to extend this syntax to additional cases. For example, consider situations where
* the timestamp is not in the xsd:dateTime datatype. In this case the full form for a literal could be used, "2012-11-02"^^xsd:date
* the timestamp is complex, e.g. an interval. In this case, a [nested unlabelled blank node](http://www.w3.org/TR/2014/REC-turtle-20140225/#unlabeled-bnodes) could be used.
* the same named graph is referenced by more than one timestamping triple, with timestamps corresponding to different events. In this case a larger tuple, e.g. (g p1 t1 p2 t2) could be used.

Further abbreviations may be considered after a broader range of extensions has been examined.

Note that each of the concrete examples above describes an RDF Dataset - there is a uniform abstract data model for timestamped graphs, independent of the form taken in the concrete syntax.

## JSON-LD Serialization

This section describes a potential [JSON-LD](http://www.w3.org/TR/json-ld/) serialization for a timestamped graph. We will exemplify the syntax on the following graphs:

**<http://example.org/graphs/1>**: 
* Generated: 2015-01-15
* Recorded: 2015-01-16
```
<http://example.org/person/Jean-Paul> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://xmlns.com/foaf/0.1/Person> .
<http://example.org/person/Jean-Paul> <http://xmlns.com/foaf/0.1/knows> "http://example.org/person/Alasdair"  .
<http://example.org/person/Jean-Paul> <http://xmlns.com/foaf/0.1/knows> "http://example.org/person/Tara"  .
<http://example.org/person/Jean-Paul> <http://xmlns.com/foaf/0.1/name> "Jean Paul Calbimonte"  .

<http://http://example.org/person/Alisdair> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://xmlns.com/foaf/0.1/Person> .
<http://http://example.org/person/Alisdair> <http://xmlns.com/foaf/0.1/knows> "http://example.org/person/Tara" .
<http://http://example.org/person/Alisdair> <http://xmlns.com/foaf/0.1/name> "Alasdair Gray" .
```

**<http://example.org/graphs/2>**: 
* Generated: 2015-01-17
* Recorded: 2015-01-18
```
<http://example.org/person/Tara> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://xmlns.com/foaf/0.1/Person> .
<http://example.org/person/Tara> <http://xmlns.com/foaf/0.1/knows> "http://example.org/person/Alasdair"  ..
<http://example.org/person/Tara> <http://xmlns.com/foaf/0.1/name> "Tara Athan"  .
```

### Array of @graph keywords with shared context

All graphs are expressed in an array inside the default graph. The [context](http://www.w3.org/TR/json-ld/#the-context) defined in the default graph (shortcuts) can be reused by all graphs. 
```
{
  "@context": {
    "generatedAt": {
      "@id": "http://www.w3.org/ns/prov#generatedAtTime",
      "@type": "http://www.w3.org/2001/XMLSchema#date"
    },
    "recordedAt": {
      "@id": "http://www.w3.org/ns/prov#recordedAt",
      "@type": "http://www.w3.org/2001/XMLSchema#date"
    },
    "Person": "http://xmlns.com/foaf/0.1/Person",
    "name": "http://xmlns.com/foaf/0.1/name",
    "knows": "http://xmlns.com/foaf/0.1/knows"
  },
   "@graph":
  [
    {
    "@id": "http://example.org/graphs/1",
    "generatedAt": "2015-01-15",
    "recordedAt": "2015-01-16",
    "@graph":
    [
      {
        "@id": "http://example.org/person/Jean-Paul",
        "@type": "Person",
        "name": "Jean Paul Calbimonte",
        "knows": ["http://example.org/person/Alasdair","http://example.org/person/Tara"]
      },
      {
        "@id": "http://http://example.org/person/Alisdair",
        "@type": "Person",
        "name": "Alasdair Gray",
        "knows": "http://example.org/person/Tara"
      }
    ]
    },
    {
    "@id": "http://example.org/graphs/2",
    "generatedAt": "2015-01-17",
    "recordedAt": "2015-01-18",
    "@graph":
    [
      {
        "@id": "http://example.org/person/Tara",
        "@type": "Person",
        "name": "Tara Athan",
        "knows": "http://example.org/person/Alasdair"
      }
    ]
    }
  ]
 }
```
See in [JSON-LD playground](http://tinyurl.com/pmn8mcp)

#### Mixing shared and 'internal' contexts

Besides the shared context, each graph can define own its own context. Duplicate context terms are overridden using a most-recently-defined-wins mechanism. In the following example, we have defined an "example" prefix ( "example": "http://example.org/person/") just for the second graph.

```
{
  "@context": {
    "generatedAt": {
      "@id": "http://www.w3.org/ns/prov#generatedAtTime",
      "@type": "http://www.w3.org/2001/XMLSchema#date"
    },
    "recordedAt": {
      "@id": "http://www.w3.org/ns/prov#recordedAt",
      "@type": "http://www.w3.org/2001/XMLSchema#date"
    },
    "Person": "http://xmlns.com/foaf/0.1/Person",
    "name": "http://xmlns.com/foaf/0.1/name",
    "knows": "http://xmlns.com/foaf/0.1/knows"
  },
   "@graph":
  [
    {
    "@id": "http://example.org/graphs/1",
    "generatedAt": "2015-01-15",
    "recordedAt": "2015-01-16",
    "@graph":
    [
      {
        "@id": "http://example.org/person/Jean-Paul",
        "@type": "Person",
        "name": "Jean Paul Calbimonte",
        "knows": ["http://example.org/person/Alasdair","http://example.org/person/Tara"]
      },
      {
        "@id": "http://http://example.org/person/Alisdair",
        "@type": "Person",
        "name": "Alasdair Gray",
        "knows": "http://example.org/person/Tara"
      }
    ]
    },
    {
    "@id": "http://example.org/graphs/2",
    "generatedAt": "2015-01-17",
    "recordedAt": "2015-01-18",
    "@graph":
    [
      {
         "@context": {
           "example": "http://example.org/person/"
         },
        "@id": "example:Tara",
        "@type": "Person",
        "name": "Tara Athan",
        "knows": {
        	"@id": "example:Alisdair"
    	}
  	
      }
    ]
    }
  ]
 }
```
See in [JSON-LD playground](http://tinyurl.com/nwmvpu6)

#### Graph 'n' is a blank node

Graph name can be a blank node by ommiting the "@id" keyword in the graph definition:

```
{
  "@context": {
    "generatedAt": {
      "@id": "http://www.w3.org/ns/prov#generatedAtTime",
      "@type": "http://www.w3.org/2001/XMLSchema#date"
    },
    "recordedAt": {
      "@id": "http://www.w3.org/ns/prov#recordedAt",
      "@type": "http://www.w3.org/2001/XMLSchema#date"
    },
    "Person": "http://xmlns.com/foaf/0.1/Person",
    "name": "http://xmlns.com/foaf/0.1/name",
    "knows": "http://xmlns.com/foaf/0.1/knows"
  },
   "@graph":
  [
    {
    "generatedAt": "2015-01-15",
    "recordedAt": "2015-01-16",
    "@graph":
    [
      {
        "@id": "http://example.org/person/Jean-Paul",
        "@type": "Person",
        "name": "Jean Paul Calbimonte",
        "knows": ["http://example.org/person/Alasdair","http://example.org/person/Tara"]
      },
      {
        "@id": "http://http://example.org/person/Alisdair",
        "@type": "Person",
        "name": "Alasdair Gray",
        "knows": "http://example.org/person/Tara"
      }
    ]
    },
    {
    "generatedAt": "2015-01-17",
    "recordedAt": "2015-01-18",
    "@graph":
    [
      {
        "@id": "http://example.org/person/Tara",
        "@type": "Person",
        "name": "Tara Athan",
        "knows": "http://http://example.org/person/Alisdair"
      }
    ]
    }
  ]
 }
```
See in [JSON-LD playground](http://tinyurl.com/o2gndo3)

### Array of @graph keywords with NO shared context

All graphs are expressed in an array with no shared [context](http://www.w3.org/TR/json-ld/#the-context). 
```
[
  {
   "@context": {
    "generatedAt": {
      "@id": "http://www.w3.org/ns/prov#generatedAtTime",
      "@type": "http://www.w3.org/2001/XMLSchema#date"
    },
    "recordedAt": {
      "@id": "http://www.w3.org/ns/prov#recordedAt",
      "@type": "http://www.w3.org/2001/XMLSchema#date"
    },
    "Person": "http://xmlns.com/foaf/0.1/Person",
    "name": "http://xmlns.com/foaf/0.1/name",
    "knows": "http://xmlns.com/foaf/0.1/knows"
   },
    "@id": "http://example.org/graphs/1",
    "generatedAt": "2015-01-15",
    "recordedAt": "2015-01-16",
    "@graph":
    [
      {
        "@id": "http://example.org/person/Jean-Paul",
        "@type": "Person",
        "name": "Jean Paul Calbimonte",
        "knows": ["http://example.org/person/Alasdair","http://example.org/person/Tara"]
      },
      {
        "@id": "http://http://example.org/person/Alisdair",
        "@type": "Person",
        "name": "Alasdair Gray",
        "knows": "http://example.org/person/Tara"
      }
    ]
  },
   {
   "@context": {
    "generatedAt": {
      "@id": "http://www.w3.org/ns/prov#generatedAtTime",
      "@type": "http://www.w3.org/2001/XMLSchema#date"
    },
    "recordedAt": {
      "@id": "http://www.w3.org/ns/prov#recordedAt",
      "@type": "http://www.w3.org/2001/XMLSchema#date"
    },
    "Person": "http://xmlns.com/foaf/0.1/Person",
    "name": "http://xmlns.com/foaf/0.1/name",
    "knows": "http://xmlns.com/foaf/0.1/knows"
   },
     "@id": "http://example.org/graphs/2",
   "generatedAt": "2015-01-17",
    "recordedAt": "2015-01-18",
    "@graph":
    [
      {
        "@id": "http://example.org/person/Tara",
        "@type": "Person",
        "name": "Tara Athan",
        "knows": "http://http://example.org/person/Alisdair"
      }
    ]
  }
 ]
```
See in [JSON-LD playground](http://tinyurl.com/obea77x)
