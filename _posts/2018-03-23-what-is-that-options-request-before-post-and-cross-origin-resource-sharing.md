I was working on project that was sending a POST request with some
data from a web client to a service hosted on a different domain. The
service's logs were saying no support for OPTIONS request, which made
sence because it expects a POST request and that's what it
supports. So who is sending this OPTIONS request when I'm tring to
sending a POST from my browser client? I checked the network tab in
browser inspector, browser indeed was sending an OPTIONS request, that
too with an empty body and I didn't see any POST request. Weird!

If you have ever wrote any ajax call, there are high chances you would
have encountered problems related to `cross-domain`, saying you're not
allowed to access resources that doesn't have same origin. This
happens when you try to call services from different hosts other than
your own server through Javascript. And I knew the reason is browser's
[Same Origin policy](https://en.wikipedia.org/wiki/Same-origin_policy)
and what that is, what I didn't knew was how it exactly works?!

What is Same Origin policy, well, the moment you navigate to an URL,
your browser allows the website developer to execute theirs JavaScript
on your machine. To limit what that script can do, like accessing data
from another web page opened in another tab or cookies created by
another websites, browser uses this Same Origin policy, which allow
one domain to access resources that belongs to that domain only
i.e. resources with same origin. This applies only on what already
loaded JS can access through DOM or by making a Ajax calls and not on
web page itself embedding static content e.g. images, CSS or Javascipt
files itself.

When Javascipt of some website tries to access resources from other
sites i.e. make a cross-origin request, the browser adds a `Origin`
header with the request and checks the response header for specific
headers like `Access-Control-Allow-Origin` which signals browser that
server allows current origin to access its data. This happens if
request does not modify data e.g. GET or HEAD request. For requests
that modify data like requests with POST method or content type as
`application/json`, browser first asks the other host if it allows
requesting host to access their resource. If other host says yes, it
goes ahead with your request.

Browser implements this by first sending something called a
*preflight* request which is a HTTP request with OPTIONS method,
containing a header `Origin` and other `Access-Control-*` headers
describing original request's method and content type, which server
can respond with proper headers describing methods it supports for
request from this origin and header it needs. Browser can cache this
information and might not verify every time.

You can see this in browser's Developer Tools. Open Inspect window for
current or any web page other than `example.com` and execute following
JS in console:

```javascript
fetch("https://example.com").then(res => console.log(res))
```

You should seen error saying `Cross-Origin Request Blocked` and if you
check Network tab, browser did made a GET request as specified above
along with a `Origin` header but since the response header did not
have any `Access-Control-*` headers, browser will not return the
response to the client and hence you only saw error in console. Now
try executing this JS:

```javascript
fetch("https://example.com", {
  method: 'POST',
  body: JSON.stringify({"foo": "bar"}),
  headers:{
    'Content-Type': 'application/json'
  }
}).then(res => console.log(res));
```

You'll get same error in console and if you check Network tab again,
you should see a OPTIONS request and not a POST request. Because the
response of OPTIONS request does not have headers that say --allow
current webpage to access it's data, browser never sent the real request.

The OPTIONS request I was getting was this preflight request and since
I was not handling this request, I never received the POST request
which browser would have sent after receiving response from it's first
request. Handling this is what is called CORS i.e. [Cross Origin
Resource
Sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing). From
client side, this is relaxation to Same Origin Policy provided by
browser. From server side, most of the API frameworks provides a
library to handle this. You can specify which origin domain you want
to allow and these libraries will handle the OPTIONS request for you
and will respond with proper headers set.

[This](https://www.html5rocks.com/en/tutorials/cors/) is good resource
to read more about CORS.
