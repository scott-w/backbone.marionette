# Route handling

If we want to build a single-page application, we need to be generate and
respond to URLs. One of the key advantages of the web over other platforms is
the ease of which we can share and bookmark information through the URL.
Because of this, Backbone has a powerful route-handling framework for responding
to URLs as they come in.

## Routers/Controllers - The C in MVC

Web frameworks in most languages include a routing system - Django's `urls.py`
and Flask's route decorators are two prominent examples of different styles.

The approach taken by Backbone and Marionette is slightly different, reflecting
nature of existing state in client-side applications. So, how and why do we use
routing?

The purpose of the router is to restore state on page load with help from the
browser's `window.location` property. We can either construct the state directly
from the URL or we can use it to start pulling extra data from the server. In
our tutorial, we'll construct information from the URL directly.

We use `Backbone.history` to set points to restore from later. If we wanted to
build a single-page app, we can use the Router and History to setup a
navigate-able system. A common example could be `/items` list where clicking an
item would navigate to `/items/id` and render the item we just clicked.

## Creating routes

Let's get started by creating a `router.js` file and set up some simple routes.
We'll alter our collection view so we can click on names to get a little more
detail about each name.

```js
var Marionette = require('backbone.marionette');

var HelloView = require('./layout');
var PersonView = require('./person');


var controller = {
  listNames: function() {
    var helloView = new HelloView({
      collection: this.collection,
      model: this.model
    });

    this.regions.get('hello').show(helloView);
  },

  displayName: function(id) {
    var personView = new PersonView({
      collection: this.collection,
      model: this.model
    });

    this.regions.get('hello').show(personView);
  }
};

var Router = Marionette.AppRouter.extend({
  controller: controller,

  initialize: function(options) {
    this.regions = options.regions;
    this.controller = options.controller;
    this.model = options.model;
  },

  appRoutes: {
    '': 'listNames',
    'name/:id': 'displayName'
  }
});

module.exports = Router;
```

We can see that our initial view creation has been moved into our controller, we
need to adapt our `app.js` to account for it:

```js
require('backbone').$ = require('jquery');
var Marionette = require('backbone.marionette');

var Person = require('./models/person');
var PersonList = require('./collections/person');

var Router = require('./router');  // 1

var modelData = {
  first_name: 'John',
  last_name: 'Jones'
};

var listData = [
  {
    first_name: 'Dave',
    last_name: 'Jones'
  },
  {
    first_name: 'Steve',
    last_name: 'Hansen'
  }
]


var App = Marionette.Application.extend({
  onStart: function(options) {
    var regions = new Marionette.RegionManager({
      regions: {
        hello: '#view-hook'
      }
    });

    var modelData = new Person(options.model);
    var collectionData = new PersonList(options.collection);

    this.router = new Router({
      collection: collectionData,
      model: modelData,
      regions: regions
    });  // 2

    Backbone.history.start();  // 3
  }
});

var app = new App();
app.start({model: modelData, collection: listData});
```

The changes are as follows:

  1. We import our Router
  2. We pass all the information into our Router to be managed there
  3. We start the Backbone history
