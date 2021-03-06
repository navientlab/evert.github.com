---
date: "2019-04-09 15:00:00 UTC"
title: "420 Enhance your calm"
permalink: /http/420-enhance-your-calm
tags:
   - http
   - http-series
location: "Bloor St W, Toronto, ON, Canada"
geo: [43.660942, -79.429954]
---

The [`420 Enhance Your Calm`][1] status code is an unofficial extension by
Twitter.

Twitter used this to tell HTTP clients that they were being rate limited. Rate
limiting means putting restrictions on the total number of requests a client
may do within a time period.

For Twitter it was useful enough to extend the protocol with a new HTTP status code.
However, adding your own HTTP status codes is a bad idea and should generally be
avoided.

In many cases it's possible to express your error condition through one of the
exiting status codes. Error statuses that are more unique to your application
should simply be communicated via a response body. (for example the standard
[`application/problem+json`][2] response).

So when is a new HTTP status code useful? The general idea is that adding a
new code to the list is useful if it's possible for a generic HTTP client
to do something with the response.

The idea of having a standard HTTP code for rate limiting was useful enough to
be turned into a real HTTP status, So a few years later, we got the
[`429 Too Many Requests`][3] status code, which Twitter ended up adopting too.

[1]: https://developer.twitter.com/en/docs/basics/response-codes
[2]: https://tools.ietf.org/html/rfc7807 "application/problem+json"
[3]: /http/429-too-many-requests "429 Too Many Requests"
