---
layout: post
title: Cross-Origin Resource Sharing (CORS)
---

In simple terms, Cross-Origin Resource Sharing is all about making requests between resources residing on different domains.

So, requests are considered "cross-origin" when the request initiator is in a different domain than the request recipient.

I can hear you saying:

> Hey dummy, everyone makes requests to other domains to fetch images, JavaScript libraries, webfonts...
> What's special with this?

In this article we will be talking about AJAX requests between different domains.
For security reasons, browsers restrict cross-origin HTTP requests initiated from within scripts.
That restriction is named the "[Same-origin policy](https://en.wikipedia.org/wiki/Same-origin_policy)".

Two origins are "same" when their following properties match:

- URI scheme (http / https),
- hostname
- port number

## How to lift the restriction

To lift the restriction, the CORS standard comes in handy and dictates a way to control and define permissions for other origins
to be authorized. The CORS standard works by adding HTTP headers to server responses and describe the set of origins that are permitted
to query the requested resource.

## The "preflight"

Browsers may "preflight" the request and ask the server for the allowed origins and supported methods.
A "preflight" is an "OPTIONS" method HTTP request sent from the browser to the server in order to determine whether the actual
request is acceptable. Upon "approval" from the server, the browser will then send the actual request.

As already said, "preflights" are not always required. The browser determines that based on the content of the actual request.

> OPTIONS is an HTTP method used to fetch further information from a webserver about a resource.
> Its an idempotent method, meaning that you shouldn't use it to change a resource.

A nice representation of a cross-origin request:

![Cross-origin request flowchart](/public/img/2016_10_01_cross_origin_resource_sharing/flowchart.png)
This work, is a derivative of [Bluesmoon](http://www.flickr.com/photos/bluesmoon/)'s work
used under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)
and is licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) by Kostas Milonas.

## The request initiator

An example JavaScript code of how a request initiator looks like:

```
// resides somewhere in example.com

$.ajaxSetup({
    headers: { "X-Requested-With": "XMLHttpRequest" }
});

$.ajax({
    method: "POST",
    url: "http://www.books.com/availability",
    data: data,
    success: function (data) {
        // ...
    },
    error: function () {
        // ...
    }
});
```

Nothing special. Just note the ```url``` of the AJAX request pointing to another domain.

Also, by adding the ```X-Requested-With``` header we ensure that the request recipient application will effectively
detect that its an AJAX (XMLHTTP) request.

## The request recipient

As previously said, when a cross-origin AJAX request takes place, may cause the browser to firstly send an OPTIONS HTTP request,
the "preflight".

On the server side, we have to handle that OPTIONS request and add the HTTP headers mandated by the CORS standard to inform
the browser if this cross-origin request is allowed. If it is, the browser will send the actual request.

```
if (
    !empty($_SERVER["REQUEST_METHOD"])
    && "OPTIONS" === $_SERVER["REQUEST_METHOD"]
) {
    header("Access-Control-Allow-Origin: example.com");
    header("Access-Control-Allow-Methods: GET, POST, OPTIONS");
    header("Access-Control-Allow-Headers: X-Requested-With");
    exit();
};
```

So, the server responds with the following "rules":

- "example.com" is allowed to query this resource
- POST, GET, and OPTIONS are viable methods to query this resource

The value of ```Access-Control-Allow-Origin``` can also be a ```*``` to allow all domains.

## Possible troubles

I have experienced PHP not being able to access the data of a POST request after a "preflight".
The ```$_POST``` global variable was empty.

I finally managed to access the data with:

```
\file_get_contents(php://input);
```

An example of how to get the POST data and store them in the ```$data``` variable.

```
$dataRaw = \file_get_contents("php://input");
\parse_str(parse_url($dataRaw)["path"], $data);
```

Assume the request data were the following:

```
firstName=Kostas&lastName=Milonas
```

Now you can access them with:

```
$firstName = $data["firstName"];
// ...
```