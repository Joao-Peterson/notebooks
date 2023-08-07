# dotnet

Everything dotnet related!

- [dotnet](#dotnet)
- [Introduction](#introduction)
- [ASP.NET](#aspnet)
	- [Controller based API's](#controller-based-apis)
	- [Controller class](#controller-class)
		- [`ApiController` decorator](#apicontroller-decorator)
		- [`Route` decorator](#route-decorator)
	- [Actions](#actions)
		- [`HttpMethod` decorator](#httpmethod-decorator)
		- [`Route` decorator](#route-decorator-1)
		- [`Consumes` decorator](#consumes-decorator)
		- [`Produces` decorator](#produces-decorator)
		- [Source binding](#source-binding)
	- [Model validation](#model-validation)
	- [Route templates](#route-templates)

# Introduction

dotnet is a system consisting of a framework and runtime to develop and run applications, is cross platform, it's main language is C#, but can be F# and VB.

# ASP.NET

asp.net is a framework over dotnet to build web applications. Simply, it has a MVC library and the framework itself is very opinionated. 

## [Controller based API's](https://learn.microsoft.com/en-us/aspnet/core/web-api/?view=aspnetcore-7.0) 

This framework uses controllers as the main mechanism for working with API, we declare a controller class in the namespace `project.Controllers` and that's it!

```c#
[ApiController]
[Route("/api/[controller]")]
public class ARouteController : ControllerBase
{
    [HttpGet]
    public IActionResult Get(){
		// method code
    }
}
```

The controller derives from `ControllerBase`, can be derived from just `Controller`, but `Controller` is reserved for application that uses views in a MVC, API's don't use that, so `ControllerBase` is recommended.


## Controller class

### `ApiController` decorator

The `ApiController` decorator is optional but simplifies things, cause it gives us some of that opinionated behavior, may we desired custom behavior, we could remove it.

It gives us:

- Attribute routing requirement
- Automatic HTTP 400 responses
- Binding source parameter inference
- Multipart/form-data request inference
- Problem details for error status codes

### `Route` decorator

The route decorator specifies the route for a controller. It accepts a string for a route, simple as that. We may also write onto the string this templated syntax: `"[controller]"`, that when used inside the string, expands to the controller class name. In this example:

```c#
[ApiController]
[Route("/api/[controller]")]
public class UsersController : ControllerBase
{
	/// ...
}
```

The assigned route fro the controller would be `"/api/users"`. It's the class name minus the `"Controller"` ahead of it, so this following class:

```c#
[ApiController]
[Route("/api/[controller]")]
public class UsersControllator : ControllerBase
{
	/// ...
}
```

Would be given the route: `"/api/userscontrollator"`.

* **Note that all of this is case-insensitive.**

## Actions

Actions are the controller class methods that process methods for the http API.

### `HttpMethod` decorator

Each action can have a decorator to sinalize it's http method:

```c#
[ApiController]
[Route("/api/[controller]")]
public class ARouteController : ControllerBase
{
    [HttpGet]
    public IActionResult Get(){
		// method code
    }

    [HttpGet("{id}")]
    public IActionResult GetById(int id){
		// method code
    }

    [HttpPost]
    public IActionResult Post(){
		// method code
    }

	// ...
}
```

A list of method decorators:

- `HttpGet`
- `HttpPost`
- `HttpPut`
- `HttpPatch`
- `HttpDelete`

And the decorator might also specify a route template, to extend the controller route or specify a templated endpoint. An example, to get a resource from the route `api/entry/1`, which is the resource number 1, the definition can be given as:

```c#
[ApiController]
[Route("/api/entry")]
public class EntryController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult Get(int id){
		// method code
    }
}
```

For more info on route templates, see [Route templates](#route-template).

### `Route` decorator

Just as controllers, actions can have specific routes that extends the controller routes. 

They work just like [Method decorators](#httpmethod-decorator), except they don't specify a method for the action, just the route.

For more info on route templates, see [Route templates](#route-template).

### `Consumes` decorator

Specify what this action can consume in it's body, the mime-type. Ex:

```c#
[ApiController]
[Route("/api/entry")]
public class EntryController : ControllerBase
{
    [HttpPost]
    [Consumes("application/json")]
    public IActionResult Post(Entry entry){
		// method code
    }
}
```

This way you can have multiple actions on the **same route and method**, but have only one of them be called by which type the client passed on the body. So you can have an action for when a user passes an xml or a json, having different validations for each type, but on the same route and method.

### `Produces` decorator

Specify what this action returns as a result.

Just like [Consume decorator](#consumes-decorator), it specifies the mime type for the body, but this time, the returned body.

We also have more produce decorators: 

- `ProducesResponseType`
- `ProducesDefaultResponseType`
- `ProducesErrorResponseType`

Each of which do what they say, amazing! 

They receive an argument(s) of type `StatusCodes`, an static class that defines http status codes. Ex:

```c#
[ApiController]
[Route("/api/entry")]
public class EntryController : ControllerBase
{
    [HttpPost]
    [Consumes("application/json")]
    [Produces("application/json")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status201OK)]
    [ProducesDefaultResponseType(StatusCodes.Status200OK)]
    [ProducesErrorResponseType(StatusCodes.Status400BadRequest)]
    public IActionResult Post(Entry entry){
		// method code
    }
}
```

### [Source binding](https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding?view=aspnetcore-7.0)

asp.net is an opinionated framework, and to makes things easier, data, like the http query, headers, the body, can be bound to parameters on a action automatically, so don't need to parse anything, just write the action itself.

An example of a action of a Post request that receives a `Entry` json body and a `id` parameter on query string:

```c#
public class Entry{
	public string name {get; set;}
	public int type {get; set;}
} 

[ApiController]
[Route("/api/entry")]
public class EntryController : ControllerBase
{
    [HttpPost]
    [Consumes("application/json")]
    public IActionResult Post(Entry entry, int id){
		// method code
    }
}
```

Request: 

```json
POST http://localhost/api/entry/42 
{
	"name": "foo bar",
	"type": 2 
}
```

Note how the json body is mapped directly into the `entry` class, and the 42 to the `id` parameter. Data is bound by name (**case insensitive**), and in certain order. 

Body is always bound to one complex type, like an object, in this case the `Entry entry`, and other data is bound to simple types, like `id`.

Decorators can be used in the actions parameters to bind data from the request:

- [FromBody](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.frombodyattribute) Request body
- [FromForm](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromformattribute) Form data in the request body
- [FromHeader](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromheaderattribute) Request header
- [FromQuery](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromqueryattribute) Request query string parameter
- [FromRoute](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromrouteattribute) Route data from the current request
- [FromServices](https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/dependency-injection?view=aspnetcore-7.0#action-injection-with-fromservices) The request service injected as an action parameter
- [AsParameters](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.asparametersattribute) Method parameters

So for example, to catch the language type of the request, you may declare an action like:

```c#
public void Get(int id, [FromHeader(Name = "Accept-Language")] string language){
	// ...
}
```

If the [API controller decorator](#apicontroller-decorator) is used on the controller, inside the action you can access `ModelState.IsValid` to check is all the values were bound correctly before doing any logic.

By default, a model state error isn't created if no value is found for a model property. The property is set to null or a default value:

- Nullable simple types are set to `null`.
- Non-nullable value types are set to `default(T)`. For example, a parameter `int id` is set to `0`.
- For complex Types, model binding creates an instance by using the default constructor, without setting properties.
- Arrays are set to `Array.Empty<T>()`, except that `byte[]` arrays are set to `null`.

If model state should be invalidated when nothing is found in form fields for a model property, use the [`[BindRequired]`](https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding?view=aspnetcore-7.0#bindrequired-attribute) attribute.

Note that this `[BindRequired]` behavior applies to model binding from posted form data, not to JSON or XML data in a request body. Request body data is handled by [input formatters](https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding?view=aspnetcore-7.0#input-formatters).

## Model validation

## Route templates

A string that defines a route. Ex:

```
/api/v1/itens/foods/{id}
```

Where we can see the literal string that defines the route but also the templated parameter `id`, that will match anything passed after the route `foods`. 