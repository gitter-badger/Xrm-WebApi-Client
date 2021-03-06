# Dynamics CRM JavaScript Web API Client
[![Build Status](https://travis-ci.org/DigitalFlow/Xrm-WebApi-Client.svg?branch=master)](https://travis-ci.org/DigitalFlow/Xrm-WebApi-Client) [![Coverage Status](https://coveralls.io/repos/github/DigitalFlow/Xrm-WebApi-Client/badge.svg?branch=master)](https://coveralls.io/github/DigitalFlow/Xrm-WebApi-Client?branch=master)

This is a framework for easing working with the Dynamics CRM WebApi using JavaScript.
It uses the awesome [BlueBird](https://github.com/petkaantonov/bluebird) framework for handling requests asynchronously based on promises.
The framework is supposed to be executed on CRM forms or on CRM web ressources, where the CRM context is available.
For running from custom web resources, be sure that the GetGlobalContext function is available, as the client will try to retrieve the context on its own.

# Index

- [Dynamics CRM JavaScript Web API Client](#dynamics-crm-javascript-web-api-client)
  * [Requirements](#requirements)
    + [CRM](#crm)
    + [Browser](#browser)
  * [How to obtain it](#how-to-obtain-it)
    + [NPM](#npm)
    + [GitHub Release](#github-release)
  * [How to build it](#how-to-build-it)
  * [Operations](#operations)
    + [Synchronous vs Asynchronous](#synchronous-vs-asynchronous)
    + [Create](#create)
      - [Return created record in create response](#return-created-record-in-create-response)
    + [Retrieve](#retrieve)
      - [Retrieve single records](#retrieve-single-records)
        * [Retrieve by ID](#retrieve-by-id)
        * [Retrieve by alternate key](#retrieve-by-alternate-key)
      - [Retrieve multiple records](#retrieve-multiple-records)
        * [Retrieve by query expression](#retrieve-by-query-expression)
        * [Retrieve by FetchXml](#retrieve-by-fetchxml)
      - [Auto expand collection-valued navigation properties](#auto-expand-collection-valued-navigation-properties)
    + [Update](#update)
      - [Update by alternate key](#update-by-alternate-key)
      - [Return updated record in update response](#return-updated-record-in-update-response)
    + [Delete](#delete)
      - [Delete by alternate key](#delete-by-alternate-key)
    + [Associate](#associate)
    + [Disassociate](#disassociate)
    + [Execute](#execute)
      - [No parameter request](#no-parameter-request)
      - [Parametrized request](#parametrized-request)
    + [Configuration](#configuration)
    + [Errors](#errors)
    + [Set Names](#set-names)
    + [Not yet implemented requests](#not-yet-implemented-requests)
    + [Promises](#promises)
  * [Headers](#headers)
    + [Header Format](#header-format)
    + [Default Headers](#default-headers)
    + [Request Headers](#request-headers)
      - [Page size](#page-size)
    + [API Version](#api-version)
  * [Remarks](#remarks)
    + [CRM App](#crm-app)

## Requirements
### CRM
This framework targets the Dynamics CRM WebApi, therefore CRM 2016 (>= v8.0) is needed.

### Browser
Although using Promises, some legacy browsers are still supported, since bluebird is used as Promise polyfill.
Bluebird is automatically included in the bundled release, no additional steps required.
For a list of supported browsers, check the bluebird [platform support](http://bluebirdjs.com/docs/install.html#supported-platforms).

## How to obtain it
### NPM
This framework is on npm as UMD, thanks to the standalone option of browserify.

The package name is xrm-webapi-client, check it out:

[![NPM version](https://img.shields.io/npm/v/xrm-webapi-client.svg?style=flat)](https://www.npmjs.com/package/xrm-webapi-client)

### GitHub Release
You can always download the browserified version of this framework by downloading the release.zip file from the latest [release](https://github.com/DigitalFlow/Xrm-WebApi-Client/releases).

## How to build it
You'll have to install [npm](https://www.npmjs.com/) on your machine.

For bootstrapping, simply run ```npm install``` once initially.
For every build, you can just call ```npm run build```. You'll find the build output in the Publish directory.

## Operations
### Synchronous vs Asynchronous
Per default, all requests are sent asynchronously.
This is the suggested way of sending requests, however, sometimes there is the need for using synchronous requests.

For example if you want to use a function for hiding / enabling a ribbon bar button, it has to return either true or false for the visibility of the button. In this case, you would need to use a synchronous request for being able to directly return values.

Be sure to avoid synchronous requests if it is possible and use asynchronous requests instead.

For sending requests synchronously, you can either set ```WebApiClient.Async``` to false, which will configure the WebApiClient to send all requests synchronously, or pass an ```async``` property in your request, like so:

```JavaScript
var request = {
    entityName: "account",
    entity: {name: "Adventure Works"},
    async: false
};

try {
    var response = WebApiClient.Create(request);

    // Process response
}
catch (error) {
    // Handle error
}
```

### Create
The client supports creation of records. You have to pass the entity logical name, and a data object:

```JavaScript
var request = {
    entityName: "account",
    entity: {name: "Adventure Works"}
};

WebApiClient.Create(request)
    .then(function(response){
        // Process response
    })
    .catch(function(error) {
        // Handle error
    });
```

#### Return created record in create response
This feature is available from Dynamics365 v8.2 upwards.
For returning the full record that was created from your request, set an appropriate Prefer header as follows:

```JavaScript
var request = {
    entityName: "account",
    entity: {name: "Adventure Works"},
    headers: [{key: "Prefer", value: "return=representation"}]
};

WebApiClient.Create(request)
    .then(function(response){
        // Process response
    })
    .catch(function(error) {
        // Handle error
    });
```

### Retrieve
The client supports retrieving of records by Id, by alternate key, fetchXml and query expressions.
For retrieving by alternate key, pass an array of objects that each have a property and a value property.
You have to pass at least the entity logical name.
You can always pass query parameters which will be appended to your retrieve requests.

#### Retrieve single records
##### Retrieve by ID

```JavaScript
var request = {
    entityName: "account",
    entityId: "00000000-0000-0000-0000-000000000001"
};

WebApiClient.Retrieve(request)
    .then(function(response){
        // Process response
    })
    .catch(function(error) {
        // Handle error
    });
```

##### Retrieve by alternate key

```JavaScript
var request = {
    entityName: "contact",
    alternateKey:
        [
            { property: "firstname", value: "Joe" },
            { property: "emailaddress1", value: "abc@example.com"}
        ]
};

WebApiClient.Retrieve(request)
    .then(function(response){
        // Process response
    })
    .catch(function(error) {
        // Handle error
    });
```

#### Retrieve multiple records
Retrieve of multiple records uses paging. Per default you can set a [page size on your requests](#page-size), however this is limited to 5000 records.
If you want to really retrieve all records, set WebApiClient.ReturnAllPages to true, as it is by default false, like this:

``` JavaScript
WebApiClient.ReturnAllPages = true;
```

By setting this to true, each retrieve multiple request will check for an @odata.nextLink property inside the response, call the next page and concatenate the results, until all records have been retrieved.

You can also pass this option per-request, like this:

```JavaScript
var request = {
    entityName: "account",
    queryParams: "?$select=name,revenue,&$orderby=revenue asc,name desc&$filter=revenue ne null",
    returnAllPages: true
};
```

##### Retrieve by query expression

```JavaScript
var request = {
    entityName: "account",
    queryParams: "?$select=name,revenue,&$orderby=revenue asc,name desc&$filter=revenue ne null"
};

WebApiClient.Retrieve(request)
    .then(function(response){
        // Process response
    })
    .catch(function(error) {
        // Handle error
    });
```

##### Retrieve by FetchXml
FetchXml requests have some special behaviour implemented. Short fetchXml will be sent as GET request using a fetchXml URL query parameter.
There is however an URL length limit of 2048 chars, so large fetchXml requests would fail, since they exceed this limit.
Since release v3.1.0, the request will automatically be sent as POST batch request, so that large fetchXml can be executed as well.
You don't have to do anything for this to happen, the URL length is checked automatically before sending the request.

```JavaScript
var request = {
    entityName: "account",
    fetchXml: "<fetch mapping='logical'>" +
                "<entity name='account'>" +
                    "<attribute name='accountid'/>" +
                    "<attribute name='name'/>" +
                "</entity>" +
              "</fetch>"
};

WebApiClient.Retrieve(request)
    .then(function(response){
        // Process response
    })
    .catch(function(error) {
        // Handle error
    });
```

#### Auto expand collection-valued navigation properties
When retrieving collection-valued navigation properties, the expand is being deferred, i.e. you don't retrieve immediate results, but a property ending in "@odata.nextLink" that contains an URL to the results for this expand. You can read more about this [here](https://msdn.microsoft.com/en-us/library/gg334767.aspx#bkmk_expandRelated).
For easing to retrieve these, we can use the `WebApiClient.Expand` function. It takes an array of records and expands all properties, that end in "@odata.nextLink".
You can additionally pass headers to the request, that will be appended to each retrieve request for properties.

```JavaScript
WebApiClient.Retrieve({
    entityName: "account",
    queryParams: "?$expand=contact_customer_accounts"
})
.then(function(response){
    return WebApiClient.Expand({
        records: response.value
    });
})
.then(function(response){        
    // Process response
})
.catch(function(error) {
    // Handle error
});
```

### Update
Update requests are supported. You have to pass the entity logical name, the ID of the record to update and an update object:

```JavaScript
var request = {
    entityName: "account",
    entityId: "00000000-0000-0000-0000-000000000001",
    entity: { name: "Contoso" }
};

WebApiClient.Update(request)
    .then(function(response){
        // Process response
    })
    .catch(function(error) {
        // Handle error
    });
```

#### Update by alternate key

```JavaScript
var request = {
    entityName: "contact",
    alternateKey:
        [
            { property: "firstname", value: "Joe" },
            { property: "emailaddress1", value: "abc@example.com"}
        ],
    entity: { lastname: "Doe" }
};

WebApiClient.Update(request)
    .then(function(response){
        // Process response
    })
    .catch(function(error) {
        // Handle error
    });
```

#### Return updated record in update response
This feature is available from Dynamics365 v8.2 upwards.
For returning the full record after applying the updates from your request, set an appropriate Prefer header as follows:

```JavaScript
var request = {
    entityName: "account",
    entityId: "00000000-0000-0000-0000-000000000001",
    entity: { name: "Contoso" },
    headers: [{key: "Prefer", value: "return=representation"}]
};

WebApiClient.Update(request)
    .then(function(response){
        // Process response
    })
    .catch(function(error) {
        // Handle error
    });
```

### Delete
Delete requests are supported. You have to pass the entity logical name, and ID of the record to delete:

```JavaScript
var request = {
    entityName: "account",
    entityId: "00000000-0000-0000-0000-000000000001"
};

WebApiClient.Delete(request)
    .then(function(response){
        // Process response
    })
    .catch(function(error) {
        // Handle error
    });
```

#### Delete by alternate key

```JavaScript
var request = {
    entityName: "contact",
    alternateKey:
        [
            { property: "firstname", value: "Joe" },
            { property: "emailaddress1", value: "abc@example.com"}
        ]
};

WebApiClient.Delete(request)
    .then(function(response){
        // Process response
    })
    .catch(function(error) {
        // Handle error
    });
```

### Associate
Associate requests are supported. You have to pass the relationship name, a source and a target entity.
This example associates an opportuntiy to an account:

```JavaScript
var request = {
    relationShip: "opportunity_customer_accounts",
    source:
        {
            entityName: "opportunity",
            entityId: "00000000-0000-0000-0000-000000000001"
        },
    target:
        {
            entityName: "account",
            entityId: "00000000-0000-0000-0000-000000000002"
        }
};

WebApiClient.Associate(request)
    .then(function(response){
        // Process response
    })
    .catch(function(error) {
        // Handle error
    });
```

### Disassociate
Disassociate requests are supported. You have to pass the relationship name, a source and a target entity.
This example disassociates an opportuntiy from an account:

```JavaScript
var request = {
    relationShip: "opportunity_customer_accounts",
    source:
        {
            entityName: "opportunity",
            entityId: "00000000-0000-0000-0000-000000000001"
        },
    target:
        {
            entityName: "account",
            entityId: "00000000-0000-0000-0000-000000000002"
        }
};

WebApiClient.Disassociate(request)
    .then(function(response){
        // Process response
    })
    .catch(function(error) {
        // Handle error
    });
```

### Execute
There is support for executing actions / functions without having to use SendRequest.
The WebApiClient has a function WebApiClient.Execute, which takes a request as parameter.
Requests are objects that base on the WebApiClient.Requests.Request base request.
When wanting to send an already implemented request using Execute, you can either use the blank request (such as the WhoAmIRequest, that does not need any parameters), or in case it needs parameters, extend an existing request.

Missing or custom action requests can be implemented as described [here](#not-yet-implemented-requests).

Check the [wiki](https://github.com/DigitalFlow/Xrm-WebApi-Client/wiki) for a list of requests that are implemented in the current release and examples on how to send them!

#### No parameter request
The WhoAmI request does not need any parameters, therefore we can just pass the blank request:

```JavaScript
var request = WebApiClient.Requests.WhoAmIRequest;

WebApiClient.Execute(request)
    .then(function(response){
        // Process response
    })
    .catch(function(error) {
        // Handle error
    });
```

#### Parametrized request
Most requests need further parameters for being sent.
When needing to send those requests, start with the blank request and call the function "with" on it, passing the needed parameters as object to it. Your passed-in parameters override possibly existing parameters with the same name.

The following parameters are supported:
- method - HTTP method for request (Required, but defined by request)
- name - Name of the request as used for the URL (Required, but defined by request)
- bound - Pass true if request is bound to a record, false if not. Has consequences for automatic URL building. By default false and defined by request.
- entityName - Name of the request's target entity. Defined by request if always the same.
- entityId - ID of the request's target record
- payload - Object that is sent as payload for the request
- headers - Headers that should be set on the request
- urlParams - Any parameters that have to be embedded in the request URL, as described [here](https://msdn.microsoft.com/en-us/library/gg309638.aspx#Anchor_2). Pass an object with parameter names as keys and the corresponding values.

Sample request for AddToQueue:
```JavaScript
var request = WebApiClient.Requests.AddToQueueRequest
    .with({
        entityId: "56ae8258-4878-e511-80d4-00155d2a68d1",
        payload: {
            Target: {
                activityid: "59ae8258-4878-e511-80d4-00155d2a68d1",
                "@odata.type": "Microsoft.Dynamics.CRM.letter"
            }
        }
    });

WebApiClient.Execute(request)
    .then(function(response){
        // Process response
    })
    .catch(function(error) {
        // Handle error
    });
```

### Configuration
When having to set multiple configuration settings for the WebApiClient, you can use the ```Configure``` function, which gets an object passed with keys and values, that get projected onto the WebApiClient:

```JavaScript
WebApiClient.Configure({
    ApiVersion: "8.2",
    ReturnAllPages: true,
    PrettifyErrors: false
});
```

### Errors
If errors occur during processing of requests, the WebAPI client by default throws an error with the text that follows this format: xhr.statusText: xhr.response.message, i.e. "Internal Server Error: The function parameter 'EntityMoniker' cannot be found.Parameter name: parameterName".

For returning the whole stringified JSON response including a custom xhrStatusText property, set

```JavaScript
WebApiClient.PrettifyErrors = false;
```

### Set Names
Set names are automatically generated according to WebApi rules and based on the entityName parameter in your request.
However there are some set names, that are not generated according to naming rules, for example ContactLeads becomes contactleadscollection. For handling those corner cases, each request allows to pass an overriddenSetName instead of the entity name, so that you can directly pass those set names that break naming rules. This should happen very rarely.
Example of passing overriddenSetName:

```JavaScript
var request = {
    overriddenSetName: "contactleadscollection",
    entity: {name: "Contoso"}
};

WebApiClient.Create(request)
    .then(function(response){
        // Process response
    })
    .catch(function(error) {
        // Handle error
    });
```

### Not yet implemented requests
If you need to use requests, that are not yet implemented (such as custom actions), you can create an executor for the missing request and append it to the WebApiClient.Requests object (if you want to reuse it). Be sure to create your missing request by calling Object.create on the base request object.
This might look something like this:

```JavaScript
WebApiClient.Requests.AddToQueueRequest = WebApiClient.Requests.Request.prototype.with({
    method: "POST",
    name: "AddToQueue",
    bound: true,
    entityName: "queue"
});

```
For further explanations regarding these requests, please check [here](#execute).
All requests should be implemented basically by now, in case of any errors in the implementations, you can override any property using the ```with``` function as described [here](#execute).

Alternatively, you can use the ```WebApiClient.SendRequest``` function.
In combination with ```WebApiClient.GetApiUrl``` and ```WebApiClient.GetSetName``` you can easily build up your request url, set your HTTP method and attach additional payload or headers.

An example of a custom implementation of the WinOpportunity request:
```Javascript
var url = WebApiClient.GetApiUrl() + "WinOpportunity";
var opportunityId = "00000000-0000-0000-0000-000000000001";
var payload = {
    "Status": 3,
    "OpportunityClose": {
        "subject": "Won Opportunity",
        "opportunityid@odata.bind": "/" + WebApiClient.GetSetName("opportunity") + "(" + opportunityId + ")"
    }
};

WebApiClient.SendRequest("POST", url, payload)
    .then(function(response){
        // Process response
    })
    .catch(function(error) {
        // Handle error
    });
```

### Promises
This client uses bluebird internally for handling promises in a cross-browser compliant way.
Therefore the promises returned by all asynchronous requests are also bluebird promises.
Bluebird itself is not exported globally anymore as of v3.0.0, but can be accessed by using ```WebApiClient.Promise```.
This decision was made for not causing issues with other scripts.

Using promises you can do something like this, too:
```Javascript
var requests = [];

for (var i = 0; i < 5; i++) {
    var request = {
        entityName: "account",
        entity: {name: "Adventure Works Nr. " + i}
    };
    requests.push(WebApiClient.Create(request));
}

WebApiClient.Promise.all(requests)
    .then(function(response){
        // Process response
    })
    .catch(function(error) {
        // Handle error
    });
```

## Headers
There is a defined set of default headers, which are sent on each request, as well as per-request headers.
Per-request headers override possibly existing default headers with the same key value.

### Header Format
Headers are represented as objects containing a key and a value property:

```JavaScript
var header = { key: "headerKey", value: "headerValue" };
```

### Default Headers
By default there is a defined set of default headers, that will be sent with each request.
The default headers can be retrieved using the WebApiClient.GetDefaultHeaders function.
You can however add own default headers by using the WebApiClient.AppendToDefaultHeaders function, which takes as much headers as dynamic arguments as you like.

Example:
```JavaScript
var header = {key: "newHeader", value: "newValue"};
WebApiClient.AppendToDefaultHeaders (header);
```

### Request Headers
You can also attach headers per request, all request parameters have a headers property, that can be used for passing per-request headers.

This could look something like this:
``` JavaScript
// Parameters for create request
var request = {
    entityName: "account",
    entity: {name: "Adventure Works"},
    headers: [ { key: "headerKey", value: "headerValue" }]
};
```

#### Page size
If you want to set a max page size for your request (supported are up to 5000 records per page), you can pass the following header:

``` JavaScript
headers: [ { key: "Prefer", value: "odata.maxpagesize=5000" }]
```

### API Version
The default API version is 8.0.
You can however change it to 8.1 if needed by using

```JavaScript
WebApiClient.ApiVersion = "8.1";
```

## Remarks
### CRM App
For using WebApiClient with the CRM App, you'll have to use the normal (= not uglified) version.
When using uglified JS in the CRM App, you might receive invalid character errors. This is not only valid for the WebApiClient, but also for some other uglified code.
