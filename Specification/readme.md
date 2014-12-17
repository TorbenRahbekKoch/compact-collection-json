# Compact Collection With Validation+JSON - Hypermedia format

## Description

Compact Collection With Validation+JSON is a JSON-based, Collection+JSON inspired read/write hypermedia-format. The format aims to allow for more auto-discoverable properties of the data from the REST-service it is supposed to be delivered by. It furthermore aims to keep the amount of data transferred for a request to a minimum by having sane defaults for as many settings as possible.

It allows for defining validation of various fields sent to the service when adding, updating or querying items.

Even though the format has been inspired by Collection+JSON it is not directly compatible.

Whereas Collection+JSON uses a name/value format for the data, this format uses anonymous objects, which takes up less space and amounts to less parsing and manipulation for the client.

#### Author:

Torben Rahbek Koch (torben@rahbekkoch.dk)

#### Status:

Definitely a work *in progress*.

## Overview

The format is a JSON-based format with several optional fields:

```json
{
  "compactcollection" :                  // REQUIRED: "compactcollection" to distinguish from Collection+JSON, even though the Content Type should ensure that.
  {
    "version"         : "1.0",           // OPTIONAL: The version of the format. If missing it **MUST** default to *"1.0"*
    "href"            : URI,             // OPTIONAL: The uri of the entire document. If missing it MUST default to the uri used to request the document.
    "links"           : [LINK],          // OPTIONAL: Navigational links
    "items"           : [ITEM],          // OPTIONAL: The data. MUST default to an empty array.
    "itemtemplate"    : ITEMTEMPLATE,    // OPTIONAL: A description of an item in "items"
    "writetemplates"  : [WRITETEMPLATE], // OPTIONAL: A description of add and update (generally POST and PUT) statements  
    "querytemplates"  : [QUERYTEMPLATE], // OPTIONAL: Query templates
    "error"           : ERROR            // OPTIONAL: A description of any error that caused a request to fail
  }
}
```
If none of the **OPTIONAL** elements are present, *"items"* MUST default to an empty array.

### "items" : [ITEM]

The "items" field is an array of ITEM elements. It is a child property of "collection". Each ITEM is an anonymous object:

```json
{
  "href"  : URI,                       // RECOMMENDED: The uri to use to address this specific ITEM.
  "links" : [LINK],                    // OPTIONAL: Navigational links for this item
  "data"  : OBJECT                     // OPTIONAL: The actual data laid out as described by ITEMTEMPLATE. MUST default to NULL.
}

```

**Note!** The **RECOMMENDED** state means that it really **SHOULD** be there but the originator **MAY** choose to leave it out, because the ITEM is, for some reason, not individually addressable. This also means that the ITEM is not writable.

### "links" : [LINK]

The "links" field is an array of LINK elements. It is a child property of "collection" or ITEM elements. Each LINK element is a navigational item, which - using Micoformats rel values - describes how to navigate onwards from the document or ITEM.

Each LINK is an anonomous object:

```json
{
  href : URI,                   // REQUIRED: The uri to navigate to
  rel  : STRING,                // REQUIRED: The microformat rel value -
  writetemplate : STRING        // OPTIONAL: The name of WRITETEMPLATE that describes the data to send to "href"
}
```

#### Templates

Templates are at the heart of this specification. They are used to describe:

- *"itemtemplate"* - The individual items returned from the service
- *"writetemplate"* - The data to send when data should be added or updated
- *"querytemplate"* - The data to send when querying the service for data.

##### "itemtemplate" : ITEMTEMPLATE

An *ITEMTEMPLATE* describes a record as it is retrieved from the server.

```json
  "itemtemplate" :
  {
    "href" : URI,                                       // OPTIONAL : URI to GET the entire ITEMTEMPLATE as individual data
    "fields" :                                          // OPTIONAL : The description of the fields of the ITEMTEMPLATE
    [
      {
        "prompt" : STRING,                              // OPTIONAL : A user friendly display name for the field
        "name"   : STRING,                              // REQUIRED : A system name for the field
        "value"  : DATETIME/STRING/NUMBER/URI/BOOL/NULL // OPTIONAL : The default value of the field, if not given in "data", missing equals NULL.
        "type"   : TYPE                                 // OPTIONAL : The type of the field, *DEFAULT* is *STRING*.
      }
    ]
  }
```
Even though both "href" and "fields" are listed as **OPTIONAL** it is **REQUIRED** to have **AT LEAST ONE** of them. It is, however, **RECOMMENDED** to have only "href" to which a GET should return a compactcollection+json document with just this one ITEMTEMPLATE. If both fields are given, the *"fields"* entry **MUST** take precedence over *"href"*. It is **RECOMMENDED** to only have one of the fields.

