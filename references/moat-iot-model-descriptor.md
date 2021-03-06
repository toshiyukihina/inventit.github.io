---
sitemap:
 priority: 0.6
 changefreq: weekly
 lastmod: 2013-09-28T00:00:00
name: moat-iot-model-descriptor.html
title: "Inventit Iot developer Network | References | MOAT IoT Model Descriptor"
layout: references
breadcrumbs:
-
 name: References
 url: /references.html
-
 name: MOAT IoT Model Descriptor
---
# MOAT IoT Model Descriptor

## What is it?

MOAT IoT Model Descriptor is a text data to describe your models, what kind of attributes and commands they have.

This descriptor is used in MOAT js, MOAT Java and MOAT C. And it should be included in a package.json of each application package. You have to use the same model descriptors between them since all these APIs handle originally the same data models.

The API specific limitations are illustrated below.
        
### MOAT js

You can create `ModelMapper`s and `ModelStub`s for the declared model types in the Model Descriptor embedded in a package.json associated with the running js application package.

Other model types than that cannot be instantiated. You may receive a runtime exception.

### MOAT Java/Android

You need to care about the use of command methods and POJO fields declarations so that the POJO class doesn't conflict with the descriptor.

The wrong class declaration causes a runtime exception.

### MOAT C

You need to care about the use of command functions and struct declarations so that they don't conflict with the descriptor.

The wrong function declaration causes a system error (and is sent to a server as `COMMAND_FAILED`).

## Format

The following example shows a typical model descriptor.

```json
"MyModel" : {
  "array" : true,
  "shared" : false,
  "attributes" : {
    "my16bitIntField" : {"type" : "int16"},
    "my32BitIntField" : {"type" : "int32"},
    "my64BitIntField" : {
      "type" : "int64",
    },
    "myBooleanField" : {"type" : "boolean"},
    "mySingleFloatField" : {"type" : "float"},
    "myDoubleFloatField" : {"type" : "double"},
    "myStringField" : {"type" : "string"},
    "myBinaryField" : {"type" : "binary"},
    "myResourceField" : {"type" : "resource"},
  },
  "commands" : {
    "myCommandName" : {"paramType" : "string"}
  }  
}
```

### Model Meta Data Annotations

 * `array` is a boolean value to determine whether or not the model represents a collection type object. `false` by default
 * `shared` tells whether or not the model is shared between devices. `array` attribute will be ignored if the annotation is set to `true`. `false` by default.

### Shared Models vs Non-shared Models

The difference between shared models and non-shared models is whether the model objects are shared between devices or not. The shared models can be accessed (<strong>read-only</strong>) from each device via `Database.querySharedByUids()`([See here](/references/moat-js-api-document.html#ClassesDatabase)).<br/>
  On the other hand, the non-shared models cannot be used between devices. They are device-private data rather than application-wide as the shared models are.

### Attribute Data Types

All integers are SIGNED.

 *  `int64` will be handled as `int32` on C-based application depending on the underlying architecture
 *  `binary` can be typically translated into `byte[]` or `unsigned char` array. The data is represented in Base64 text on a cloud database. The max array length depends on the total size of the data (<strong>300</strong>) Primitive arrays are not supported as a transfer type except for `binary` type
 * `resource`&nbsp;is expected to be used for a large binary data like an image file. The value has the following JSON text for instance:<br />

```json
{
  ...(snip)...
  "myResourceField" : {
    "get" : "https://my.resource.url?with=parameter",
    "put" : "https://my.resource.url?with=parameter",
    "type" : "sandbox.type.net"
  },
  ...(snip)...
}
```

Clients are able to issue an HTTP request with appropriate URL value(s) if any. The keys of the URLs correspond to HTTP methods available.

These string values are always lower case.

The `type` property is a URL provider type, which is an underlying storage service identifier. The sandbox environment provides Amazon S3 backed storage identified by `s3crs.service-sync.com`.

The field value can be accessible only from a remote device or a remote REST client rather than MOAT js scripts.

### Implementation Specific Info
<div class="alert alert-info">
<strong>s3crs.service-sync.com:</strong><br />

The URLs provided by the type <code>s3crs.service-sync.com</code> has the following restrictions for now:<br />
<code>post</code> is not supported. Use <code>put</code> instead.<br />
<code>put</code> always requires <code>application/octet-stream</code> as <code>Content-Type</code> header when uploading. No other content type is allowed.<br />
<code>head</code> returns <code>ETag</code> as a <code>Content-MD5</code> value. You can check the MD5 hash of the content <code>delete</code> is available.<br />
</div>

### Commands Declaration

All function/method names called from a remote server must be declared at `commands` attribute in `package.json`. The value specifies the parameter type or `null` as a value of the `paramType` if no parameter is expected. Note that the number of parameter is always <strong>one</strong>.
