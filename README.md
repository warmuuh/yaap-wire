#Yaap-Wire plugin [![Build Status](https://travis-ci.org/warmuuh/yaap-wire.svg?branch=master)](https://travis-ci.org/warmuuh/yaap)
This project contains annotations that only work as a [wire.js](https://github.com/cujojs/wire)-plugin.


##wire.js Integration
To use @Autowired (and annotations in general) in wire.js, simply add it as a plugin:
```js
var wire = require("wire");
wire({
	level: "INFO",
	logger: {create:  './Logger'},

	plugins: [
		{module: "yaap/wire"}
	]
}, {require: require}).then(function(ctx){
	ctx.logger.log("message");
}, console.error);
```
Everything else is done by yaap, so you can start use your annotations:

```js
//Logger.js

module.exports = {
  log: function(message, /*@Autowire*/ level){
		console.log(level + ": " + message);
	}
}
```
`level` references the value in the wire-context (with value "INFO") now.

`Remark:` Yaap/wire uses the parameter name to autowire. You can also supply a reference name with `/*@Autowire("level")*/`

`Remark:` You can also annotate the whole function with `@Autowire` so every parameter will be autowired by name.



##Wire annotations

* `@Autowired([<refName>])` (parameter/function/class): if a parameter is unassigned or null, a bean from the wire-context will be injected instead.
		If the annotation is placed at a parameter, the given refName or, if ommitted, the parameter-name will be used to resolve the reference.
		If the annotation is function-level, all parameters that are null or unassigned will be autowired by parameter-name.
		If the annotation is class-level, the parameter could be a string, an array of strings or an object that maps strings onto strings. New Properties according to the given names will be injected with the referenced beans.

* `@PostConstruct` (function): the annotated function will be called after container finished configuring the bean.

* `@PreDestroy` (function): the annotated function will be called, if context.destroy() is called.


##Express.js integration

There are also out-of-the-box annotations included for creating webapps in a springMVC-like manner.
More information on how to setup this integration is available [here](docs/express.md)
A simple example of a service:

```js
MyService.prototype = {
    index: function ()/*@GET*/ {
		return 'index';
    },

    submit: function (name, age)/*@POST @Param*/ {
     var msg = (age < 18)? "You are too young" : "You are welcome!";
	 return {view:'greet', model:{name: name, msg: msg}};
    }
};
```
As known from SpringMVC, the returned values determine, which view will be called and with what parameters.
More details on this in my [blogpost](http://cubiccow.blogspot.com/2013/02/autowire-for-javascript.html).


###browser-specific wire annotations
for these annotations, you need to add the additional `yaap/wire/html`-plugin! (as shown in the browser-example)

* `@On(<refName>, <event>)` (function): the annotated function will automatically be bound the the event of the given dom-node.
	This is intended to be used with the `wire/dom`-plugin. <refName> can reference one or more elements in the dom (though
	it is probably better practice to reference a bean in the wirecontext, which itself references the dom-nodes).
	For example, you could bind a clickhandler with this annotation: `@On("dom.all!.btn","click")`.
	The annotated method accepts one argument that is the event (though you could add additional @autowired arguments, if you want)

###Node-Specific wire-annotations (Express.js)
for these annotations, you need to add the additional `yaap/wire/express`-plugin! (and you need to feed the express-application into the plugin as shown in the express-example)

* `@GET/POST/PUT/DELETE([<pathspec>])` (function): registeres the function as an endpoint in an express-application with either the given path
	or the functionname as path, if omitted.
* `@Param([<name>])` (parameter/function): fetches the annotated parameter from the query/path/post parameters.
	If annotation is on function-level, all parameters (except `@Body`-annotated parameters) are injected by-name.
* `@Body` (parameter/function): if parameter-level, then the request-body will be injected. If on function-level,
	it states that the return-value will be returned as-is (Just like @ResponseBody does in SpringMVC).
* `@JSON` (function): states, that the returned value will be send as response in json-format. (using [response.json](http://expressjs.com/api.html#res.json)).
* `@Callback` (parameter): a callback will be injected so you can return your response asynchronously. You can also simply return a promise, so you do not need to use a callback for asynchronous operations.
