[![Build Status](https://travis-ci.org/guylabs/angular-spring-data-rest.svg?branch=master)](https://travis-ci.org/guylabs/angular-spring-data-rest)
[![Coverage Status](https://img.shields.io/coveralls/guylabs/angular-spring-data-rest.svg)](https://coveralls.io/r/guylabs/angular-spring-data-rest)
[![Bower version](https://badge.fury.io/bo/angular-spring-data-rest.svg)](http://badge.fury.io/bo/angular-spring-data-rest)
[![npm version](https://badge.fury.io/js/angular-spring-data-rest.svg)](http://badge.fury.io/js/angular-spring-data-rest)

# angular-spring-data-rest

> An AngularJS module with an additional interceptor which eases the work with a [Spring Data REST](http://projects.spring.io/spring-data-rest) backend.

## See how it works

If you want to see the module in action, then check out the sample application which runs with Spring Boot and is super easy to setup: [angular-spring-data-rest-sample](https://github.com/guylabs/angular-spring-data-rest-sample)

#Table of contents

- [Quick start](#quick-start)
- [Overview](#overview)
- [The `SpringDataRestAdapter`](#the-springdatarestadapter)
    - [Usage of `SpringDataRestAdapter`](#usage-of-springdatarestadapter)
    - [Usage of `_resources` method](#usage-of-_resources-method)
        - [The `_resources` method parameters and return type](#the-_resources-method-parameters-and-return-type)
        - [`_resources` usage example](#_resources-usage-example)
        - [Exchange the underlying Angular `$resource` function](#exchange-the-underlying-angular-resource-function)
    - [Usage of `_embeddedItems` property](#usage-of-_embeddeditems-property)
        - [`_embeddedItems` usage example](#_embeddeditems-usage-example)
    - [How to automatically fetch links](#how-to-automatically-fetch-links)
        - [Fetch multiple or all links](#fetch-multiple-or-all-links)
        - [Exchange the underlying fetch function](#exchange-the-underlying-fetch-function)
        - [The fetch method parameters](#the-fetch-method-parameters)
    - [How to use `SpringDataRestAdapter` with promises](#how-to-use-springdatarestadapter-with-promises)
    - [Configuration of the `SpringDataRestAdapter`](#configuration-of-the-springdatarestadapter)
- [The `SpringDataRestInterceptor`](#the-springdatarestinterceptor)
- [Dependencies](#dependencies)
- [Release notes](#release-notes)
- [Acknowledgements](#acknowledgements)
- [License](#license)


## Quick start

To use the `SpringDataRestAdapter` *Angular* module with [bower](http://bower.io) execute the following command to install the package:

```bash
bower install angular-spring-data-rest
```
or with [npm](https://www.npmjs.com/):
```bash
npm install angular-spring-data-rest
```

## Overview

*Spring Data REST* integrates [Spring HATEOAS](http://projects.spring.io/spring-hateoas) by default. This simplifies the creation of REST presentations which are generated with the [HATEOAS](http://en.wikipedia.org/wiki/HATEOAS) principle.

Therefore the *Spring Data REST* responses have resources and embedded resources like in the following JSON response example:

```json
{
    "_links": {
        "self": {
            "href": "http://localhost:8080/categories{?page,size,sort}",
            "templated": true
        }
    },
    "_embedded": {
        "categories": [
            {
                "name": "Test category 1",
                "_links": {
                    "self": {
                        "href": "http://localhost:8080/categories/1"
                    },
                    "parentCategory": {
                        "href": "http://localhost:8080/categories/1/parentCategory"
                    }
                }
            },
            {
                "name": "Test category 2",
                "_links": {
                    "self": {
                        "href": "http://localhost:8080/categories/2"
                    },
                    "parentCategory": {
                        "href": "http://localhost:8080/categories/2/parentCategory"
                    }
                }
            }
        ]
    },
    "page": {
        "size": 20,
        "totalElements": 2,
        "totalPages": 1,
        "number": 0
    }
}
```
The above response is the result of calling the endpoint to get all categories(`http://localhost:8080/categories`). It contains the `_links` property which holds an object with several named resources(e.g. `self`). These resources are used to navigate to a related entity(e.g. `parentCategory`) or to a specific endpoint defined by the backend.

The `_embedded` property holds an array of the requested items, and each of the items has again a `_links` property which defines the resources for the specific item. You can find more about how the responses of *Spring Data REST* look like [here](http://projects.spring.io/spring-data-rest).

This *Angular* module provides two ways of processing a response from the *Spring Data REST* backend and ease the usage with the resources and embedded items:

1. Using an instance of the `SpringDataRestAdapter`.
2. Add the `SpringDataRestInterceptor` to the Angular `$httpProvider.interceptors` such that all responses are processed.

## The `SpringDataRestAdapter`

The `spring-data-rest` *Angular* module provides a provider for the `SpringDataRestAdapter` object. This object is the core of the module and it processes a given response and adds the following additional properties/methods to it:

1. `_resources`: this method wraps the *Angular* `$resource` function by default (this is [exchangeable](#exchange-the-underlying-angular-resource-function)) and adds an easy way to retrieve the resources defined in the `_links` property. It is also used to retrieve all available resources of the given object. Read more about this method [here](#usage-of-_resources-method).
2. `_embeddedItems`: this property replaces the `_embedded` property and sets the named array (`categories` in the upper example response) with the embedded items as its value. Read more about this property [here](#usage-of-_embeddedItems-property).

Spring Data REST also generates an index response when you make a `GET` response to the configured base url of the dispatcher servlet. This response looks like the following example:

```javascript
{
  "_links" : {
    "users" : {
      "href" : "http://localhost:8080/users{?page,size,sort}",
      "templated" : true
    },
    "categories" : {
      "href" : "http://localhost:8080/categories{?page,size,sort}",
      "templated" : true
    },
    "accounts" : {
      "href" : "http://localhost:8080/accounts{?page,size,sort}",
      "templated" : true
    },
    ...
  }
}
```

This response shows all configured Spring Data REST repositories and the links to these resources. The `SpringDataRestAdapter` is also able to handle this response and to provide an easy method to retrieve all the available resources with the `_resources` method. Please read more about this [here](#the-_resources-method-parameters-and-return-type).

Another feature is that the `SpringDataRestAdapter` is able to automatically fetch the links of a resource and adds the response of the link as a property to the main response. Please read more about this [here](#how-to-automatically-fetch-links).

### Usage of `SpringDataRestAdapter`

To use the `SpringDataRestAdapter` object you need to include the `angular-spring-data-rest.js` file (or the minified version) and add the `spring-data-rest` module as a dependency in your module declaration:

```javascript
var myApp = angular.module("myApplication", ["ngResource", "spring-data-rest"]);
```

Now you are able use the `SpringDataRestAdapter` object and process a given response and you will get a promise back which resolves with the processes response:

```javascript
SpringDataRestAdapter.process(response).then(function(processedResponse) {
  ...
});
```

The response can be a `String` or a promise from an `$http.get()` call (or any other promise which resolves the response) like in the following example:

```javascript
var httpPromise = $http.get('/rest/categories');

SpringDataRestAdapter.process(httpPromise).then(function (processedResponse) {
    $scope.categories = processedResponse._embeddedItems;
});
```

For simplicity the rest of the examples assume that you already have the response present. Please read on on how to use the `_resources` method and the `_embeddedItems` property to ease the handling of resources and embedded items.

### Usage of `_resources` method

The `_resources` property is added on the same level of the JSON response object where a `_links` property exists. When for example the following JSON response object is given:

```javascript
var response = {
    "_links": {
        "self": {
            "href": "http://localhost:8080/categories{?page,size,sort}",
            "templated": true
        }
    },
    "_embedded": {
        ...
    }
    ...
}
```

Then the `SpringDataRestAdapter` will add the `_resources` method to the same level such that you can call it the following way (inside the `then` function of the promise):

```javascript
SpringDataRestAdapter.process(response).then(function(processedResponse) {
  processedResponse._resources(linkName, paramDefaults, actions, options);
});
```

This `_resources` method is added recursively to all the properties of the JSON response object where a `_links` property exists.

#### The `_resources` method parameters and return type

The `_resources` method takes the following four parameters:

* `linkName`: the name of the link's `href` you want to call with the underlying *Angular* `$resource` function. You can also pass in a resource object with parameters in the following way:

```javascript
SpringDataRestAdapter.process(response).then(function(processedResponse) {
  var resourceObject = {
    "name": "self",
    "parameters": {
        "size": 20,
        "sort": "asc"
    }
  }
  processedResponse._resources(resourceObject, paramDefaults, actions, options);
});
```

This will call *Angular* `$resource` method by default (this is [exchangeable](#exchange-the-underlying-angular-resource-function)) with the `href` of the `self` resource and will add the parameters `size` and `sort` as query string to the URL. If the resource object parameters and the `paramDefaults` parameters are set, then these two objects are merged such that the resource object parameters appear first in the new object and the `paramDefaults` parameters last.

* `paramDefaults`: the default values for url parameters. Read more  [here](https://docs.angularjs.org/api/ngResource/service/$resource).
* `actions`: custom action that should extend the default set of the `$resource` actions. Read more [here](https://docs.angularjs.org/api/ngResource/service/$resource).
* `options`: custom settings that should extend the default `$resourceProvider` behavior Read more [here](https://docs.angularjs.org/api/ngResource/service/$resource).

You are also able to add URL templates to the passed in `linkName` parameter. (This just works if you pass a string and not a resource object) The following example explains the usage:

```javascript
SpringDataRestAdapter.process(response).then(function(processedResponse) {
  var resources = processedResponse._resources("self/:id", {id: "@id"});
  var item = resources.get({id: 1}).then(function(data) {
    item = data;
  };
});
```

The `_resources` method returns the *Angular* resource "class" object with methods for the default set of resource actions. Read more [here](https://docs.angularjs.org/api/ngResource/service/$resource).

If no parameter is given the `_resources` method will return all available resources objects of the given object. When for example the following JSON response object is given:

```javascript
var response = {
    "_links": {
        "self": {
            "href": "http://localhost:8080/categories{?page,size,sort}",
            "templated": true
        },
        "parentCategory": {
            "href": "http://localhost:8080/categories/1/parentCategory"
        }
    },
    "_embedded": {
        ...
    }
    ...
}
```
Then the following call to the `_resources` method without any parameter will return an array of all available resource objects.

```javascript
SpringDataRestAdapter.process(response).then(function(processedResponse) {
  var availableResources = processedResponse._resources();
});
```

The above call will result in the following return value:

```json
[
    {
        "name":"self",
        "parameters": {
            "page": "",
            "size": "",
            "sort": ""
        }
    },
    {
        "name":"parentCategory"
    }
]
```

This functionality is useful if you want to first check all available resources before using the `_resources` method to retrieve the specific resource.

#### `_resources` usage example

This example refers to the JSON response in the [Overview](#overview). If you want to get the parent category of a category you would call the `_resources` method the following way:

```javascript
SpringDataRestAdapter.process(response).then(function(processedResponse) {
  var parentCategoryResource = processedResponse._embeddedItems[0]._resources("parentCategory");

  // create a GET request, with the help of the Angular resource class, to the parent category
  // url and log the response to the console
  var parentCategory = parentCategoryResource.get(function() {
    console.log("Parent category name: " + parentCategory.name);
  });
});
```

#### Exchange the underlying Angular `$resource` function

If you want to exchange the underlying call to the *Angular* `$resource` method then you are able to do this within the configuration of the `SpringDataRestAdapter`. By default it will use the *Angular* `$resource` function.

The following example shows how to set a custom function:

```javascript
myApp.config(function (SpringDataRestAdapterProvider) {

    // set the new resource function
    SpringDataRestAdapterProvider.config({
        'resourcesFunction': function (url, paramDefaults, actions, options) {
            // do the call to the backend and return your desired object
        }
    });
});
```

The description of the parameters you will find [here](#the-_resources-method-parameters-and-return-type). You can also read more about the configuration of the `SpringDataRestAdapter` [here](#configuration-of-the-springdatarestadapter) (and how to use the `$http` service in the configuration phase)

### Usage of `_embeddedItems` property

The `_embeddedItems` property is just a convention property created by the `SpringDataRestAdapter` to easily iterate over the `_emebedded` items in the response. Like with the `_resources` method, the `SpringDataRestAdapter` will recursively create an `_embeddedItems` property on the same level as a `_embedded` property exists for all the JSON response properties.

#### `_embeddedItems` usage example

This example refers to the JSON response in the [Overview](#overview). If you want to iterate over all categories in the response you would do it in the following way:

```javascript
SpringDataRestAdapter.process(response).then(function(processedResponse) {

  // log the name of all categories contained in the response to the console
  angular.forEach(processedResponse._embeddedItems, function (category, key) {
    console.log("Category name: " + category.name);
  });
});
```

**Be aware** that the original `_embedded` property gets deleted after the `SpringDataRestAdapter` processed the response.

### How to automatically fetch links

The `SpringDataRestAdapter` is able to fetch specified links automatically. This means that if you have the following response:

```json
{
    "_links": {
        "self": {
            "href": "http://localhost:8080/categories{?page,size,sort}",
            "templated": true
        },
        "anotherLink": {
            "href": "http://localhost:8080/anotherLink"
        }
    },
    ...
}
```
and you want to fetch the data from the `anotherLink` link then you just need to pass the link name to the `SpringDataRestAdapter` process function:

```javascript
SpringDataRestAdapter.process(response, 'anotherLink').then(function(processedResponse) {
  var fetchedObject = processedResponse.anotherLink;
});
```
Now you are able to get the data from the processed resource by just accessing the property named `anotherLink`.

The `SpringDataRestAdapter` by default adds the response of the link to a property in the original response with the same name as the link.

If you want to process the returned response again with the `SpringDataRestAdapter` then you are able to set the `recursive` flag when creating it:

```javascript
SpringDataRestAdapter.process(response, 'anotherLink', true).then(function(processedResponse) {
  ...
});
```

Now the response of the `anotherLink` will be processed the same way as the main response was processed. But *be aware* when setting the recursive flag to true, because when your responses of the links contain the same link name again, then it will end up in a infinite loop.

It will not fetch the `self` link as this would make no sense because the data is already in the response. The `self` key is also configurable. Read more [here](#configuration-of-the-springdatarestadapter).

As a last configuration option you are able to set the fourth parameter called `fetchMultiple` of the `process` method which is used to define if a link is fetched multiple times if the same link is contained in multiple links and you want to resolve each one of them. The drawback
is that it won't cache the responses in any way. So if you enable this then multiple network calls will be made. Here an example:

```javascript
SpringDataRestAdapter.process(response, 'anotherLink', true, true).then(function(processedResponse) {
  ...
});
```

:warning: If you set the `fetchMultiple` to `true` you could end up in an infinite loop if you have circular dependencies in your `_links`.

#### Fetch multiple or all links

If you want to fetch multiple links then you are able to add an array of strings with the given link names:

```javascript
SpringDataRestAdapter.process(response, ['anotherLink', 'testLink']).then(function(processedResponse) {
  ...
});
```
and if you want to fetch all links, then you can use the predefined and also configurable `fetchAllLinkNamesKey`:

```javascript
SpringDataRestAdapter.process(response, '_allLinks').then(function(processedResponse) {
  ...
});
```

Please read more [here](#configuration-of-the-springdatarestadapter) on how to configure the `fetchAllLinkNamesKey`.

#### Exchange the underlying fetch function

If you want to exchange the underlying function to fetch the links then you are able to do this within the configuration of the `SpringDataRestAdapter`. By default it will use the *Angular* `$http` function.

The following example shows how to set a custom function:

```javascript
myApp.config(function (SpringDataRestAdapterProvider) {

    // set the new resource function
    SpringDataRestAdapterProvider.config({
        'fetchFunction': function (url, key, data, fetchLinkNames, recursive) {
            // fetch the url and add the key to the data object
        }
    });
});
```

The description of the parameters you will find [here](#the-fetch-method-parameters). You can also read more about the configuration of the `SpringDataRestAdapter` [here](#configuration-of-the-springdatarestadapter) (and how to use the `$http` service in the configuration phase)

#### The fetch method parameters

The parameters for the fetch method are the following:

* `url`: The url of the link from which to fetch the data.
* `key`: The name of the link which is used to name the property in the data object.
* `data`: The data object in which the new property is created with the response of the called `url`.
* `fetchLinkNames`: The fetch link names to allow to process the fetched response recursiveley
* `recursive`: True if the fetched response should be processed recursively, false otherwise.

### How to use `SpringDataRestAdapter` with promises

The `SpringDataRestAdapter` is also able to process promises instead of data objects. The data object which is passed to the specified promise when it is resolved needs to be in the following formats:

```javascript
{
    data: {
        "_links": {
            "self": {
                "href": "http://localhost:8080/categories{?page,size,sort}",
                "templated": true
            },
            "anotherLink": {
                "href": "http://localhost:8080/anotherLink"
            }
        },
        // the rest of the JSON response
        ...
    }
}
```
or
```javascript
{
    "_links": {
        "self": {
            "href": "http://localhost:8080/categories{?page,size,sort}",
            "templated": true
        },
        "anotherLink": {
            "href": "http://localhost:8080/anotherLink"
        }
    },
    // the rest of the JSON response
    ...
}
```

The `data` property of the second format of the promise object is the JSON response of the back end. To process such a promise you need to call the `SpringDataRestAdapter` like in the following example:

```javascript
SpringDataRestAdapter.process(promise).then(function(processedResponse) {
    // you can now use the processedResponse as any other processed response from the SpringDataRestAdapter
};
```

You can also right away use the promise support with the `Angular` `$http.get()` method like in the following example:

```javascript
SpringDataRestAdapter.process($http.get('categories')).then(function(processedResponse) {
    $scope.categories = processedResponse._embeddedItems;
};
```

### Configuration of the `SpringDataRestAdapter`

The `SpringDataRestAdapter` is designed to be configurable and you are able to configure the following properties:

* `linksKey` (default: `_links`): the property name where the resources are stored.
* `linksHrefKey` (default: `href`): the property name where the URL is stored in a link object.
* `linksSelfLinkName` (default: `self`): the name of the self link in the links object.
* `embeddedKey` (default: `_embedded`): the property name where the embedded items are stored.
* `embeddedNewKey` (default: `_embeddedItems`): the property name where the array of embedded items are stored.
* `embeddedNamedResources` (default: `false`): true if the embedded resources names (can be more than one) should be
left as is, false if they should be removed and be replaced by the value of the first embedded resource
    * Example if set to true:
    ```json
    {
        ...
        "_embeddedItems": {
            "categories": [...]
        }
        ...
    }
    ```
    * Example if set to false:
    ```json
    {
        ...
        "_embeddedItems": [...]
        ...
    }
    ```
* `hrefKey` (default: `href`): the property name where the url is stored under each specific link.
* `resourcesKey` (default: `_resources`): the property name where the resource method is stored.
* `resourcesFunction` (default: `undefined`): the function to use to call the backend. Read more how to do this [here](#exchange-the-underlying-angular-resource-function)
* `fetchFunction` (default: `undefined`): the function to use to fetch data from the backend. Read more how to do this [here](#exchange-the-underlying-fetch-function)
* `fetchAllKey` (default: `_allLinks`): the key to pass to the `SpringDataRestAdapter` to fetch all available links

You are able to configure the `SpringDataRestAdapter` provider in a *Angular* configuration block in the following way:

```javascript
myApp.config(function (SpringDataRestAdapterProvider) {

    // set the links key to _myLinks
    SpringDataRestAdapterProvider.config({
        'linksKey': '_myLinks'
    });
});
```

The config method of the `SpringDataRestAdapterProvider` takes a configuration object and you are able to override each value or completely replace the whole configuration object. The default configuration object looks like this:

```json
{
    "linksKey": "_links",
    "linksHrefKey": "href",
    "linksSelfLinkName": "self",
    "embeddedKey": "_embedded",
    "embeddedNewKey": "_embeddedItems",
    "embeddedNamedResources": false,
    "resourcesKey": "_resources",
    "resourcesFunction": undefined,
    "fetchFunction": undefined,
    "fetchAllKey": "_allLinks"
}
```
If you want to use the `$http` service inside the fetch function in the configuration phase then you need to use the Angular injector to get the `$http` service:

```javascript
myApp.config(function (SpringDataRestAdapterProvider) {

    // set the new fetch function
    SpringDataRestAdapterProvider.config({
        fetchFunction: function (url, key, data, fetchLinkNames, recursive) {
            var $http = angular.injector(['ng']).get('$http');

            $http.get('/rest/endpoint').then(function (responseData) {
                console.log(responseData);
            })
        }
    });
});
```

## The `SpringDataRestInterceptor`

If you want to use the `SpringDataRestAdapter` for all responses of the *Angular* `$http` service then you can add the `SpringDataRestInterceptor` to the `$httpProvider.interceptors` in an *Angular* configuration block:

```javascript
myApp.config(function (SpringDataRestInterceptorProvider) {
    SpringDataRestInterceptorProvider.apply();
});
```
The `apply` method will automatically add the `SpringDataRestInterceptor` to the `$httpProvider.interceptors`.

## Dependencies

The `spring-data-rest` *Angular* module requires the `ngResource` *Angular* module.

## Release notes

Check them here: [Release notes](https://github.com/guylabs/angular-spring-data-rest/blob/master/RELEASENOTES.md)

## Acknowledgements

When I first searched for an *Angular* module for Spring Data REST I just found the marvelous project of Jeremy which is useful if you just use [Spring HATEOAS](http://projects.spring.io/spring-hateoas). So please have a look at his [angular-hateoas](https://github.com/jmarquis/angular-hateoas) project because he gave me the full permissions to use his code base and project structure for this project here. At this point I want to thank him for his nice work.

## License

This *Angular* module is available under the MIT license.

(c) All rights reserved Guy Brand
