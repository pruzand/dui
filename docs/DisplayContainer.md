---
layout: default
title: delite/DisplayContainer
---

# delite/DisplayContainer

`delite/DisplayContainer` is a [`delite/Container`](Container.md) subclass which adds the ability for the container to 
manage the appearance of its children.

##### Table of Contents
[Using DisplayContainer](#using)  
[Extending DisplayContainer](#extending)  
[Events](#events)  
[Writing a Controller for DisplayContainer](#controller)    

<a name="using"></a>
## Using DisplayContainer

On a container extending `delite/DisplayContainer` one can trigger the visibility of a particular child by calling
the `show()` function. Conversely one can make sure a particular child is hidden by calling the `hide()` function. By
default the display container will just show or hide the child. However subclasses can implement particular transition
effects in order to transition from one visibility state to another. For this reason both functions managing the visibility
return a `Promise` that will be resolved when the transition is finished. Also both accept the same parameters:
  
* `dest` : Widget/HTMLElement or id that points to the child the function must show or hide
* `params` : Optional params to customize the transition, interpreted by subclasses.
  
```js
mycontainer.show("mychildid", {/* optional params depending on the subclass */}).then(function() {
  // promise has been resolved, the child is now visible
});
``` 

<a name="extending"></a>
## Extending DisplayContainer

In order for a container to leverage `delite/DisplayContainer` it must extend it and possibly implement the `changeDisplay()` 
and/or `load()` functions in order to customize their behavior. The following subclass is looking at the parameters passed
to the `show()` or `hide()` function in order to perform a visual transition when switching the child visibility. In 
particular it performs a fading in or out transition based on the value of the `fade` parameter.

```js
require(["delite/register", "delite/DisplayContainer", "dojo/Deferred"/*, ...*/], 
  function (register, DisplayContainer, Deferred/*, ...*/) {
  return register("my-container", [HTMElement, DisplayContainer], {
    changeDisplay: dcl.superCall(function(sup) {
      return function (widget, params) {
        var deferred = new Deferred();
        if (params.fade === "in") {
          // if there is a parameter telling us to do a fade in let's do it
          $(widget).fadeIn(1000, function() {
             deferred.resolve();
          });
        }
        sup.apply(this, arguments);
        if (params.fade === "out") {
          // if there is a parameter telling us to do a fade in let's do it by setting corresponding CSS class
          $(widget).fadeIn(1000, function() {
            deferred.resolve();
          });
        }
        return deferred.promise;
      };    
    })
  });
});
```

Note that as the function performs an asynchronous action it returns a promise that will be resolved once the action
is completed.

One can trigger a fade in this way:

```js
mycontainer.show(child, { fade : "in" });
```

<a name="events"></a>
## Events

The `delite/DisplayContainer` class provides the following events:

|event name|dispatched|cancelable|bubbles|properties|
|----------|----------|----------|-------|----------|
|delite-display-load|on any show or hide action|True|True|<ul><li>`dest`: the reference of the child to load</li><li>`loadDeferred`: the deferred to resolve once the child will be loaded</li><li>`hide`: set to true if in a hide action</li><li>any other param passed to the show or hide function</li></ul>|
|delite-before-show|just before a child is shown|No|True|<ul><li>`dest`: the reference of the loaded child</li><li>`child`: the child to show</li><li>any other param passed to the show function</li></ul>|
|delite-after-show|after a child has been shown|No|True|<ul><li>`dest`: the reference of the loaded child</li><li>`child`: the child that has been shown</li><li>any other param passed to the show function</li></ul>|
|delite-before-hide|just before a child is hidden|No|True|<ul><li>`dest`: the reference of the loaded child</li><li>`child`: the child to hide</li><li>any other param passed to the hide function</li></ul>|
|delite-after-hide|after a child has been hidden|No|True|<ul><li>`dest`: the reference of the loaded child</li><li>`child`: the child that has been hidden</li><li>any other param passed to the hide function</li></ul>|


<a name="controller"></a>
## Writing a Controller for DisplayContainer

An application framework such as [dapp](https://github.com/ibm-js/dapp) can decide to listen to events from 
`delite/DisplayContainer` in order to provide a controller that provides alternate/customized features.

In the following example the controller is listening to `delite-display-load` event in order to provide the ability
to load child defined in external HTML files:

```js
require(["delite/register", "delite/DisplayContainer", "dojo/request"/*, ...*/], 
  function (register, DisplayContainer, request/*, ...*/) {
  document.addEventListener("delite-display-load", function(event) {
    event.preventDefault();
    // build a file name from the destination
    request(event.dest+".html", function (data) {
       // build a child from the data in the file
       var child = a_parse_function(data);
       // resolve with the child 
       event.loadDeferred.resolve({ child: child });
    });
  });
});
```

In order to notify `delite/DisplayContainer` that the controller is handling child loading, the controller must:

  * call `preventDefault()` on the `delite-display-load` event.
  * resolve the event's `loadDeferred` deferred when it has finished loading the child
