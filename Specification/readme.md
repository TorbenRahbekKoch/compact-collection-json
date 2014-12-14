# Compact Collection With Validation+JSON - Hypermedia format

## Description

Compact Collection With Validation+JSON is a JSON-based, Collection+JSON inspired read/write hypermedia-format. The format aims to allow for more auto-discoverable properties of the data from the REST-service it is supposed to be delivered by. It furthermore aims to keep the amount of data transferred for a request to a minimum by having sane defaults for as many settings as possible.

It allows for defining validation of various fields sent to the service when adding, updating or querying items.

Even though the format has been inspired by Collection+JSON it is not directly compatible.

#### Author:

Torben Rahbek Koch (torben@rahbekkoch.dk)

#### Status:

Definitely a work *in progress*.

## Overview

The format is a JSON-based format with several optional fields:

```json
{
  "collection" :
  {
    "version" : "1.0",        // OPTIONAL: The version of the format. If missing it **MUST** default to *"1.0"*
    "href"    : URI,          // OPTIONAL: The uri of the document. If missing it MUST default to the uri used to request the document.
    "links"   : [LINK],       // OPTIONAL: Navigational links
    "items"   : [ITEM],       // OPTIONAL: The data. MUST default to no data.
    "queries" : [QUERY],      // OPTIONAL: Query templates
    "itemtemplate" : ITEMTEMPLATE,  // OPTIONAL: A description of a item in "items"
    "writetemplate" : WRITETEMPLATE, // OPTIONAL: A description of add and update statements  
  }
}
```

### "items"

The "items" field is an array of ITEM elements. It is a child property of "collection". Each ITEM is an anonymous object:

```json
{
  "href"  : URI,                       // REQUIRED: The uri to use to request this specific ITEM.
  "links" : [LINK],                    // OPTIONAL: Navigational links for this item
  "data"  : OBJECT                      // OPTIONAL: The actual data laid out as described by "itemtemplate". MUST default to NULL.
}

```

### "links"

The "links" field is an array of LINK elements. It is a child property of "collection" or ITEM elements. Each LINK element is a navigational item, which - using Micoformats rel values - describes how to navigate on from the document or ITEM.

Each LINK is an anonomous object:

```json
{
  href : URI,                   // REQUIRED: The uri to navigate to
  rel  : STRING,                // REQUIRED: The microformat rel value
}
```

#### Templates

Templates are at the heart of this specification. They are used to describe:

- *"itemtemplate"* - The individual items returned from the service
- *"writetemplate"* - The data to send when an item should be added or updated
- *"querytemplate"* - The data to send when querying the service for data.

##### "itemtemplate"

An *"itemtemplate"* describes a record as it is retrieved from the server.

```json
  "itemtemplate" :
  {
    "href" : URI, // URI to obtain the entire "itemtemplate" as an individual data
    "fields" :
    [
      {
        "prompt" : STRING,                              // OPTIONAL: A user friendly display name for the field
        "name"   : STRING,                              // REQUIRED: A system name for the field
        "value"  : DATETIME/STRING/NUMBER/URI/BOOL/NULL // OPTIONAL: The value of the field, missing equals NULL.
        "type"   : TYPE                                 // REQUIRED: The type of the field
      }
    ]
  }
```

**AT LEAST ONE** of the fields *"href"* and *"fields"* **MUST** be given. If both fields are given, the *"fields"* entry **MUST** take precedence over *"href"*. It is **RECOMMENDED** to only have one of the fields.

Using just the *"href"* field allows the client to download and cache the template for future use.

##### "writetemplate"

A *"writetemplate"* is an extension of the *"itemtemplate"* with optional validation descriptions for the data to send to the service.