Using just the *"href"* field allows the client to download and cache the template for future use.

##### "writetemplates" : [WRITETEMPLATE]

A *WRITETEMPLATE* is an extension of the *"itemtemplate"* with optional validation descriptions for the data to send to the service.

```json
  "writetemplate" :
  {
      "name"   : STRING,                                  // REQUIRED : A unique (in [WRITETEMPLATE]) name, which can be referred to from a LINK.
      "href"   : URI,                                     // OPTIONAL : URI to GET this WRITETEMPLATE as individual data
      "method" : STRING,                                  // OPTIONAL : "POST" or "PUT" - which HTTP method to use when sending data.
      "fields" :                                          // OPTIONAL : The description of the fields of the WRITETEMPLATE
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
      ],
      "validation" :                                        // OPTIONAL: Record-wide validation
      {
        "validator"   : STRING                              // REQUIRED: The name/type of the validator
        "settings"    : OBJECT,                             // OPTIONAL: Each type of validator will have defaults. See description for each validator.
        "description" : STRING,                             // OPTIONAL: A user friendly description of the requirements for the field.
      }
  }
```
Even though both "href" and "fields" are listed as **OPTIONAL** it is **REQUIRED** to have at least one of them. It is, however, **RECOMMENDED** to have only "href" to which a GET should return a compactcollection+json document with just this one WRITETEMPLATE in "writetemplates".

Even though the *"validator"* is **REQUIRED**, it is only **REQUIRED** as part of *"validation"*, which is **OPTIONAL**.

If a client does not support a specific validator, it **SHOULD**  skip that validation and assume that the data are valid. It is up to the server to return a reasonable error message if the validation fails server side.

##### "querytemplates" : [QUERYTEMPLATE]

A *"QUERYTEMPLATE"* is part of the *"querytemplates"* section, which defines specific item queries that can be sent to the server.
Just as a *WRITETEMPLATE* can have validators given the same applies to a *QUERYTEMPLATE*

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

### "error" : ERROR

When an error happens on the server, it is vital to be able to advise the user on how to proceed. The *"error"* field provides a load of properties to help with this.
[HTTP status code](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)

```json
  "error":                                 // OPTIONAL
  {
    "code"               : INT,            // REQUIRED: The HTTP status code, which **SHOULD** be the same as returned in the HTTP document containing the JSON
    "message"            : STRING,         // OPTIONAL: User friendly description of the problem
    "errorclass"         : STRING,         // OPTIONAL: The error class, default **MUST** be "UNRECOVERABLE"
    "detailedErrorClass" : STRING,         // OPTIONAL: Detailed error class, e.g. "TIMEOUT", default is empty
    "messageid"          : STRING,         // OPTIONAL: Message/log id for the log message which may have been logged for the error, can be used to correlate interaction with support.
    "correlationid"      : STRING,         // OPTIONAL: message/log correlation id for the log message which may have been logged for the error",
    "extended"           : OBJECT,         // OPTIONAL: extended error data, the contents of which depends on *"detailederrorclass"*
    "links"              : [LINK]          // OPTIONAL: Navigational links to aid the client in escaping the error.
  }
```

#### Error Classes

There are, basically [three different error classes](http://softwarepassion.eu/error-handling-the-easy-way/):

  - Unrecoverable
  - Recoverable
  - Userrecoverable

"errorClass" MUST be one of those three values and it MUST NOT be case-sensitive. The DEFAULT, if left out, is "Unrecoverable".

There are many more detailed error classes, all of which can be grouped under one of the main error classes above, e.g.

  - Authorization
  - Timeout
  - Invaliddata

Each application is free to define their own detailed error classes as they see fit.

### TYPE

Description of types.

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
