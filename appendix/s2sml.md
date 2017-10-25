# S2SML

## Description

S2SML is a mapping language for mapping time-series data in TritanDB to graph model Linked Data. The creation of an S2SML mapping allows the execution of a SPARQL graph query on underlying time-series data. The language is compatible with R2RML \(can be translated to and from\) but provides the ability to

* Abstract metadata and provides constructs to collapse intermediate nodes
* Generate unique identifiers on-the-fly with a new concept called Faux nodes

## Structure

S2SML mappings are written in pure RDF and consist of a set of quad tuples of the form \(s,p,o,c\) where:

* s is the subject,
* p is the predicate,
* o is the object,
* and c is the context

Each subject, predicate, object and context in a tuple can be one of a few [types of nodes](#types).

* A subject might be an IRI, IRI Map, Blank Node or Faux Node
* A predicate is an IRI
* A object might be an IRI, IRI Map, Blank Node, Faux Node, Literal or Literal Map
* A context is an IRI

### Types of Nodes {#types}

#### IRI

An Internationalized Resource Identifier \(IRI\) node is a unicode string that:

* does not contain any control characters \( \#x00 - \#x1F, \#x7F-\#x9F\)
* and would produce a valid
  [URI](http://www.isi.edu/in-notes/rfc2396.txt)
  character sequence representing an absolute URI with optional fragment identifier when subjected to the encoding described below.

An example is:

```
<http://knoesis.wright.edu/ssw/ont/weather.owl#degrees>
```

#### Blank Node

The blank nodes in an RDF graph are drawn from an infinite set which are pairwise disjoint from the set of all RDF URI references and the set of all literals. In S2SML, blank nodes are assigned a[UUID](https://www.ietf.org/rfc/rfc4122.txt)as a bNodeId \(these are "practically unique"\). Given two blank nodes, it is possible to determine whether or not they are the same by comparing their bNodeId.

An example is:

```
_:bNodeId 
e.g. bNodeId = 61487f30-bf79-4d74-b52c-6ed96e74cb43
```

#### Literal

[Literals](https://www.w3.org/TR/2004/REC-rdf-concepts-20040210/#section-Literals) are used to identify values such as strings, numbers and dates by means of a lexical representation. A literal might be [plain](https://www.w3.org/TR/2004/REC-rdf-concepts-20040210/#dfn-plain-literal) or [typed](https://www.w3.org/TR/2004/REC-rdf-concepts-20040210/#dfn-typed-literal), with the typed literal containing an additional datatype IRI. An example is:

```
"-111.88222"ˆˆ
<
xsd:float
>
```

#### IRI Map

A IRI map node is a unicode string made up of multiple components, these components include IRI string parts and reference bindings to columns of a logical table. Reference bindings to columns of logical tables are specified by enclosing them in curly braces \(“{” and “}”\). The enclosed string, called a reference binding, should refer to the logical table name and column name separated by a dot character \("."\). The following syntax rules apply to valid string templates:

* Pairs of unescaped curly braces must enclose valid logical table names and column names separated by a dot.
* Curly braces that do not enclose column names must be escaped by a backslash character \(“\”\). This also applies to curly braces within column names.
* Backslash characters \(“\”\) must be escaped by preceding them with another backslash character, yielding “\”. This also applies to backslashes within column names.
* There should be at least one pair of unescaped curly braces e.g. the cardinality of the set of reference bindings should be more than or equals to 1.

An example is:

```
<http://knoesis.wright.edu/ssw/{sensors.sensorName}>
```

#### Literal Map

A Literal map node is a unicode string within a [typed](https://www.w3.org/TR/2004/REC-rdf-concepts-20040210/#dfn-typed-literal) containing a reference binding to a column of a logical table. The enclosed string, called a reference binding, should refer to the logical table name and column name separated by a dot character \("."\). The typed literal has a specific datatype IRI [http://iot.soton.ac.uk/s2s/s2sml\#literalMap](http://iot.soton.ac.uk/s2s/s2sml#literalMap).

An example is:

```
"readings.temperature"ˆˆ<s2s:literalMap>
```

#### Faux Node

A Faux node is similar to an IRI Map, however, instead of containing a reference binding to an existing column within a logical table, it contains a promise to create a uniquely identifiable column within the logical table if necessary. A Faux node is a unicode string made up of IRI string parts and a promise. Promises are specified by enclosing them in curly braces \(“{” and “}”\). The enclosed string, called a promise, should refer to the logical table name and the keyword "uuid" separated by a dot character \("."\).

An example is:

```
<http://knoesis.wright.edu/ssw/obs/{readings.uuid}>
```

## Example

This is a sample S2SML mapping:

```
_:1a <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://purl.oclc.org/NET/iot#WeatherObservation>.
    _:1a <http://purl.oclc.org/NET/ssnx/ssn#observationResult> _:1c.
    _:1a <http://purl.oclc.org/NET/ssnx/ssn#observationSamplingTime> _:1b.

    _:1a <http://purl.oclc.org/NET/ssnx/ssn#observedBy> <http://iot.soton.ac.uk/smarthome/sensor#environmental1>.
_:1b <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/2006/time#Instant>.
    _:1b <http://www.w3.org/2006/time#inXSDDateTime> "environment.TimestampUTC".
_:1c <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://purl.oclc.org/NET/iot#WeatherSensorOutput>.
    _:1c <http://purl.oclc.org/NET/ssnx/ssn#hasValue> _:1d.
    _:1c <http://purl.oclc.org/NET/ssnx/ssn#hasValue> _:1e.
    _:1c <http://purl.oclc.org/NET/ssnx/ssn#isProducedBy> <http://iot.soton.ac.uk/smarthome/sensor#environmental1>.
_:1d <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://purl.oclc.org/NET/iot#InternalTemperatureValue>.
    _:1d <http://purl.oclc.org/NET/iot#hasQuantityValue> "environment.insideTemp".
_:1e <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://purl.oclc.org/NET/iot#ExternalTemperatureValue>.
    _:1e <http://purl.oclc.org/NET/iot#hasQuantityValue> "environment.outsideTemp".
```

[![](https://cloud.githubusercontent.com/assets/9078335/14705403/35dd9ff2-07b1-11e6-9779-2e18483965ba.png)](https://cloud.githubusercontent.com/assets/9078335/14705403/35dd9ff2-07b1-11e6-9779-2e18483965ba.png)

## Data Ontology Modelling

S2SML is designed primarily to support modelling ontologies that represent data. An example is a sensor ontology which represents:

* Device Metadata \(e.g. sensor device location, sensor specifications\)
* Observation Metadata \(e.g. units of measure for the observation\)
* Observation Data \(e.g. value of measurement\)

#### Ephemeral VS Faux Nodes

In S2SML, identifier nodes can be modelled using either as ephemeral nodes, which are represented as blank nodes, or faux nodes, which are nodes that contain a promise to materialise identifier values if called upon as part of query projections.

