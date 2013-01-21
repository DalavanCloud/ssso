% Simple Service Status Ontology (SSSO)
% Jakob Voß
% GIT_REVISION_DATE

# Introduction

The **Simple Service Status Ontology (SSSO)** is an event-based RDF ontology
for typical status in fulfillment of a service.

## Status of this document

This HTML document and the RDF serializations of the Simple Service Status
Ontology ([`ssso.ttl`](ssso.ttl) in RDF/Turtle and [`ssss.owl`](ssso.owl) in
RDF/XML) are generated automatically from a source file written in Pandoc
Markdown syntax. Sources and updates are available at
<http://github.com/gbv/ssso>. The current version of this document was last
modified at GIT_REVISION_DATE with revision GIT_REVISION_HASH.


## Namespaces

The URI namespace of this ontology is <http://purl.org/ontology/ssso#>. The
namespace prefix `ssso` is recommeded. The URI of this ontology as a whole
is <http://purl.org/ontology/ssso>.

    @prefix ssso: <http://purl.org/ontology/ssso#> .
    @base         <http://purl.org/ontology/ssso> .

The following namspace prefixes are used to refer to other ontologies:

    @prefix crm:    <http://purl.org/NET/cidoc-crm/core#> .
    @prefix dctype: <http://purl.org/dc/dcmitype/> .
    @prefix dul:    <http://www.loa-cnr.it/ontologies/DUL.owl#> .
    @prefix event:  <http://purl.org/ontology/c4dm/event.owl#> .
    @prefix foaf:   <http://xmlns.com/foaf/0.1/> .
    @prefix lode:   <http://linkedevents.org/ontology/> .
    @prefix ncal:   <http://www.semanticdesktop.org/ontologies/2007/04/02/ncal#> .
    @prefix owl:    <http://www.w3.org/2002/07/owl#> .
    @prefix prov:   <http://www.w3.org/ns/prov#> .
    @prefix rdfs:   <http://www.w3.org/2000/01/rdf-schema#> .
    @prefix schema: <http://schema.org/> .
    @prefix vann:   <http://purl.org/vocab/vann/> .

## Ontology

    <> a owl:Ontology ;
        rdfs:label "Simple Service Status Ontology" ;
        rdfs:label "SSSO" ;
        vann:preferredNamespacePrefix "ssso" .

# Overview

