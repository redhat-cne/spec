# Cloud Native Events (CNE) Publish/Subscribe - Version 0.0.1

## Abstract

Cloud Native Events PubSub(CNE-PubSub)  is a Rest-API specification for defining the format of publisher and subscription data.

## Table of Contents

- [Overview](#overview)
- [Notations and Terminology](#notations-and-terminology)
- [Example](#example)

## Overview
Publish/subscribe messaging, or pub/sub messaging, is a form of asynchronous service-to-service communication used in 
serverless and microservices architectures. In a pub/sub model, any message published to a topic is immediately 
received by all of the subscribers to the topic.

## Notations and Terminology

### Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

For clarity, when a feature is marked as "OPTIONAL" this means that it is
OPTIONAL for both the [Producer](#producer) and [Consumer](#consumer) of a
message to support that feature. In other words, a producer can choose to
include that feature in a message if it wants, and a consumer can choose to
support that feature if it wants. A consumer that does not support that feature
will then silently ignore that part of the message. The producer needs to be
prepared for the situation where a consumer ignores that feature. An
[Intermediary](#intermediary) SHOULD forward OPTIONAL attributes.

### Terminology

This specification defines the following terms:


#### Publish

A "publish" is a data record created by the [Producer](#producer) expressing an event type that will be published and its resource. 
Publish will contain two types of information: the [Endpoint URI](#EndpointUri) representing 
the return url for rest api to perform action and [Resource](#resource) metadata providing 
resource information about the evnt to be published. 

#### Subscribe

An "subscribe" is a data record created by [Consumer](#consumer) expressing an intrest to subscribe to an occurance of the 
event  for a given resource type. `Subscribe` will contain two types of information: 
the [Endpoint URI](#event-data) representing  the [Endpoint URI](#EndpointUri) for rest api to performa action
and [Resource](#resource) metadata providing resource information about the event that is subscribed to. 

#### Producer

The "producer" is a specific instance, process or device that creates the data
structure describing the event that it would like to publish .

Producer publishes informatuion about the event data that will be produced to a  [Intermediary/Sidecar](#intermediary/sidecar), creates a publication and returns 
the status to the [Endpoint URI](#EndpointUri).

#### Consumer

A "consumer" creates a subscription to the event it is intrested to consume via [Intermediary/Sidecar](#intermediary/sidecar). It uses the  resource  to request to subscribe to a specific event data and speificy the [Endpoint URI](#EndpointUri) to process on recieve of the event data.


#### Intermediary/Sidecar

An "intermediary" or a "sidecar" receives a message containing an pub sub request for the purpose of
creatting the publisher or subscription details.


### Type System

The following abstract data types are available for use in attributes. Each of
these types MAY be represented differently by different event formats and in
protocol metadata fields. This specification defines a canonical
string-encoding for each type that MUST be supported by all implementations.
- `String` - Sequence of allowable Unicode characters. The following characters
  are disallowed:
  - the "control characters" in the ranges U+0000-U+001F and U+007F-U+009F (both
    ranges inclusive), since most have no agreed-on meaning, and some, such as
    U+000A (newline), are not usable in contexts such as HTTP headers.
  - code points
    [identified as noncharacters by Unicode](http://www.unicode.org/faq/private_use.html#noncharacters).
  - code points identifying Surrogates, U+D800-U+DBFF and U+DC00-U+DFFF, both
    ranges inclusive, unless used properly in pairs. Thus (in JSON notation)
    "\uDEAD" is invalid because it is an unpaired surrogate, while
    "\uD800\uDEAD" would be legal.
    
- `EndpointUri` - Absolute uniform resource identifier.
  - String encoding: `Absolute URI` as defined in
    [RFC 3986 Section 4.3](https://tools.ietf.org/html/rfc3986#section-4.3).
- `UriLocation` - Absolute uniform resource identifier.
  - String encoding: `Absolute URI` as defined in
    [RFC 3986 Section 4.3](https://tools.ietf.org/html/rfc3986#section-4.3).


A strongly-typed programming model that represents a Cloud Native Events or any extension
MUST be able to convert from and to the canonical string-encoding to the
runtime/language native type that best corresponds to the abstract type.

### UriLocation
The API client can ignore it in the POST body when creating a subscription/publisher resource 
(this will be sent to the client after the resource eitehr publisher or subscription is created).


### REQUIRED Attributes

The following attributes are REQUIRED to be present in all Cloud Native Events:

### EndpointUri

Endpoint URI (a.k.a callback URI), e.g. http://localhost:8080/resourcestatus/ptp
Please note that ‘localhost’ is mandatory and cannot be replaced by an IP or FQDN.


#### Resource
Yang path to value specification.
The format of the resource address is shown in Table 1.1.3.2-2.  The resource address specifies the event producer with a hierarchical path.  The path format provides the ability for management and monitoring to extend beyond a single cluster and node.  

/{clusterName}/{siteName}/{nodeName}/{resource}
Field definitions are shown in Table 1.1.32-2.

Address Component |  Description |  Example |
--- | --- | --- |
clusterName | The name of the cloud where the producer exists.  A ‘.’ can be used to indicate the cluster where the consumer exists. | eastern-edge  | 
siteName | The name of the site containing the node(s) that the event producer is located in. A regular expression with * or . may be specified to subscribe to multiple sites.  | cellsite16385  |
nodeName  | Name of the Worker node or Compute node where the producer exists.  The name must map to the nomenclature in use for the underlying cloud infrastructure.  A regular expression with * or . may be specified to subscribe to multiple nodes. | node27 or node* -> all nodes | 
resource | The hierarchical name for the resource.  For this specification, the resource hierarchy will align with the yang models specified in O-RAN.WG4.MP-Yangs-v05.00 to the extent possible.  A * may be specified in the address hierarchy to subscribe to events below the specified level.| ../o-ran-sync/sync-group/sync-status/sync-state , ../o-ran-sync/sync-group/*| 


#### SpecVersion

- `specversion`
- Type: `String`
- Description: The version of the Cloud Native Events specification which the pubsub
  uses. Compliant event producers MUST use a value of `1.0` when referring to this version of the
  specification.

- Constraints:
  - REQUIRED
  - MUST be a non-empty string


# Example

The following example shows a request to create subscription/publisher serialized as JSON:

```JSON
{
    "endpointuri": "http://example.com/endpointUri",
    "resourceaddress": "/cluster/node/ptp", 
}
The following example shows a response to create subscription/publisher serialized as JSON:

```JSON
{
    "specversion" : "1.0",
    "id": "5ce55d17-9234-4fee-a589-d0f10cb32b8e",
    "endpointuri": "http://example.com/endpointUri",
    "resourceaddress": "/cluster/node/ptp", 
    "urilocation": "http://localhost:9090/api/oranevenet/subscription/789be75d-7ac3-472e-bbbc-6d62878aad4a",
}
