% RestDoc - Documenting REST APIs
% Thorsten Hoeger - Taimos GmbH - thorsten.hoeger@taimos.de
 name - company - mail
% October 2012

## Abstract

## 1. Introduction

### 1.1 About REST

REpresentational State Transfer (REST) is a style of software architecture for distributed systems such as the World Wide Web. The term representational state transfer was introduced and defined in 2000 by Roy Fielding in his doctoral dissertation.\[[REST]\] 

The REST architectural style was developed in parallel with HTTP/1.1, based on the existing design of HTTP/1.0. The largest implementation of a system conforming to the REST architectural style is the World Wide Web. REST exemplifies how the Web's architecture emerged by characterizing and constraining the macro-interactions of the four components of the Web, namely origin servers, gateways, proxies and clients, without imposing limitations on the individual participants. As such, REST essentially governs the proper behavior of participants.
REST-style architectures consist of clients and servers. Clients initiate requests to servers; servers process requests and return appropriate responses. Requests and responses are built around the transfer of representations of resources. A resource can be essentially any coherent and meaningful concept that may be addressed. A representation of a resource is typically a document that captures the current or intended state of a resource.
The client begins sending requests when it is ready to make the transition to a new state. While one or more requests are outstanding, the client is considered in transition. The representation of each application state contains links that may be used the next time the client chooses to initiate a new state transition.
REST facilitates the transaction between web servers by allowing loose coupling between different services. REST is less strongly typed than its counterpart, SOAP. The REST language is based on the use of nouns and verbs, and has an emphasis on readability.

Source: Wikipedia\[[WikiREST]\]

### 1.2. Terminology
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119. \[[RFC2119]\]

### 1.3 REST API Example

For this specification a simple API for setting and retrieving localized messages is used.

~~~~~
---------------------- ----------------- --------- ------------------------ --------------
Resource               Id                Methods   Representation           Status Codes
---------------------- ----------------- --------- ------------------------ --------------
/{locale}/{messageId}  LocalizedMessage  GET, PUT  message format (string)  200, 201, 404
/fallback/{locale}     FallbackLocale    GET, PUT  language format (string) 200, 201
---------------------- ----------------- --------- ------------------------ --------------
~~~~~

To update or create a new localized message you PUT a string value to it's /{locale}/{messageId} URI. To assign a fallback locale for missing messages, you PUT a locale name to /fallback/{locale}.

## 2. Components

