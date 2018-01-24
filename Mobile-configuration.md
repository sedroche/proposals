# Standarize mobile configuration format

## Introduction

When generating 

## Problem Description

Mobile configuration format is not standarized. It contains various fields that have no meaning for the mobile users.
Configuration also exposes some internal fields. 

```json
{
 "config": {
          "headers": {},
          "mysql_db": "unifiedpush",
          "mysql_password": "unifiedpush",
          "mysql_username": "unifiedpush",
          "name": "unified-push-server",
          "type": "unified-push-server",
          "uri": "http://ups-myproject.192.168.37.1.nip.io"
        },
        "name": "unified-push-server"
}
```

## Expectations

- Provide mobile configuration that has only esential fields for mobile users
- Standarize format so users to not break functionalities once users will start using configuration
 
## Proposed Solution

Provide standard for mobile configuration that will be extendable and have backwards compatibility.

### Mobile cli specifics

Mobile-cli currently print configuration out to the std. 
Mobile-cli should have option to create `mobile-services.json` file in specified location.

### Json format specific

#### Top level object
```json
{
 "version" : "number",
 "namespace": "string",
 "services": { "type": "array", "items":"Service"}
}
```

### Service object

{
  "id": "string",
  "name": "string",
  "type": "string",
  "url": "string",
  "metadata": "array"

}

