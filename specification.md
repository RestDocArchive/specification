% RestDoc - Documenting REST APIs
% Thorsten Hoeger thorsten.hoeger@taimos.de
% October 2012

## Abstract

## 1. Introduction

### 1.1 REST

### 1.2 Use Cases

### 1.3 REST API Example

For this specification a simple API for setting and retrieving localized messages is used.

~~~~~
---------------------- --------- ------------------------ --------------
Resource               Methods   Representation           Status Codes
---------------------- --------- ------------------------ --------------
/{locale}/{messageId}  GET, PUT  message format (string)  200, 201, 404
/fallback/{locale}     GET, PUT  language format (string) 200, 201
---------------------- --------- ------------------------ --------------
~~~~~

To update or create a new localized message you PUT a string value to it's /{locale}/{messageId} URI. To assign a fallback locale for missing messages, you PUT a locale name to /fallback/{locale}.

## 2. Components

### 2.1 Resource

### 2.2 Parameter

### 2.3 Validation

### 2.4 Method

### 2.5 Header

### 2.6 Response

### 2.7 Schema

## 3. RestDoc

### 3.1 Structure

## 4. Representations

### 4.1 JSON

see [Section 5.2](#extended-restdoc-example)

### 4.2 Extensions

## 5. Examples

### 5.2 Extended RestDoc example

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
      "methods": {
        "GET": { "statusCodes": { "200": "OK" } },
        "PUT": { "statusCodes": { "201": "Created" } }
      },
      "params": {
        "locale": { 
          "validations": [ { "type": "match", "pattern": "[a-z]+(_[A-Z]+)?(\\\\.[a-z-]+)?" } ]
        }
      }
    }
  ]
}
~~~~~


## 6. Extensions

### 6.1 Extensibility

### 

## A. References

### A.1 Normative

[RFC2119]: http://www.ietf.org/rfc/rfc2119.txt "RFC 2119 - Key words for use in RFCs to Indicate Requirement Levels"
\[RFC2119\]: RFC 2119 - Key words for use in RFCs to Indicate Requirement Levels

[RFC4627]: http://www.ietf.org/rfc/rfc4627.txt "RFC 4627 - The application/json Media Type for JavaScript Object Notation (JSON)"
\[RFC4627\]: RFC 4627 - The application/json Media Type for JavaScript Object Notation (JSON)

[RFC6570]: http://www.ietf.org/rfc/rfc6570.txt "RFC 6570 - URI Template"
\[RFC6570\]: RFC 6570 - URI Template

[RFC2616]: http://www.ietf.org/rfc/rfc2616.txt "RFC 2616 - Hypertext Transfer Protocol -- HTTP/1.1"
\[RFC2616\]: RFC 2616 - Hypertext Transfer Protocol -- HTTP/1.1

[XML]: http://www.w3.org/TR/REC-xml "Extensible Markup Language (XML) 1.0 (Fifth Edition)"
\[XML\]: Extensible Markup Language (XML) 1.0 (Fifth Edition)

### A.1 Informative

[REST]: http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm "R. Fielding - Architectural Styles and the Design of Network-based Software Architectures, Chapter 5"
\[REST\]: R. Fielding - Architectural Styles and the Design of Network-based Software Architectures, Chapter 5