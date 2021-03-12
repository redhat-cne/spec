# Cloud Native Events (CNE)- Version 0.0.1

## Abstract

Cloud Native Events (CNE) is a Rest-API specification for defining the format of event
data.

## Table of Contents

- [Overview](#overview)
- [Notations and Terminology](#notations-and-terminology)
- [Context Attributes](#context-attributes)
- [Event Data](#event-data)
- [Size Limits](#size-limits)
- [Privacy & Security](#privacy-and-security)
- [Example](#example)

## Overview



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

#### Occurrence

An "occurrence" is the capture of a statement of fact during the operation of a
software system. This might occur because of a signal raised by the system or a
signal being observed by the system, because of a state change, because of a
timer elapsing, or any other noteworthy activity. For example, a device might go
into an alert state because the battery is low, or a virtual machine is about to
perform a scheduled reboot.

#### Event

An "event" is a data record expressing an occurrence and its context. Events are
routed from an event producer (the source) to interested event consumers. The
routing can be performed based on information contained in the event, but an
event will not identify a specific routing destination. Events will contain two
types of information: the [Event Data](#event-data) representing the Occurrence
and [Context](#context) metadata providing contextual information about the
Occurrence. A single occurrence MAY result in more than one event.

#### Producer

The "producer" is a specific instance, process or device that creates the data
structure describing the Cloud Native Events.

#### Source

The "source" is the context in which the occurrence happened. In a distributed
system it might consist of multiple [Producers](#producer). If a source is not
aware of Cloud Native Events, an external producer creates the Cloud Native Events on behalf of
the source.

#### Consumer

A "consumer" receives the event and acts upon it. It uses the context and data
to execute some logic, which might lead to the occurrence of new events.
In Cloud Native Events consumer is a sidecar.

#### intermediary/sidecar

An "intermediary" or a "sidecar" receives a message containing an event for the purpose of
forwarding it to the next receiver, which might be another intermediary/sidecar or a
[Consumer](#consumer). A typical task for an intermediary is to route the event
to receivers based on the information in the [Context](#context).

#### Context

Context metadata will be encapsulated in the
[Context Attributes](#context-attributes). Tools and application code can use
this information to identify the relationship of Events to aspects of the system
or to other Events.

#### Data

Domain-specific information about the occurrence (i.e. the payload). This might
include information about the occurrence, details about the data that was
changed, or more. See the [Event Data](#event-data) section for more
information.

#### Event Format

An Event Format specifies how to serialize a Cloud Native Events as a sequence of bytes.
Stand-alone event formats, such as the [JSON format](json-format.md), specify
serialization independent of any protocol or storage medium. Protocol Bindings
MAY define formats that are dependent on the protocol.

#### Message

Events are transported from a source to a destination via messages.

A "structured-mode message" is one where the event is fully encoded using
a stand-alone event format and stored in the message body.

A "binary-mode message" is one where the event data is stored in the message
body, and event attributes are stored as part of message meta-data.

#### Protocol

Messages can be delivered HTTP


## Context Attributes

Every Cloud Native Events conforming to this specification MUST include context
attributes designated as REQUIRED, MAY include one or more OPTIONAL context
attributes and MAY include one or more extension attributes.

These attributes, while descriptive of the event, are designed such that they
can be serialized independent of the event data. This allows for them to be
inspected at the destination without having to deserialize the event data.

### Attribute Naming Convention

The Cloud Native Events specifications define mappings to various protocols and
encodings, and the accompanying Cloud Native Events SDK targets  only golang  for now.

Cloud Native Events attribute names MUST consist of lower-case letters ('a' to 'z') or
digits ('0' to '9') from the ASCII character set. Attribute names SHOULD be
descriptive and terse and SHOULD NOT exceed 20 characters in length.

### Type System

The following abstract data types are available for use in attributes. Each of
these types MAY be represented differently by different event formats and in
protocol metadata fields. This specification defines a canonical
string-encoding for each type that MUST be supported by all implementations.

- `Boolean` - a boolean value of "true" or "false".
  - String encoding: a case-sensitive value of `true` or `false`.
- `Integer` - A whole number in the range -2,147,483,648 to +2,147,483,647
  inclusive. This is the range of a signed, 32-bit, twos-complement encoding.
  Event formats do not have to use this encoding, but they MUST only use
  `Integer` values in this range.
  - String encoding: Integer portion of the JSON Number per
    [RFC 7159, Section 6](https://tools.ietf.org/html/rfc7159#section-6)
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
- `Binary` - Sequence of bytes.
  - String encoding: Base64 encoding per
    [RFC4648](https://tools.ietf.org/html/rfc4648).
- `EndpointUri` - Absolute uniform resource identifier.
  - String encoding: `Absolute URI` as defined in
    [RFC 3986 Section 4.3](https://tools.ietf.org/html/rfc3986#section-4.3).
- `UriLocation` - Absolute uniform resource identifier.
  - String encoding: `Absolute URI` as defined in
    [RFC 3986 Section 4.3](https://tools.ietf.org/html/rfc3986#section-4.3).
- `Timestamp` - Date and time expression using the Gregorian Calendar.
  - String encoding: [RFC 3339](https://tools.ietf.org/html/rfc3339).


All context attribute values MUST be of one of the types listed above.
Attribute values MAY be presented as native types or canonical strings.

A strongly-typed programming model that represents a Cloud Native Events or any extension
MUST be able to convert from and to the canonical string-encoding to the
runtime/language native type that best corresponds to the abstract type.

For example, the `time` attribute might be represented by the language's native
_datetime_ type in a given implementation, but it MUST be settable providing an
RFC3339 string, and it MUST be convertible to an RFC3339 string when mapped to a
header of an HTTP message.


An attribute value of type `Timestamp` might indeed be routed as a string
through multiple hops and only materialize as a native runtime/language type at
the producer and ultimate consumer. The `Timestamp` might also be routed as a
native protocol type and might be mapped to/from the respective
language/runtime types at the producer and consumer ends, and never materialize
as a string.

The choice of serialization mechanism will determine how the context attributes
and the event data will be serialized. For example, in the case of a JSON
serialization, the context attributes and the event data might both appear
within the same JSON object.

### REQUIRED Attributes

The following attributes are REQUIRED to be present in all Cloud Native Events:

#### id

- Type: `String`
- Description: Identifies the event. Producers MUST ensure that `source` + `id`
  is unique for each distinct event. If a duplicate event is re-sent (e.g. due
  to a network error) it MAY have the same `id`. Consumers MAY assume that
  Events with identical `source` and `id` are duplicates.
- Constraints:
  - REQUIRED
  - MUST be a non-empty string
  - MUST be unique within the scope of the producer
- Examples:
  - An event counter maintained by the producer
  - A UUID

#### resourceaddress
The format of the resource address is shown in Table 1.1.3.2-2.  The resource address specifies the event producer with a hierarchical path.  The path format provides the ability for management and monitoring to extend beyond a single cluster and node.  

/{clusterName}/{siteName}/{nodeName}/{resource}
Field definitions are shown in Table 1.1.32-2.

Address Component |  Description |  Example |
--- | --- | --- |
clusterName | The name of the cloud where the producer exists.  A ‘.’ can be used to indicate the cluster where the consumer exists. | eastern-edge  | 
siteName | The name of the site containing the node(s) that the event producer is located in. A regular expression with * or . may be specified to subscribe to multiple sites.  | cellsite16385  |
nodeName  | Name of the Worker node or Compute node where the producer exists.  The name must map to the nomenclature in use for the underlying cloud infrastructure.  A regular expression with * or . may be specified to subscribe to multiple nodes. | node27 or node* -> all nodes | 
resource | The hierarchical name for the resource.  For this specification, the resource hierarchy will align with the yang models specified in O-RAN.WG4.MP-Yangs-v05.00 to the extent possible.  A * may be specified in the address hierarchy to subscribe to events below the specified level.| ../o-ran-sync/sync-group/sync-status/sync-state , ../o-ran-sync/sync-group/*| 


#### specversion

- Type: `String`
- Description: The version of the Cloud Native Events specification which the event
  uses. This enables the interpretation of the context. Compliant event
  producers MUST use a value of `1.0` when referring to this version of the
  specification.

  Currently, this attribute will only have the 'major' and 'minor' version
  numbers included in it. This allows for 'patch' changes to the specification
  to be made without changing this property's value in the serialization.
  Note: for 'release candidate' releases a suffix might be used for testing
  purposes.

- Constraints:
  - REQUIRED
  - MUST be a non-empty string

#### type

- Type: `String`
- Description: This attribute contains a value describing the type of event
  related to the originating occurrence. Often this attribute is used for
  routing, observability, policy enforcement, etc.

- Constraints:
  - REQUIRED
  - MUST be a non-empty string
  - SHOULD be prefixed with a reverse-DNS name. The prefixed domain dictates the
    organization which defines the semantics of this event type.
- Examples
  - cevent.synchronization-state-change


#### time

- Type: `Timestamp`
- Description: Timestamp of when the occurrence happened. If the time of the
  occurrence cannot be determined then this attribute MAY be set to some other
  time (such as the current time) by the Cloud Native Events producer, however all
  producers for the same `source` MUST be consistent in this respect. In other
  words, either they all use the actual time of the occurrence or they all use
  the same algorithm to determine the value used.
- Constraints:
  - OPTIONAL
  - If present, MUST adhere to the format specified in
    [RFC 3339](https://tools.ietf.org/html/rfc3339)


## Event Data

As defined by the term [Data](#data), Cloud Native Events data MAY include domain-specific
information about the occurrence based on enumeramtion type. When present, this information will be
encapsulated within `data.value_type`.

# Size Limits

Cloud Native Events will be forwarded through one or more generic
intermediaries, each of which might impose limits on the size of forwarded
events. The Cloud Native Events is wrapped within the CloudEvents json objkect (See CloudEvents Spec here)

The "size" of an event is its wire-size and includes every bit that is
transmitted on the wire for the event: protocol frame-metadata, event metadata,
and event data, based on the chosen event format and the chosen protocol
binding.

If an application configuration requires for events to be routed across
different protocols or for events to be re-encoded, the least efficient
protocol and encoding used by the application SHOULD be considered for
compliance with these size constraints:

- Intermediaries MUST?MAY wrap Cloud Native Events wwithin CloudEvents object and forwarded data 
   size shouldbe of 64 KByte or less.
- Consumers SHOULD accept events of a size of at least 64 KByte.

In effect, these rules will allow producers to publish events up to 64KB in size
safely. Safely here means that it is generally reasonable to expect the event to
be accepted and retransmitted by all intermediaries. It is in any particular
consumer's control, whether it wants to accept or reject events of that size due
to local considerations.

Generally, Cloud Native Events publishers SHOULD keep events compact by avoiding
embedding large data items into event payloads and rather use the event payload
to link to such data items. From an access control perspective, this approach
also allows for a broader distribution of events, because accessing
event-related details through resolving links allows for differentiated access
control and selective disclosure, rather than having sensitive details embedded
in the event directly.

# Privacy and Security

# Example

The following example shows a O-RAN request to create subscription/publisher serialized as JSON:

```JSON
{
    "specversion" : "1.0",
    "subscriptionid": "",
    "resource" : "/sync/sync-status/sync-state",
    "endpointuri": "http://localhost:8080/event/alert",
    "urilocation": "",
    "data" : {}
}
The following example shows a O-RAN response to create subscription/publisher serialized as JSON:

```JSON
{
    "specversion" : "1.0",
    "subscriptionid": "789be75d-7ac3-472e-bbbc-6d62878aad4a",
    "resource" : "/sync/sync-status/sync-state",
    "endpointuri": "http://localhost:8080/event/alert",
    "urilocation": "http://localhost:9090/api/oranevenet/subscription/789be75d-7ac3-472e-bbbc-6d62878aad4a",
    "data" : {}
}

```
The following example shows a Cloud Native Events serialized as JSON:
```JSON
{
    "type" : "event.synchronization-state-change",
    "time" : "2021-02-05T17:31:00Z",
    "data" : { 
    "version" : "1.0", 
    "values" : [
          { 
            "type": "notification",
            "resource" : "/sync/sync-status/sync-state",
            "value_type" : "enumeration",
            "value" : "HOLDOVER"
          }
    ]
  }
}
```

# Example

The following example shows a Cloud Native Events embeded in CloudEvents and serialized as JSON:

```JSON
{

    "type" : "event.synchronization-state-change",
    "source" : "https://localhost:9090/subscription/33-3123-213-31-3",
    "subject" : "123",
    "id" : "A234-1234-1234",
    "time" : "2021-02-05T17:31:00Z",
    "datacontenttype" : "text/xml",
    "data" : {
    "specversion" : "1.0",
    "type" : "event.synchronization-state-change",
    "time" : "2021-02-05T17:31:00Z",
    "datacontenttype" : "text/xml",
    "data" : { 
     "version" : "1.0", 
     "values" : [
          { 
            "type": "notification",
            "resource" : "/sync/sync-status/sync-state",
            "value_type" : "enumeration",
            "value" : "HOLDOVER"
          }
    ]
  }
}

```