**_These docs are for Marionette 3 which is still in pre-release. Some parts may
not be accurate or up-to-date_**

# Marionette.Object

The Marionette Object is a common base class that all Marionette classes can
extend from to provide common APIs and behaviors. This section outlines the
possible behaviors of `Object` and links through to the relevant sections for
more detail.

## Documentation Index

* [Initializing an Object](#initializing-an-object)
* [Events](#events)
* [Destroying An Object](#destroying-a-object)
* [Object Methods](#object-methods)
  * [`mergeOptions`](#mergeoptions)
  * [`getOption`](#getoption)
  * [`bindEntityEvents`](#bindentityevents)
* [Basic Use](#basic-use)


## Initializing an Object

Whenever an Object is instantiated with `new`, the `initialize` method is
called with the same arguments passed on instantiation:

```javascript
var Mn = require('backbone.marionette');

var Friend = Mn.Object.extend({
  initialize: function(options){
    console.log(options.name);
  }
});

var john = new Friend({name: 'John'});
```

## Events

`Marionette.Object` extends `Backbone.Events` and includes `triggerMethod`.
This makes it easy for Objects to emit events that other objects can listen for
with `on` or `listenTo`.

```javascript
var Mn = require('backbone.marionette');

var Friend = Mn.Object.extend({
  graduate: function() {
    this.triggerMethod('announce', 'I graduated!!!');
  }
});

var john = new Friend({name: 'John'});

john.on('announce', function(message) {
  console.log(message); // I graduated!!!
});

john.graduate();
```

For more information, see the [Documentation for Events](marionette.events.md).

## Radio Events

`Marionette.Object` integrates with `Backbone.Radio` to provide powerful
messaging capabilities.  Objects can respond to both of Radio's message types;
`Events` and `Requests`.  The syntax is similar to the `events` syntax from
Backbone Views, and looks like this:

```javascript
var Mn = require('backbone.marionette');

var CustomObject = Mn.Object.extend({
  radioEvents: {
    'app start': 'onAppStart',
    'books finish': 'onBooksFinish',
  },

  radioRequests: {
    'resources bar': 'getResources',
  },

  getBar: function() {
    //
  }
});
```

See the [Documentation for Backbone Radio](backbone.radio.md) for more
information.

## Object Methods

Marionette objects provide a number of methods that can be used throughout your
application.

### `mergeOptions`

Merge keys from the `options` object directly onto the instance. This is the
preferred way to access set options onto the Object

See the [Common Marionette Concepts](basics.md#the-mergeoptions-method) for a
more complete description of how to use `mergeOptions`.

### `getOption`

Retrieve options set on an object at instantiation or with the `mergeOptions`
method. This is the preferred way to access the internal `this.options`
attribute.

See the [Common Marionette Concepts](basics.md#the-getoptions-method) for a
more complete description of how to use `getOption`.

### `bindEntityEvents`

Helps bind a backbone "entity" to methods on a target object. More information
[bindEntityEvents](./marionette.functions.md#marionettebindentityevents).

## Destroying A Object

Objects have a `destroy` method that unbind the events that are directly
attached to the instance.

Invoking the `destroy` method will trigger a "before:destroy" event and
corresponding `onBeforeDestroy` method call. These calls will be passed any
arguments `destroy` was invoked with. Invoking `destroy` will return the object,
this can be useful for chaining.

```javascript
var Mn = require('backbone.marionette');

// define a object with an onDestroy method
var MyObject = Mn.Object.extend({

  onBeforeDestroy: function(arg1, arg2){
    // put custom code here, to destroy this object
  }

});

// create a new object instance
var obj = new MyObject();

// add some event handlers
obj.on("before:destroy", function(arg1, arg2){ ... });
obj.listenTo(something, "bar", function(){...});

// destroy the object: unbind all of the
// event handlers, trigger the "destroy" event and
// call the onDestroy method
obj.destroy(arg1, arg2);
```

## Basic Use

Selections is a simple Object that manages a selection of things.
Because Selections extends from Object, it gets `initialize` and `Events`
for free.

```javascript
var Mn = require('backbone.marionette');
var Filters = require('./selections/filters');

var Selections = Mn.Object.extend({
  initialize: function(){
    this.selections = {};
  },

  select: function(key, item){
    this.triggerMethod("select", key, item);
    this.selections[key] = item;
  },

  deselect: function(key, item) {
    this.triggerMethod("deselect", key, item);
    delete this.selections[key];
  }

});

var selections = new Selections({
  filters: Filters
});

// use the built in EventBinder
selections.listenTo(selections, "select", function(key, item) {
  console.log(item);
});

selections.select('toy', Truck);
```
