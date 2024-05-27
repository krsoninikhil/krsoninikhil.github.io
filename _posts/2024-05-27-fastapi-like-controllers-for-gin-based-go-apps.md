Everytime I start a new project in Go, setting up the boilerplat is hassle.
Unless you add your configuration loader, DB connections and authentication APIs, middleware,
it's same story everytime. While writing all of these packages is fun and gets you in the flow,
it takes time and it's not as quick as writing a app in FastAPI. 

The biggest pain point I faced is writing handlers and repeating request parsing logic for
handler with a different request types because frameworks like Gin provides you context as 
argument to handler and expect you bind it however you expect which is great but Go supports
generic and I don't see why this can't be generalised and make the request type directly 
available in the handler.

## Getting Rid Of Request Binding In Every Handler

Gin's handler function:

```go
type HandlerFunc func(c *gin.Context)
```

So, this time when I started a personal project, I changed the focus to solve for generic 
based handlers. After a few iterations following seems to take care of exactly what I was looking for:

```go
type bindedHandlerFunc[Req, Res any] func(*gin.Context, Req) (*Res, error) 
```

This allowed to change your route mapping from:

```go
// request type
type (
    RequestBody struct {
        JsonField  string   `json:"json_field"`
        QueryParam string   `form:"query_param"`
    }
    ResponseBody struct {
        QueryParamInput   string `json:"query_param_input"`
        JsonFieldInput    string `json:"json_field_input"`
        PathParamInput    int    `json:"path_param_input"`
    }
)

// handler
func myHandler(c *gin.Context) {
    var req RequestBody
    if err := c.ShouldBindJSON(req); err != nil {
        return errors.Wrap(err, "error binding request body")
    }
    if err := c.ShouldBindQuery(req); err != nil {
        return errors.Wrap(err, "error binding query params")
    }

    p := c.Param("path_param") // this could be part of request struct, but then you still need to add a ShouldBindUri block
    pInt, err := strconv.ParseInt(p, 10, 64)
    if err != nil {
        return errors.Wrapf(err, "error parsing path_param: %v", p)
    }

    return c.JSON(http.StatusOK, ResponseBody{
        QueryParamInput: req.QueryParam, 
        JsonFieldInput:  req.JsonField,
        PathParamInput:  pInt,
    })
}
// routes
r.GET("/path", myHandler)
```

to something like:
```go
// request type
type RequestBody struct {
    JsonField  string   `json:"json_field"`
    PathParam  int      `uri:"path_param"`
    QueryParam string   `form:"query_param"`
}

// handler
func myHandler(c *gin.Context, req RequestBody) (*ResponseBody, error) {
    return c.JSON(http.StatusOK, ResponseBody{
        QueryParamInput: req.QueryParam, 
        JsonFieldInput:  req.JsonField,
        PathParamInput:  req.PathParam,
    })
}

r.GET("/path", request.BindAll(myHandler))
```

Pretty neat, han?

If you don't want to allow binding from URI and body in the same request, 
there `request.BindGet`, `request.BindCreate`, `request.BindDelete` 
to only bind for query, request body and path param respectively.

You can find this `request` package [here on GitHub](https://github.com/krsoninikhil/go-rest-kit/tree/main/request).

## Exposing ready made CRUD Endpoints

Once this was achieved, I remeber from my Rails experience, for a CRUD api,
all you need to do is provide a resource and CRUD is exposed. So to achieve 
similar feature, I wrote a `crud` package which allows you to expose the CRUD 
endpoints just by writing the model and then request, response type. 
Read more about it in the readme here - 
[https://github.com/krsoninikhil/go-rest-kit](https://github.com/krsoninikhil/go-rest-kit)

The module has other useful packages like pre written handlers for auth, configuration loader, 
postgres connection, etc.

--
[Nikhil](https://twitter.com/krsoninikhil)