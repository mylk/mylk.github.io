---
layout: post
title: Avoid clickjacking or how to manage framing
---

There may be some times where you would need to:

- deny embedding your web application into other websites
- allow a website to embed your web application

The first one is also known as protection from "clickjacking".

> Clickjacking is tricking users into clicking something seemingly harmless and different from what they think they click on,
> in order to reveal confidential information or infect them.

Embedding your web application into a 3rd party website all about someone using either a ```<frame>```, ```<iframe>``` or ```<object>```
HTML tag to include your content into his website.

[#](#how-to-avoid-clickjacking){:.header-anchor}

## How to avoid clickjacking

There are a couple of ways to fight against clickjacking:

- use the appropriate HTTP headers,
- "frame-busting" JavaScripts

### JavaScript is not the solution

Using JavaScript to prevent being "framed" is really popular, but be careful about depending on it. The "restricted zones"
give the ability to disable JavaScript in the context of a frame, turning the frame-busting code useless.

Some examples of how to disable JavaScript on iframes:

On Chrome:

```
<iframe src="http://www.example.com" sandbox></iframe>
```

On IE:

```
<iframe src="http://www.example.com" security="restricted"></iframe>
```

On Firefox (works on IE too) the parent page (the page that frames you) has to just enable the "design mode".

```
document.designMode = "on";
```

I would suggest not to depend on frame-busting with JavaScript.

[#](#the-http-headers-way){:.header-anchor}

## The HTTP headers way

Using HTTP response headers is the most reliable way. The headers tell the browser that embedding a page of yours
into another page is not permitted.

The appropriate header to deny framing:

```
X-Frame-Options: DENY
```

The ```DENY``` option, will even prevent your own web application to frame your web application.

[#](#manage-framing){:.header-anchor}

## Manage framing

There may be a case where you want your web application / website to be framed but only into a specific website.

Use the header that fits your case:

```
X-Frame-Options: SAMEORIGIN

X-Frame-Options: ALLOW-FROM https://example.com/
```

With ```SAMEORIGIN``` only your own web application can frame your web application and with ```ALLOW-FROM``` only the defined page
can frame your web application.

[#](#setting-up-x-frame-options){:.header-anchor}

## Setting-up X-Frame-Options

The ```X-Frame-Options``` header will not work if you just place it on an HTML ```<meta>``` tag.

One way to set it up is to manually send the header on every response, but the most simple way is to configure your webserver to do so.

Also, its really possible that your PHP framework may give you the ability to send the header by just configuring the framework,
so you won't need to touch your webserver configuration at all.

[#](#limitations-of-x-frame-options){:.header-anchor}

## Limitations of X-Frame-Options

The ```ALLOW-FROM``` option is relatively recent and may not be supported by all browsers. So, if the visitor's browser doesn't support it,
you don't provide any clickjacking protection to that visitor.

Something else disturbing is that there is no straight-forward way to add multiple origins to ```ALLOW-FROM```.
Just one ```X-Frame-Options``` header is allowed and can have only one value on the ```ALLOW-FROM``` directive.
So, you will need some programming juggling there.

Last but not least, even while ```X-Frame-Options``` is the established way of managing framing and being supported by the majority of browsers,
it was never standardized and is being deprecated in favour of the```frame-ancestors``` directive.

[#](#the-frame-ancestors-directive){:.header-anchor}

## The frame-ancestors directive

```frame-ancestors``` is a directive of the ```Content-Security-Policy``` (CSP) header.

To deny framing by all origins:

```
Content-Security-Policy: frame-ancestors 'none'
```

To allow framing from your origin only:

```
Content-Security-Policy: frame-ancestors 'self'
```

To allow only a single website to frame you:

```
Content-Security-Policy: frame-ancestors example.com
```

What's nice is that ```frame-ancestors``` allows you to define multiple origins by separating them with spaces.

[#](#limitations-of-csp){:.header-anchor}

## Limitations of CSP

The browser support of the ```frame-ancestors``` is still limited (2016 Q4).

Also, if you send both ```X-Frame-Options``` and ```frame-ancestors```, some browsers will give priority to
```X-Frame-Options```, even if the standard says that ```X-Frame-Options``` has to be ignored if the ```frame-ancestors``` directive is defined.

Note that neither CSP will work if is set in an HTML ```<meta>``` tag.