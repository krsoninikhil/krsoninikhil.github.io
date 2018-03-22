I was working on project that needed to send a POST request with some data to
a service hosted on a different domain. The service's logs were saying no
support for OPTIONS request, which was the case, because it expects a
POST request and that's what it supports. So why is it getting a OPTIONS
request? I checked the network tab in browser inspector, it indeed was
sending an OPTIONS request, that too with an empty body. Weird!

If you have ever wrote any ajax calls, there are high chances you would have
encountered problems related to `cross-domain`, saying you're not allowed to
access resources that doesn't have same origin. This happens when you try to
call services from different hosts than your own server. And I knew the reason
is browser's [Same Origin policy](https://en.wikipedia.org/wiki/Same-origin_policy). What didn't knew how it works?!

What is Same Origin policy, well, the moment you navigate to an URL, the default settings of your browser allows the website developer to execute their
JavaScript on your machine. To limit what that script can do, like stealing cookies set by another site, acessing your local file system and may be uploading your personal data stored on your personal machine to their servers, browser uses this Same Origin policy, which allow one domain to access resources that belongs to that domain only. And when someone tries to access resources from
other sites, the browser asks the other host if it allows current host to access thas resource. If other host says yes, it goes ahead with your request.

For few cases, like requests with POST method or content type as `application/json`, it first sends something called a *preflight* request with OPTIONS method, containing a header `Origin`, which server can read and returns the methods it supports for request from this origin and header it needs. Browser can cache this information and might not verify every time.

The OPTIONS request I was getting, was this preflight request and since I was not handling this request, I never received the POST request which browser would have sent after receiving response from it's first request. Handling this on server side is what is called CORS, Cross Origin Resource Sharing. Most of the API framework provides a library to handle this. You can specify which origin domain you want to allow and these libraries will handle the OPTIONS request for it and will respond with proper headers set.

[This](https://www.html5rocks.com/en/tutorials/cors/) is good resoure to read more about CORS.