```json
  "writetemplate" :
    {
      "href"   : URI, // URI to obtain the entire writetemplate as an individual data
      "fields" :
      [
        {
          "prompt" : STRING,                              // OPTIONAL: A user friendly display name for the field
          "name"   : STRING,                              // REQUIRED: A system name for the field
          "value"  : DATETIME/STRING/NUMBER/URI/BOOL/NULL // OPTIONAL: The value of the field, missing equals NULL
          "type"   : TYPE                                 // OPTIONAL: The type of the field, missing equals STRING
          "required"  : BOOL,                             // OPTIONAL: Whether the field is required or not, missing equals TRUE
          "validation" :                                  // OPTIONAL: How to validate the individual field before sending to server.
          {
            "validator" : STRING,                         // REQUIRED: The name/type of the validator
            "settings" :  OBJECT,                         // OPTIONAL: Each type of validator will have defaults. See description for each validator.
            "description" : STRING                        // OPTIONAL: A user friendly description of the requirements for the field.
          }
        }
      }
    ],
    "validation" :                                        // OPTIONAL: Record-wide validation
    {
      "validator"   : STRING                              // REQUIRED: The name/type of the validator
      "settings"    : OBJECT,                             // OPTIONAL: Each type of validator will have defaults. See description for each validator.
      "description" : STRING,                             // OPTIONAL: A user friendly description of the requirements for the field.
    }
  }
```
Even though the *"validator"* is **REQUIRED**, it is only **REQUIRED** as part of *"validation"*, which is **OPTIONAL**.

If a client does not support a specific validator, it **SHOULD**  skip that validation and assume that the data are valid. It is up to the server to return a reasonable error message if the validation fails server side.

##### "querytemplate"

A *"querytemplate"* is part of the *"queries"* section, which defines specific item queries that can be sent to the server.
Just as a *"writetemplate"* can have validators given the same applies to a *"querytemplate"*

```json
  "querytemplate" :
  {
    "href"  : URI,      // REQUIRED: The uri to follow to GET the query
    "rel"   : STRING,   // REQUIRED: Microformat rel value  
    "title" : STRING,   // OPTIONAL: User friendly display name for the entire query
    "name"  : STRING,   // OPTIONAL: A unique (in "queries") name for the query.
    "fields":           //
    [
      {
        "prompt" : STRING,                                // OPTIONAL: A user friendly display name for the field
        "name"   : STRING,                                // REQUIRED: A system name for the field
        "value"  : DATETIME/STRING/NUMBER/URI/BOOL/NULL,  // OPTIONAL: The value of the field, missing equals NULL
        "type"   : TYPE,                                  // OPTIONAL: The type of the field, missing equals STRING
        "validation" :                                    // OPTIONAL: How to validate the individual field before sending to server.
        {
          "validator" : STRING,                           // REQUIRED: The name/type of the validator
          "settings" :  OBJECT,                           // OPTIONAL: Each type of validator will have defaults. See description for each validator.
          "description" : STRING                          // OPTIONAL: A user friendly description of the requirements for the field.
        }
      }
    ],
    "validation" :                                        // Query-wide validation
    {
      "validator"   : STRING,                             // REQUIRED: The name/type of the validator
      "settings"    : OBJECT,                             // OPTIONAL: Each type of validator will have defaults. See description for each validator.
      "description" : STRING,                             // OPTIONAL: A user friendly description of the requirements for the field.
    }
  }
```


**NOTE!** The validators work in combination with the "type" and the "required" fields. If a value cannot be coerced to the given "type" it is not necessary to invoke the validators.

**NOTE!** Having the "validation" in the document does not imply that you should not validate on the server!

  "queries" :
  [
    { "href" : URI,       // The uri to follow to execute the query
      "rel"  : STRING,    // Microformat rel
      "title" : STRING,   // User friendly name for the entire query  
      "name"   : STRING,  // unique (in "queries" name for the query)
      "querytemplate" : TEMPLATE

  ]

### Error Object

When an error happens on the server, it is vital to be able to advise the user on how to proceed. The *"error"* field provides a load of properties to help with this.
[HTTP status code](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)

```json
  "error":
  {
    "code"       : STRING,                        // REQUIRED: The HTTP status code, which should be the same as returned in the HTTP document containing the JSON
    "message"    : STRING,                        // OPTIONAL: User friendly description of the problem
    "errorclass" : STRING,                        // OPTIONAL: The error class, default **MUST** be "UNRECOVERABLE"
    "detailedErrorClass" : STRING,                // OPTIONAL: Detailed error class, e.g. "TIMEOUT", default is empty
    "messageid"  : STRING,                        // OPTIONAL: Message/log id for the log message which may have been logged for the error, can be used to correlate interaction with support.
    "correlationid" : STRING,                     // OPTIONAL: message/log correlation id for the log message which may have been logged for the error",

    "extended" : OBJECT                           // OPTIONAL: extended error data, the contents of which depends on *"detailederrorclass"*

  }
```

### Validators

Here will be described the various kinds of validators, how they are supposed to validate and what extra data they need.

- range
- minLength
- maxLength
- email
- url,
- href (service used to validate value)
- and
- or
- compare (with other fields)