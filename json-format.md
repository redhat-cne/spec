# JSON Event Format for O-RAN Event - Version 0.0.1

## Abstract

The JSON Format for O-RAN Event defines how events are expressed in JavaScript
Object Notation (JSON) Data Interchange Format ([RFC8259][rfc8259]).

## Table of Contents

1. [Introduction](#1-introduction)
2. [JSON Batch Format](#4-json-batch-format)
3. [References](#5-references)

## 1. Introduction

[O-RAN Event][ce] is a standardized and protocol-agnostic definition of the
structure and metadata description of events for  cloud applications.
 This specification defines how the
elements defined in the O-RAN Event specification are to be represented in the
JavaScript Object Notation (JSON) Data Interchange Format ([RFC8259][rfc8259]).


### 1.1. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC2119][rfc2119].


### 2.2. Type System Mapping

The [O-RAN Event type system][ce-types] MUST be mapped to JSON types as follows,
with exceptions noted below.

| CloudEvents   | JSON                                                           |
| ------------- | -------------------------------------------------------------- |
| Boolean       | [boolean][json-bool]                                           |
| Integer       | [number][json-number], only the `int` component is permitted   |
| String        | [string][json-string]                                          |
| URI           | [string][json-string] following [RFC 3986][rfc3986]            |
| URI-reference | [string][json-string] following [RFC 3986][rfc3986]            |
| Timestamp     | [string][json-string] following [RFC 3339][rfc3339] (ISO 8601) |

Unset attributes MAY be encoded to the JSON value of `null`. When decoding
attributes and a `null` value is encountered, it MUST be treated as the
equivalent of unset or omitted.

Extension specifications MAY define secondary mapping rules for the values of
attributes they define, but MUST also include the previously defined primary
mapping.

For instance, the attribute value might be a data structure defined in a
standard outside of O-RAN Event, with a formal JSON mapping, and there might be
risk of translation errors or information loss when the original format is not
preserved.

An extension specification that defines a secondary mapping rule for JSON, and
any revision of such a specification, MUST also define explicit mapping rules
for all other event formats that are part of the O-RAN Event core at the time of
the submission or revision.

If required, like when decoding Maps, the O-RAN Event type can be determined by
inference using the rules from the mapping table, whereby the only potentially
ambiguous JSON data type is `string`. The value is compatible with the
respective O-RAN Event type when the mapping rules are fulfilled.

### 2.3. Examples

The following table shows exemplary attribute mappings:

| CloudEvents     | Type             | Exemplary JSON Value    |
| --------------- | ---------------- | ----------------------- |
| type            | String           | "com.example.someevent" |
| specversion     | String           | "1.0"                   |
| source          | URI-reference    | "/mycontext"            |
| id              | String           | "1234-1234-1234"        |
| id              | String           | null        |
| time            | Timestamp        | "2018-04-05T17:31:00Z"  |
| time            | Timestamp (null) | null                    |
| data            | array            |    |

### 2.4. JSONSchema Validation

The O-RAN event [JSONSchema](http://json-schema.org) for the spec is located
[here](spec.json) and contains the definitions for validating events in JSON.

## 3. Envelope

Each O-RAN event can be wholly represented as a JSON object.

Such a representation MUST use the media type `application/cloudevents+json`.

All REQUIRED and all not omitted OPTIONAL attributes in the given event MUST
become members of the JSON object, with the respective JSON object member name
matching the attribute name, and the member's type and value being mapped using
the [type system mapping](#22-type-system-mapping).

OPTIONAL not omitted attributes MAY be represeted as a `null` JSON value.

### 3.1. Handling of "data"

Before taking action, a JSON serializer MUST first determine the runtime data
type of the `data` content.

The implementation MUST translate the data value into a
[JSON value][json-value], 

### 3.2. Examples


```JSON
{
    "specversion" : "1.0",
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



## 4. JSON Batch Format

In the _JSON Batch Format_ several O-RAN event are batched into a single JSON
document. The document is a JSON array filled with O-RAN event in the [JSON
Event format][json-format].

Although the _JSON Batch Format_ builds ontop of the _JSON Format_, it is
considered as a separate format: a valid implementation of the _JSON Format_
doesn't need to support it. The _JSON Batch Format_ MUST NOT be used when only
support for the _JSON Format_ is indicated.

### 4.1. Mapping O-RAN Event

This section defines how a batch of O-RAN event is mapped to JSON.

The outermost JSON element is a [JSON Array][json-array], which contains as
elements O-RAN event rendered in accordance with the [JSON event
format][json-format] specification.

### 4.2. Envelope

A JSON Batch of O-RAN event MUST use the media type
`application/json`.

### 4.3. Examples

An example containing  O-RAN event: JSON data.

```JSON

    {
    "specversion" : "1.0",
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

An example of an empty batch of O-RAN EVENT (typically used in a response, but
also valid in a request):

```JSON
[]
```

## 5. References

- [RFC2046][rfc2046] Multipurpose Internet Mail Extensions (MIME) Part Two:
  Media Types
- [RFC2119][rfc2119] Key words for use in RFCs to Indicate Requirement Levels
- [RFC4627][rfc4627] The application/json Media Type for JavaScript Object
  Notation (JSON)
- [RFC4648][rfc4648] The Base16, Base32, and Base64 Data Encodings
- [RFC6839][rfc6839] Additional Media Type Structured Syntax Suffixes
- [RFC8259][rfc8259] The JavaScript Object Notation (JSON) Data Interchange
  Format

[base64]: https://tools.ietf.org/html/rfc4648#section-4
[ce]: ./spec.md
[ce-types]: ./spec.md#type-system
[content-type]: https://tools.ietf.org/html/rfc7231#section-3.1.1.5
[json-format]: ./json-format.md
[json-geoseq]:https://www.iana.org/assignments/media-types/application/geo+json-seq
[json-object]: https://tools.ietf.org/html/rfc7159#section-4
[json-seq]: https://www.iana.org/assignments/media-types/application/json-seq
[json-bool]: https://tools.ietf.org/html/rfc7159#section-3
[json-number]: https://tools.ietf.org/html/rfc7159#section-6
[json-string]: https://tools.ietf.org/html/rfc7159#section-7
[json-value]: https://tools.ietf.org/html/rfc7159#section-3
[json-array]: https://tools.ietf.org/html/rfc7159#section-5
[rfc2046]: https://tools.ietf.org/html/rfc2046
[rfc2119]: https://tools.ietf.org/html/rfc2119
[rfc3986]: https://tools.ietf.org/html/rfc3986
[rfc4627]: https://tools.ietf.org/html/rfc4627
[rfc4648]: https://tools.ietf.org/html/rfc4648
[rfc6839]: https://tools.ietf.org/html/rfc6839#section-3.1
[rfc8259]: https://tools.ietf.org/html/rfc8259
[rfc3339]: https://www.ietf.org/rfc/rfc3339.txt