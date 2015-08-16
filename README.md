# Brinkbit HTTP Filesystem API
> A generalized RESTful API for managing a remote file system

# Table of Contents

1. [Overview](#overview)
1. [Resources](#resources)
1. [Request](#request)
1. [Response](#response)
1. [Actions](#actions)
  1. [Get](#get)
    1. [Read](#1.1)
    1. [Help](#1.2)
    1. [Search](#1.3)
    1. [Inspect](#1.4)
  1. [Post](#post)
    1. [Create](#2.1)
    1. [Copy](#2.2)
  1. [Put](#post)
    1. [Update](#3.1)
    1. [Move](#3.2)
    1. [Rename](#3.3)
  1. [Delete](#delete)
    1. [Delete](#4.1)

**[⬆ back to top](#table-of-contents)**

# Overview

This API is intended to be the standard for communication between the client and server for the filesystem component of the Brinkbit IDE.
While our use case is specific, we hope to develop this as a generalized implementation-agnostic standard for managing remote file systems via HTTP requests.

The http-fs-api follows the [RESTful API](https://en.wikipedia.org/wiki/Representational_state_transfer) systems architecture style.
What this means in practice is that all actions are mapped to four [HTTP methods](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) (POST, GET, PUT, DELETE) which in turn represent the four [CRUD commands](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) (Create, Read, Update, Delete).
The url requested determines what resource will be manipulated.

While the API is designed to handle paths, directories, and files as though the backend were an actual file system, it's important to note that your backend implementation does not need to be an actual filesystem.
In fact, we recommend that you not store the manipulated resources on an actual filesystem but use some other form of data storage i.e. a database or S3 Buckets.

All examples contained in this document are written as [ajax requests](http://api.jquery.com/jquery.ajax/) and use the fictitious `http://cats.com` as the domain.
In all the examples, `/fs/` will be the route on which the file system API is defined.

**[⬆ back to top](#table-of-contents)**

# Resources

Our filesystem has two types of resources, files and directories.
Both can be manipulated with the same actions, but each behave a little differently.

TODO: expand explanation

**[⬆ back to top](#table-of-contents)**

# Request

Requests are made on a uri with one of the following four [HTTP methods](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html): "GET", "POST", "PUT", "DELETE".

A uri represents the resource to take action on.
For example the uri `/img.png`

Every request accepts an optional JSON object as a request parameter.

Every request can optionally specify an action field in the request parameters. For example:

```javascript
$.ajax({
  url: 'http://cats.com/fs/Siamese.img'
  method: 'PUT',
  data: {
    action: 'Move',
    destination: 'Siberian.img'
  }
})
```
**[⬆ back to top](#table-of-contents)**

# Response

TODO: outline what a generalized response looks like

**[⬆ back to top](#table-of-contents)**

## Actions

TODO: explain what actions are

### Get

- [1.1](#1.1) <a name='1.1'></a> **Read** *default*
  > Request file or directory contents.
    Default GET action

  Parameters
  > *none*

  File:
  ```javascript
  // request
  $.ajax({
    url: 'http://cats.com/fs/Siamese.img'
  });

  // response is the image data
  ```

  Directory:
  ```javascript
  // request
  $.ajax({
    url: 'http://cats.com/fs/prettyCats/'
  });

  // response
  {
    code: 200,
    data: [
      'kindof_pretty_cat.png',
      'very_pretty_cat.png',
      'morePrettyCats/' // a directory, signified by the trailing /
    ]
  }
  ```
  Errors
  + `404` - Invalid path / Resource does not exist


- [1.2](#1.2) <a name='1.2'></a> **Help**
  > Request detailed information for a given HTTP method

  Parameters
  + `method` `string` *optional* -
    an http method i.e. Get, Put, Post, or Delete

  Returns a raw text help message
  ```javascript
  // request
  $.ajax({
    url: 'http://cats.com/fs/Siamese.img',
    data: {
      action: 'Help',
      action: 'Open'
    }
  });

  // response
  {
    code: 200,
    data: "Returns file and directory contents. Default GET action.\nParameters: none\n"
  }
  ```

  Errors
  + `404` - Invalid path / Resource does not exist


- [1.3](#1.3) <a name='1.3'></a> **Search**
  > Run a query on the requested resource.
    Note, only valid on directories

  Parameters
  + `query` `regex` *optional* -
    a regular expression to run against the requested directory
  + `type` `string` *optional* -
    filter by resource type, either 'directory' or 'file'
  + `range` `integer array` *optional* -
    `[ from, to ]` - the range of depths to query.
    Zero is the current working directory.

  Returns an array of matching resources:
  ```javascript
  // request
  $.ajax({
    url: 'http://cats.com/fs/',
    data: {
      query: '/cat/gi'
    }
  });

  // response
  {
    code: 200,
    data: [
      'prettyCats/',
      'prettyCats/kindof_pretty_cat.png',
      'prettyCats/very_pretty_cat.png',
      'prettyCats/morePrettyCats/',
      'ugly/ugly_cat.jpg',
      'catcatcat.jpg'
    ]
  }
  ```
  Request only directories:
  ```javascript
  // request
  $.ajax({
    url: 'http://cats.com/fs/',
    data: {
      query: '/cat/gi',
      type: 'directory'
    }
  });

  // response
  {
    code: 200,
    data: [
      'prettyCats/',
      'prettyCats/morePrettyCats/',
    ]
  }
  ```
  Request all child resources one level deep:
  ```javascript
  // request
  $.ajax({
    url: 'http://cats.com/fs/',
    data: {
      depth: [1, 1] // from level one to level one
    }
  });

  // response
  {
    code: 200,
    data: [
      'prettyCats/kindof_pretty_cat.png',
      'prettyCats/very_pretty_cat.png',
      'prettyCats/morePrettyCats/',
      'ugly/ugly_cat.jpg',
      'ugly/wombat.png'
    ]
  }
  ```

  Errors
  + `404` - Invalid path / Resource does not exist
  + `501` - Invalid action / Action-Resource type conflict


- [1.4](#1.4) <a name='1.4'></a> **Inspect**
  > Request detailed information about a resource

  Parameters
  + `fields` `array` -
    an array of strings for each requested field

  TODO: examples

  Errors
  + `404` - Invalid path / Resource does not exist

**[⬆ back to top](#table-of-contents)**

### Post

- [2.1](#2.1) <a name='2.1'></a> **Create** *default*
  > Create a resource with optional initial data.
    Default POST action.

  Parameters
  + `data` `FormData` -
    the initial data to store in the resource

  Returns
  ```javascript
  // request
  $.ajax({
    url: 'http://cats.com/fs/mycats/Fluffy.img',
    type: 'POST',
    data: formdata,
    processData: false,
    contentType: false
  });

  // response
  {
    code: 200
  }
  ```

  Errors
  + `409` - Invalid path / Resource already exists
  + `415` - Invalid file type
  + `413` - Request data too large


- [2.2](#2.2) <a name='2.2'></a> **Copy**
  > Copy a resource

  Parameters
  + `destination` `string` -
    the full path where the copy should be created

  TODO: examples

  Errors
  + Invalid path / Resource does not exist (404)
  + Invalid destination / Resource already exists (409)


**[⬆ back to top](#table-of-contents)**

### Put

- [3.1](#3.1) <a name='3.1'></a> **Update** *default*
  > Modify an existing resource.
    Default PUT action

  Parameters
  + `data` `FormData` -
    the data to store in the resource

  TODO: examples

  Errors
  + `404` - Invalid path / Resource does not exist
  + `413` - Request data too large



- [3.2](#3.2) <a name='3.2'></a> **Move**
  > Relocate an existing resource

  Parameters
  + `destination` `string` -
    the path to which the resource will be moved

  TODO: examples
  Errors
  + `404` - Invalid path / Resource does not exist
  + `409` - Invalid destination / Resource already exists

- [3.3](#3.3) <a name='3.3'></a> **Rename**
  > Rename an existing resource

  Parameters
  + TODO: outline parameters

  TODO: examples
  TODO: outline errors

**[⬆ back to top](#table-of-contents)**

### Delete

- [4.1](#4.1) <a name='4.1'></a> **Delete**
  > Destroy an existing resource

  Parameters
  > none

  TODO: examples

  Errors
  + `404` - Invalid path / Resource does not exist



**[⬆ back to top](#table-of-contents)**

# TODOs

- mention resources mapping to URIs
- mention json data type
- describe HTTP commands
- outline response format
- outline request format
- outline what an action is (i.e. Open, Save, Copy)
- outline the different actions and their HTTP command mappings
- outline how to contribute
