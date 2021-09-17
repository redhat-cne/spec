# Cloud Native Events (CNE)- Version 0.1.0

## Abstract

Cloud Native Events (CNE) is a Rest-API specification for defining the format of event
data.

## Table of Contents

- [Overview](#overview)
- [Notations and Terminology](#notations-and-terminology)
- [Type System](#type-system)
- [REQUIRED Attributes](#required-attributes)
- [Event Data](#event-data)
- [Size Limits](#size-limits)
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
event will not identify a specific routing destination. Events will contain 
the [Event Data](#event-data) representing the Occurrence
and providing contextual information about the Occurrence. 
A single occurrence MAY result in more than one event.

#### Producer

The "producer" is a specific instance, process or device that creates the data
structure describing the Cloud Native Events.

#### Source

The "source" is the context in which the occurrence happened. In a distributed
system it might consist of multiple [Producers](#producer). If a source is not
aware of Cloud Native Events, an external producer creates the Cloud Native Events on behalf of
the source.


#### Context
Context metadata will be encapsulated in the event as resource address. 

#### Consumer

A "consumer" receives the event and acts upon it. It uses the context and data
to execute some logic, which might lead to the occurrence of new events.
In Cloud Native Events consumer is a sidecar.

#### intermediary/sidecar

An "intermediary" or a "sidecar" receives a message containing an event for the purpose of
forwarding it to the next receiver, which might be another intermediary/sidecar or a
[Consumer](#consumer). A typical task for an intermediary is to route the event
to receivers based on the information in the [Context](#context).


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

## Type System

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
- `Timestamp` - Date and time expression using the Gregorian Calendar.
  - String encoding: [RFC 3339](https://tools.ietf.org/html/rfc3339).



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

## REQUIRED Attributes

The following attributes are REQUIRED to be present in all Cloud Native Events:

### id

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

### type

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

### dataContentType

- Type: `String`
- Description: This attribute contains a value describing the content type
  of the event data.

- Constraints:
  - REQUIRED
  - MUST be a non-empty string
- Examples
  - application/json


### time

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


### data

This is defined in the next section.

## Event Data

Cloud Native Events data includes a version number and a generic `Values` field or a Redfish hardware specific `Data` field.
As defined by the term Data, Cloud Native Events data MAY include domain-specific
information about the occurrence based on enumeramtion type. When present, this information will be
encapsulated within `data.value_type`.

### version

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

### Values
Values represent array of event types that occured for a given occurance of event 
[{data_type:"[data_type](#data_type)"}, "resource":[resource](#resouorce)],"[value_type](#value_type)","[value](#value)}]

“data_type”: “notification”
              “resource” : “/sync/sync-status/sync-state”,
               “value_type” : “enumeration”,
               "value" : ”HOLDOVER"


#### data_type
Defines tthe event types . This can be of two types.
-`notification`
  Notificatuion types defines the event data is of type notification of event occured.
- `metric`
   metric are time-series data in decimal format.

#### resource
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


#### value_type
- `value_type`
    The type format of the [`value`](#value) property.


#### value
- `value`
  String representation of value in value_type format

### Data
Data represents a Redfish Event as defined in Redfish schema
 [Event.v1_4_1.json](https://redfish.dmtf.org/schemas/v1/Event.v1_4_1.json).
The Event schema describes the JSON payload received by an Event Destination,
which has subscribed to event notification, when events occur.  This Resource
contains data about events, including descriptions, severity, and a MessageId
link to a Message Registry that can be accessed for further information.

#### REQUIRED Attributes

The following attributes are REQUIRED to be present in the `Data` property.

- `@odata.type`
	The type of a resource.
  - Type: `String`

- `Events`
	Array of event records with type `EventRecord`.
  - Type: array of `EventRecord`

- `Id`
	ID of the event.
  - Type: `String`

- `Name`
  Name of event.
  - Type: `String`

#### OPTIONAL Attributes

The following attributes are OPTIONAL to be present in the `Data` property.

- `@odata.context`
  The OData description of a payload. 
  - Type: `String`

- `Actions`
  The available actions for this resource.
  - Type: `Binary`.

- `Context`
   A context can be supplied at subscription time.  This property is the
   context value supplied by the subscriber.
  - Type: `String`

- `Description`
  The description of this resource. Used for commonality in the schema definitions.
  - Type: `String`

- `Oem`
  This is the manufacturer/provider specific extension.
  - Type: `Binary`

#### EventRecord
EventRecord is defined in Redfish schema
[Event_v1_4_1_EventRecord](https://redfish.dmtf.org/schemas/v1/Event.v1_4_1.json).
Additional information returned from Message Registry is added from
[Message.v1_0_8](https://redfish.dmtf.org/schemas/v1/Message.v1_0_8.json).


##### REQUIRED Attributes

The following attributes are REQUIRED to be present in the `EventRecord` property.

- `EventType`
	This indicates the type of event sent, according to the definitions in the EventService.
 
- `MessageId`
	This property shall be a key into message registry as described in the Redfish specification.

- `MemberId`
	 This is the identifier for the member within the collection.


##### OPTIONAL Attributes

The following attributes are OPTIONAL to be present in the `EventRecord` property.

- `Actions`
  The available actions for this resource.
  - Type: `Binary`.

- `Context`
   This property has been Deprecated in favor of Context found at the root level of the object.
  - Type: `String`

- `EventGroupId`
  This value is the identifier used to correlate events that came from the same cause.
  - Type: `Int`

- `EventId`
  The value of this property shall indicate a unique identifier for the event.
  - Type: `String`

- `EventTimestamp`
  This is time the event occurred.
  - Type: `String`.

- `Message`
  Full message of the event.
  - Type: `String`

- `Message`
  Message of the event.
  - Type: `String`

- `Message`
  This property shall contain an optional human readable message.
  - Type: `String`

- `MessageArgs`
  This array of message arguments are substituted for the arguments
	in the message when looked up in the message registry.
  - Type: array of `String`

- `MessageId`
  This property shall be a key into message registry as described in the Redfish specification.
  - Type: `String`

- `Oem`
  This is the manufacturer/provider specific extension.
  - Type: `Binary`

- `OriginOfCondition`
  This indicates the resource that originated the condition that caused the event to be generated.
  - Type: `Binary`

- `Severity`
  This is the severity of the event.
  - Type: `String`

- `Resolution`
  The following fields are defined in Redfish schema Message.v1_0_8.
  These are additional information returned from Message Registry.
	Used to provide suggestions on how to resolve the situation that caused the error.
  - Type: `String`


## Size Limits

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


## Example

### Cloud Native Events
The following example shows a Cloud Native Events serialized as JSON:
(Following json should be validated with Cloud native events event_spec.json schema)

```JSON
{
  "id": "5ce55d17-9234-4fee-a589-d0f10cb32b8e",
  "type": "event.synchronization-state-chang",
  "time": "2021-02-05T17:31:00Z",
  "data": {
    "version": "v1.0",
    "values": [
      {
        "resource": "/cluster/node/ptp",
        "data_type": "notification",
        "value_type": "enumeration",
        "value": "ACQUIRING-SYNC"
      },
      {
        "resource": "/cluster/node/clock",
        "data_type": "metric",
        "value_type": "decimal64.3",
        "value": 100.3
      }
    ]
  }
}
```

### Cloud Native Events with Redfish hardware event data
The following example shows a Cloud Native Events with Redfish data serialized as JSON:
(The Redfish data should be validated with https://redfish.dmtf.org/schemas/v1/Event.v1_4_1.json schema)

```JSON
{
  "id": "438704b1-cf54-4b81-b03a-491a76b0e371",
  "type": "HW_EVENT",
  "time": "2021-08-09T18:29:51.187379474Z",
  "data": {
    "version": "v1.0",
    "data": {
      "@odata.context": "/redfish/v1/$metadata#Event.Event",
      "@odata.id": "/redfish/v1/EventService/Events/5e004f5a-e3d1-11eb-ae9c-3448edf18a38",
      "@odata.type": "#Event.v1_3_0.Event",
      "Context": "any string is valid",
      "Events": [
        {
          "Context": "any string is valid",
          "EventId": "2162",
          "EventTimestamp": "2021-07-13T15:07:59+0300",
          "EventType": "Alert",
          "MemberId": "615703",
          "Message": "The system board Inlet temperature is less than the lower warning threshold.",
          "MessageArgs": [
            "Inlet"
          ],
          "MessageArgs@odata.count": 1,
          "MessageId": "TMP0100",
          "Severity": "Warning"
        }
      ],
      "Id": "5e004f5a-e3d1-11eb-ae9c-3448edf18a38",
      "Name": "Event Array"
    }
  }
}
```

### Cloud Events
The following example shows a Cloud Native Events embeded in CloudEvents and serialized as JSON:
(Following json should be validated with CloudEvents scheam at https://github.com/cloudevents/spec/blob/v1.0.1/spec.json)

```JSON
{
  "type": "event.synchronization-state-change",
  "source": "/cluster/node/ptp",
  "id": "789be75d-7ac3-472e-bbbc-6d62878aad4a",
  "time": "2021-02-05T17:31:00Z",
  "datacontenttype": "application/json",
  "specversion": "v1.0",
  "data": {
    "version": "1.0",
    "values": [
      {
        "type": "notification",
        "resource": "/sync/sync-status/sync-state",
        "value_type": "enumeration",
        "value": "HOLDOVER"
      },
      {
        "type": "notification",
        "resource": "/sync/sync-status/sync-state",
        "value_type": "enumeration",
        "value": "HOLDOVER"
      }
    ]
  }
}
```

### Cloud Events with Redfish hardware event data
The following example shows a Cloud Native Events with Redfish hardware event data embeded in CloudEvents and serialized as JSON:
(Following json should be validated with CloudEvents scheam at https://github.com/cloudevents/spec/blob/v1.0.1/spec.json)

```JSON
{
  "type": "HW_EVENT",
  "source": "/cluster/node/mynode/redfish/event",
  "id": "789be75d-7ac3-472e-bbbc-6d62878aad4a",
  "time": "2021-08-09T18:29:51.187379474Z",
  "datacontenttype": "application/json",
  "specversion": "v1.0",
  "data": {
    "version": "v1.0",
    "data": {
      "@odata.context": "/redfish/v1/$metadata#Event.Event",
      "@odata.id": "/redfish/v1/EventService/Events/5e004f5a-e3d1-11eb-ae9c-3448edf18a38",
      "@odata.type": "#Event.v1_3_0.Event",
      "Context": "any string is valid",
      "Events": [
        {
          "Context": "any string is valid",
          "EventId": "2162",
          "EventTimestamp": "2021-07-13T15:07:59+0300",
          "EventType": "Alert",
          "MemberId": "615703",
          "Message": "The system board Inlet temperature is less than the lower warning threshold.",
          "MessageArgs": [
            "Inlet"
          ],
          "MessageArgs@odata.count": 1,
          "MessageId": "TMP0100",
          "Severity": "Warning"
        }
      ],
      "Id": "5e004f5a-e3d1-11eb-ae9c-3448edf18a38",
      "Name": "Event Array"
    }
  }
}
```