The RestDoc model consist of different components, which are described in the following sections. Each component is part of the root model or is part of another component. The overall structure of RestDoc in defined in chapter [3.1](#structure)

### 2.1 Resource

A _resource_ MUST have a unique _id_ that names the resource and SHOULD have a _description_ containing further information about the intented use of the given resource. The _path_ MUST be part of the resource definition and MUST be unique. The path is a URI \[[RFC2396]\] or URI template \[[RFC6570]\]. If the path contains variables the resource MUST contain a set of _parameter_ definitions. The resource MUST contain one definition for each available HTTP _method_.

In the running example there are 2 resources with the paths "/{locale}/{messageId}" and "/fallback/{locale}".

### 2.2 Parameter

A _parameter_ definition is used to define variable expansion for URI templates used in resources paths. A parameter definition SHOULD contain a _description_ describing the given parameter. A parameter MAY define _validations_ for the content.

In the running example there are 2 parameters defined: `locale` and `messageId`.

### 2.3 Validation

_Validations_ define means to check the content of parameters. Validations are a set of type/pattern pairs with _type_ defining the validation type and _pattern_ containing the regular expression to apply.
Currently the only supported type is _match_. Multiple entries within one validation set are disjunctive, so the entries are ORed.

### 2.4 Method

A _method_ entry represents the pair of _HTTP Verb_ and _resource_. It SHOULD contain a _description_ that describes the effect of calling this method. It also SHOULD contain a set of _status code_ definitions. A Status Code is represented by an integer with the HTTP Status Code and a textual definition what this return code means. The method SHOULD contain a set of accepted MediaTypes and SHOULD contain a set of request _headers_. Further the method SHOULD contain a _response_ definition. In addition a method MAY provide a set of _examples_, each consisting of an expanded path (no templates), a body and optional headers.

In the running example each resource has the two methods GET and PUT. The resource "/{locale}/{messageId}" can return a different set of status codes depending on which method is requested: GET can return 200(OK), or 404(Not Found), while PUT should only ever return 201(Created).

### 2.5 Header

A _header_ is defined by its name as it is used in the HTTP message and SHOULD contain a _description_. A header can be required or optional with optional being the default. According to the position of the header definition they are API global or method local.

### 2.6 MediaTypes

Request and response MediaTypes can be defined via a type/schema pair. _type_ is the MediaType as defined in \[[RFC4288]\] and schema defines the URI of the content schema. (e.g. XSD, JSON-Schema, etc) The schema field is optional.

### 2.7 Response

The response definition consists of a set of MediaTypes for return values and a set of response headers. Both sets are optional.

### 2.8 Schema

A schema is identified by its URI. There are two types of schema definitions. If the MediaType is JSON the schema can be defined inline otherwise the schema definition points to the URL containing the schema document.

## 3. RestDoc

### 3.1 Structure

The overall structure of a RestDoc documentation consists of an optional schema set, an optional global headers set and a required resource set.

### 3.2 Retrieval

The retrieval of the RestDoc documentation is done via the HTTP OPTIONS verb. Querying the server with an asterisk as path (``OPTIONS * HTTP/1.1``) MUST return a RestDoc documentation with all resources available.

Performing an OPTIONS request with an unexpanded URI path MUST return a RestDoc resource description. Given our example application the request ``OPTIONS /%7Blocale%7D/%7BmessageId%7D HTTP/1.1`` MUST return the RestDoc resource description for the "LocalizedMessage" resource.

## 4. Representations

### 4.1 JSON

The default representation of RestDoc is JSON with the custom MediaType ``application/x-restdoc+json``. In the following you find a textual definition of the RestDoc JSON representation.

The root object contains of two object with the names "schema" and "headers" and an array "resources". 

The "schema" object contains objects with the schema URI as name and the schema definition as content. The content contains a field type with value "inline" or "url". Inline schemas provide a schema object with inline JSON-Schema and URL schemas provide a field url with the URL of the schema document.

~~~~~ {.javascript}
  "schemas" : {
    "http://some.json/msg" : {
      "type" : "inline",
      "schema" : {
        "type" : "object",
        "properties" : {
          "id" : {
            "type" : "string"
          },
          "content" : {
            "type" : "string"
          }
        }
      }
    },
	"http://some.xml/msg" : {
	  "type" : "url",
	  "url" : "http://some.xml/msg.xsd"
	}
  },
~~~~~

The "headers" object contains a "request" and a "response" object. The contents of these objects are itself objects with the header name as field name and the content as described in [Section 2.5](#header).

~~~~~ {.javascript}
  "headers": {
    "request": {
      "Authorization": {
        "description": "See http://myapi.com/docs/auth",
        "required": true
      }
    },
    "response": {
      "X-RateLimit-Total": {
        "description": "The number of API calls allowed per-hour"
      }
    }
  },
~~~~~

The resources array contains of objects with the following fields:

- id: the id of the resource
- description: the resource description
- path: the resource URI or URI template
- params: the parameter object
- methods: the method object

The _params_ object has the parameter names as field names and an object as content.

- description: the parameter description
- validations: array of parameter validations
	- type: the validation type (match)
	- pattern: the regular expression

~~~~~ {.javascript}
  "params": { // URI parameters descriptions
    "locale": {
      "description": "A standard locale string, e.g. \"en_US.utf-8\"",
      "validations": [ { 
	    "type": "match", 
		"pattern": "[a-z]+(_[A-Z]+)?(\\\\.[a-z-]+)?" 
      } ]
    }
  },
~~~~~

The _methods_ object has HTTP verbs as field names and an object as content.

- description: the method description
- statusCodes: the status code object
- accepts: the accept MediaTypes
- headers: the headers object
- response: the response object with types and headers
	- types: the response MediaTypes
	- headers: the response headers object
- examples: array of example objects
	- path: the expanded path URI
	- headers: request headers
	- body: the example request body

see [Section 5.1](#full-restdoc-example) for a full example of a JSON representation of RestDoc

## 5. Examples

### 5.1 Full RestDoc example

~~~~~ {.javascript}
OPTIONS * HTTP/1.1
Accept: application/json;

{
  "schemas" : {
    "http://some.json/msg" : {
      "type" : "inline",
      "schema" : {
        "type" : "object",
        "properties" : {
          "id" : {
            "type" : "string"
          },
          "content" : {
            "type" : "string"
          }
        }
      }
    }
  },
  // Headers can be described at the server level and/or per-method
  "headers": {
    "request": {
      "Authorization": {
        "description": "This server uses a custom authentication scheme. See http://myapi.com/docs/auth",
        "required": true
      }
    },
    "response": {
      "X-RateLimit-Total": {
        "description": "The number of API calls allowed per-hour"
      },
      "X-RateLimit-Remaining": {
        "description": "Number of requests remaining until next refill"
      },
      "X-RateLimit-Reset": {
        "description": "The time at which X-RateLimit-Remaining will be reset back to X-RateLimit-Total"
      }
    }
  },
  "resources": [
    {
      "id": "LocalizedMessage",
      "description": "A localized message",
      "path": "/{locale}/{messageId}{?seasonal}", // representing query params with L3 URI templates
      "params": { // URI parameters descriptions
        "locale": {
          "description": "A standard locale string, e.g. \"en_US.utf-8\"",
          "validations": [ { "type": "match", "pattern": "[a-z]+(_[A-Z]+)?(\\\\.[a-z-]+)?" } ]
        },
        "messageId": {
          "description": "A free-form message string",
          "validations": [ { "type": "match", "pattern": "[a-z_]+" } ]
        },
        "seasonal": {
          "description": "Whether the message is seasonal.",
          "validations": [ { "type": "match", "pattern": "^(true|false|yes|no)$" } ]
        }
      },
      "methods": {
        "PUT": {
          "description": "Update or create a message",
          "statusCodes": { "201": "Created" },
          "accepts": [   // Representations accepted by the method on this resource.
            { "type": "text/plain" },
            { "type": "application/json", "schema": "http://some.json/msg" }
          ],
          "headers": { // Request headers only, response headers are defined under 'response'
            "X-User-Token": {
              "description": "Used to identify the user creating the message"
            }
          },
          "response": { // Response representations this resource/method provides
            "types": [
              { "type": "text/plain" },
              { "type": "application/json", "schema": "http://some.json/msg" },
            ],
            "headers": {
              "X-What-Response-Header-Would-Only-Show-Up-For-One-Resource?": {}
            }
          },
          "examples": [
            {
              "path": "/en_US/greeting",
              "body": "Hello, world!"
            },
            {
              "path": "/en_US/greeting",
              "headers": {"Content-Type": "application/json"},
              "body": "{\"id\":\"greeting\",\"content\":\"Hello, world!\"}!"
            }
          ]
        },
        "GET": {
          "description": "Retrieve a message",
          "statusCodes": { 
             "200": "Message retrieved successfully", 
             "404": "Message not found"
          },
          "accepts": [] // Explicitly indicate that this method takes no request body. 
        }
      }
    },
    {
      // This resource has no human-readable documentation, but still provides some info on how to use it.
      "id": "FallbackLocale",
	  "path": "/fallback/{locale}",
      "params": {
        "locale": { 
          "validations": [ { "type": "match", "pattern": "[a-z]+(_[A-Z]+)?(\\\\.[a-z-]+)?" } ]
        }
      },
      "methods": {
        "GET": { "statusCodes": { "200": "OK" } },
        "PUT": { "statusCodes": { "201": "Created" } }
      }
    }
  ]
}
~~~~~

## 6. Extensions

### 6.1 Extensibility

RestDoc is designed to be extensible. Therefore any additional fields not named in this specification MUST be ignored by implementations that do not understand them. But implementations MUST NOT rely on any additional fields to be RestDoc compliant. Extensions MUST NOT change the meaning of fields defined in this specification.

An extension may for example defined other validation types than "match" but must not change the behavior of the type "match".

### 6.2 Naming conventions

Extensions to RestDoc are given a unique name beginning with _RestDoc-_. The example extension above could be named _RestDoc-Validations_.

## 7. Copyright

This specification is copyrighted by the authors named in section 7.1. Is is free to use for any purposes commercial or non-commercial.

### 7.1 Authors

The following authors are responsible for the RestDoc core-specification:

~~~~
Thorsten Hoeger
Taimos GmbH
Hohenzollernstrasse 32
D-73262 Reichenbach
thorsten.hoeger@taimos.de
~~~~
```
Saar Yahalom
Microsoft
Shenkar 13, Hertzeliya
Israel
saary .at. microsoft.com
```
~~~~
please fill in other authors
~~~~

### 7.2 Contact

This specification and any related work is located at <http://www.restdoc.org> and <https://github.com/RestDoc>. 

## A. References

### A.1 Normative

[RFC2119]: http://www.ietf.org/rfc/rfc2119.txt "RFC 2119 - Key words for use in RFCs to Indicate Requirement Levels"
\[RFC2119\]: RFC 2119 - Key words for use in RFCs to Indicate Requirement Levels

[RFC4627]: http://www.ietf.org/rfc/rfc4627.txt "RFC 4627 - The application/json Media Type for JavaScript Object Notation (JSON)"
\[RFC4627\]: RFC 4627 - The application/json Media Type for JavaScript Object Notation (JSON)

[RFC6570]: http://www.ietf.org/rfc/rfc6570.txt "RFC 6570 - URI Template"
\[RFC6570\]: RFC 6570 - URI Template

[RFC4288]: http://www.ietf.org/rfc/rfc4288.txt "RFC 4288 - Media Type Specifications and Registration Procedures"
\[RFC4288\]: RFC 4288 - Media Type Specifications and Registration Procedures

[RFC2396]: http://www.ietf.org/rfc/rfc2396.txt "RFC 2396 - Uniform Resource Identifiers (URI): Generic Syntax"
\[RFC2396\]: RFC 2396 - Uniform Resource Identifiers (URI): Generic Syntax

[RFC2616]: http://www.ietf.org/rfc/rfc2616.txt "RFC 2616 - Hypertext Transfer Protocol -- HTTP/1.1"
\[RFC2616\]: RFC 2616 - Hypertext Transfer Protocol -- HTTP/1.1

[XML]: http://www.w3.org/TR/REC-xml "W3C - Extensible Markup Language (XML) 1.0 (Fifth Edition)"
\[XML\]: W3C - Extensible Markup Language (XML) 1.0 (Fifth Edition)

### A.1 Informative

[REST]: http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm "R. Fielding - Architectural Styles and the Design of Network-based Software Architectures, Chapter 5"
\[REST\]: R. Fielding - Architectural Styles and the Design of Network-based Software Architectures, Chapter 5

[WikiREST]: http://en.wikipedia.org/wiki/Representational_state_transfer "Wikipedia - Representational state transfer"
\[WikiREST\]: "Wikipedia - Representational state transfer"