A service fulfillment is modeled as chain of service events, each being an
instance of [Service](#service). Services can be chained in time with the
properties [nextService](#nextservice) and [previousService](#previousservice).

Five typical service status are defined as disjoint subclasses:

* A [ReservedService](#reservedservice) is in status *reserved*:\
  the service has been accepted for fulfillment but no action has taken place.
* A [PreparedService](#preparedservice) is in status *prepared*:\
  the fulfillment is being prepared but is has not actually started.
* A [ProvidedService](#providedservice) is in status *provided*:\
  the service is ready to be fulfilled on request.
* A [ExecutedService](#executedservice) is in status *executed*:\
  the service is actually being fulfilled.
* A [RejectedService](#rejectedservice) is in status *rejected*:\
  the service has been refused or stopped.

Actual services do not need to implement all of these service status. A service
is typically connected to at least one service [ServiceProvider](#provider) and
at least one service [ServiceConsumer](#serviceconsumer). 

The following diagram illustrates the classes and properties definied in this ontology:

```
                                   nextService / previousService
                                             -----
                                            |     |
                                            v     v
   +-----------------+   provided     +-----------------+   consumedBy   +-----------------+
   | ServiceProvider |--------------->|     Service     |--------------->| ServiceConsumer |
   |                 |<---------------|                 |<---------------|                 |
   +-----------------+   providedBy   | ReservedService |   consumed     +-----------------+
                                      | PreparedService |
                                      | ProvidedService |
                                      | ExecutedService |
                                      | RejectedService |
                                      +-----------------+
```

This ontology does not make any assumptions about types of services. Examples
include buying a product from a shop and lending a book from a library. The
class [TimeTravel](#timetravel) is included as artifical example of a service
type.

# Classes

## Service

A service is an Event or activity. Several RDF ontologies have been proposed 
for such entities, maybe because ontology engineers cannot agree on semantics:

* Dublin Core Metadata Terms
* Event ontology
* Provenance ontology
* LODE ontology (Linking Open Descriptions of Events)
* DOLCE+DnS Ultralite ontology
* CIDOC-CRM expressed in OWL
* Schema.org Ontology
* NEPOMUK Calendar Ontology

An SSSO Service is subclass of all of them to make happy multiple communities.

    ssso:Service a owl:Class ;
        rdfs:label "Service" ;
        rdfs:subClassOf 
            dctype:Event , 
            event:Event , 
            prov:Activity , 
            lode:Event , 
            dul:Event ,
            crm:E7_Activity ,
            ncal:Event ,
            schema:Event ;
            rdfs:isDefinedBy <> .

Service [providers](#serviceprovider) can be connected to a service with
property [providedBy](#providedby) and service [consumers](#serviceconsumer)
can be connected to a service with property [consumedBy](#consumedBy).

The time when a service started and/or ended can be expressed as instance of
`xsd:dateTime` or `xsd:date` with properties `prov:startedAtTime` or
`schema:startDate` and `prov:endedAtTime` or `schema:endDate` respectively. The
starting time must be equal to or earlier than the ending time (unless the
service is an instance of [TimeTravel](#timetravel) and
[ExecutedService](#executedservice)).


## ReservedService

A reserved service is a [Service](#service) that has been accepted by a service
provider for execution but not prepared yet. The reserved service has neither
been prepared by a service provider but only queued for further processing.
A typical example is a product order that has been placed but not payed yet or
a payed ticket to a theater performance.

    ssso:ReservedService a owl:Class ;
        rdfs:label "ReservedService" ;
        rdfs:subClassOf ssso:Service ;
        owl:disjointWith 
          ssso:PreparedService, ssso:ProvidedService, 
          ssso:ExecutedService, ssso:RejectedService ;
        rdfs:isDefinedBy <> .

## PreparedService

A prepared service is being prepared to be provided or executed. A typical example 
is a product that is being send to its consumer.

    ssso:PreparedService a owl:Class ;
        rdfs:label "ReservedService" ;
        rdfs:subClassOf ssso:Service ;
        owl:disjointWith
          ssso:ReservedService, ssso:ProvidedService, 
          ssso:ExecutedService, ssso:RejectedService ;
        rdfs:isDefinedBy <> .

## ProvidedService

A provided service is being made available for immediate execution. A typical example is a
product that is ready for being picked up by its consumer.

    ssso:ReservedService a owl:Class ;
        rdfs:label "ReservedService" ;
        rdfs:subClassOf ssso:Service ;
        owl:disjointWith
          ssso:ReservedService, ssso:PreparedService, 
          ssso:ExecutedService, ssso:RejectedService ;
        rdfs:isDefinedBy <> .

## ExecutedService

An executed service represents the actual execution event of fulfillment of a
service. A typical example is a theater performance that is being played.

    ssso:ExecutedService a owl:Class ;
        rdfs:label "ExecutedService" ;
        rdfs:subClassOf ssso:Service ;
        owl:disjointWith
          ssso:ReservedService, ssso:PreparedService, 
          ssso:ProvidedService, ssso:RejectedService ;
        rdfs:isDefinedBy <> .

## RejectedService

A rejected service has been rejected by its provider or by its consumer. The
rejection may be infinite or it may be followed by another service when the
reason for rejection has been removed.

    ssso:RejectedService a owl:Class ;
        rdfs:label "RejectedService" ;
        rdfs:subClassOf ssso:Service ;
        owl:disjointWith
          ssso:ReservedService, ssso:PreparedService, 
          ssso:ProvidedService, ssso:ExecutedService ;
        rdfs:isDefinedBy <> .

## ServiceProvider

A service provider is an entity that is responsible for providing a
[Service](#service).  Typical providers are instances of `foaf:Agent`, such as
organizations or people, but the Simple Service Status Ontology does not put
any constraints on the nature of providers.

    ssso:ServiceProvider a owl:Class ;
        rdfs:label "ServiceProvider" ;
        rdfs:isDefinedBy <> .

## ServiceConsumer

A service consumer is an entity that is requesting or consuming a
[Service](#service).  Typical consumers are instances of `foaf:Agent`, such as
organizations or people, but the Simple Service Status Ontology does not put
any constraints on the nature of consumers.

    ssso:ServiceConsumer a owl:Class ;
        rdfs:label "ServiceConsumer" ;
        rdfs:isDefinedBy <> .

## TimeTravel

An event which ends before it has been started. Details have been implemented
in the future.

    ssso:TimeTravel a owl:Class ;
        rdfs:label "TimeTravel" ;
        rdfs:isDefinedBy <> .

# Properties

## provided

Relates a [ServiceProvider](#serviceprovider) instance to a
[Service](#service) instance .

    ssso:providedBy a owl:ObjectProperty ;
        rdfs:label "provided" ;
        rdfs:domain ssso:ServiceProvider ;
        rdfs:range ssso:Service ;
        owl:inverseOf ssso:providedBy ;
        rdfs:isDefinedBy <> .

## providedBy

Relates a [Service](#service) instance to a [ServiceProvider](#serviceprovider)
instance.

    ssso:providedBy a owl:ObjectProperty ;
        rdfs:label "providedBy" ;
        rdfs:domain ssso:Service ;
        rdfs:range ssso:ServiceProvider ;
        owl:inverseOf ssso:provided ;
        rdfs:isDefinedBy <> .

## consumed

Relates a [ServiceConsumer](#serviceconsumer) instance to a [Service](#service)
instance.

    ssso:consumedBy a owl:ObjectProperty ;
        rdfs:label "consumed" ;
        rdfs:domain ssso:ServiceConsumer ;
        rdfs:range ssso:Service ;
        owl:inverseOf ssso:consumedBy ;
        rdfs:isDefinedBy <> .

## consumedBy

Relates a [Service](#service) instance to a [ServiceConsumer](#serviceconsumer)
instance.

    ssso:consumedBy a owl:ObjectProperty ;
        rdfs:label "consumedBy" ;
        rdfs:domain ssso:Service ;
        rdfs:range ssso:ServiceConsumer ;
        owl:inverseOf ssso:consumed ;
        rdfs:isDefinedBy <> .

## nextService

Relates a service instances to another service instance which is following in time.
The starting time of the following service instance MUST be equal or later then the
ending time of the previous service (unless one of the services is an instance of 
[ExecutedService](#executedservice) and [TimeTravel](#timetravel)).

    ssso:nextService a owl:ObjectProperty ;
        rdfs:label "nextService" ;
        rdfs:domain ssso:Service ;
        rdfs:range  ssso:Service ;
        owl:inverseOf ssso:previousService ;
        rdfs:isDefinedBy <> .

## previousService

Relates a service instances to another service instance which is preceding in time.
The ending time of the previousg service instance MUST be equal or earlier then the
starting time of the next service (unless one of the services is an instance of 
[ExecutedService](#executedservice) and [TimeTravel](#timetravel)).

    ssso:previous a owl:ObjectProperty ;
        rdfs:label "previousService" ;
        rdfs:domain ssso:Service ;
        rdfs:range  ssso:Service ;
        owl:inverseOf ssso:nextService ;
        rdfs:isDefinedBy <> .

# References

<!-- ## Normative Reference

...-->

## Informative References

SSSO is loosely connected to the following ontologies: it is compatible with
them but their use is optional. Feel free to rely on or ignore additional parts
of these ontologies when using SSSO.

* *Dublin Core Metadata Terms*. 
  <http://dublincore.org/documents/dcmi-terms/> .
* *Schema.org Ontology*. 
  <http://schema.org/> .
* *Event Ontology*. 
  <http://motools.sourceforge.net/event/event.html> .
* *Provenance Ontology*. 
  <http://www.w3.org/TR/prov-o/> .
* *Linking Open Descriptions of Events*.
  <http://linkedevents.org/ontology/> .
* Antoni Mylka (Editor). 2007. *NEPOMUK Calendar Ontology*.
  <http://www.semanticdesktop.org/ontologies/ncal/> .
* *DOLCE+DnS Ultralite (DUL) ontology*. 
  *<http://ontologydesignpatterns.org/wiki/Ontology:DOLCE+DnS_Ultralite>*.
* *CIDOC CRM in OWL 2*.
  <http://bloody-byte.net/rdf/cidoc-crm/>.

SSSO was motivated by the design of an ontology for the Patrons Account
Information API (PAIA): <http://purl.org/ontology/paia>
