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

## A simpler app

We've really stretched the hello world as far as we can take it, so we're going
to cut it back a little to make it easier to demo the router features.

Let's get started by creating a `router.js` file and set up some simple routes.
We'll alter our collection view so we can click on names to get a little more
detail about each name.

```js
var Marionette = require('backbone.marionette');


var controller = {  // 1
  listNames: function() {  // 2
    this.app.listNames();
  },

  displayName: function(id) {  // 3
    var model = this.collection.get(id);
    this.app.displayName(model);

  }
};

var Router = Marionette.AppRouter.extend({
  controller: controller,  // 4

  initialize: function(options) {  // 5
    this.app = options.app;
    this.collection = options.collection;
  },

  appRoutes: {  // 6
    '': 'listNames',
    'name/:id': 'displayName'
  }
});

module.exports = Router;
```

Let's break down the router and controller at the most interesting points:

  1. We create a basic object to be our `controller`.
  2. We name some methods as functions, which we'll use later.
  3. We can specify some arguments that the route can pass in.
  4. We specify the `controller` to use on the `AppRouter`.
  5. As with other Backbone and Marionette features, we can pass options.
  6. We define our `appRoutes` as a key-value hash of route to method.

### Using `appRoutes`

We use `appRoutes` to map URLs to methods. We use the fragment (`#`) as our
client-side URL - everything after the fragment is parsed as the total URL. We
can pass variables in from the URL using the `:<var_name>` format as we can see
in the route for `displayName` that will match `name/15`, for example.

An important consideration for routers is that the routes themselves must not
run any initialization logic - this must stay in `initialize`. This is because
the routes are triggered on back/forward which, in fragments, doesn't cause a
page reload. This would attempt to initialize our app twice!

### Carrying on

Now that we have our Router, we'll use it to manage our page loads and start
creating our views:

```js
require('backbone').$ = require('jquery');
var Marionette = require('backbone.marionette');

var PersonList = require('./collections/person');

var IndexView = require('./views/personlist');
var PersonView = require('./views/person');

var Router = require('./router');  // 1

var listData = [
  {
    id: 1,
    first_name: 'Dave',
    last_name: 'Jones'
  },
  {
    id: 2,
    first_name: 'Steve',
    last_name: 'Hansen'
  }
]


var App = Marionette.Application.extend({
  onStart: function(options) {
    this.regions = new Marionette.RegionManager({
      regions: {
        root: '#view-hook'
      }
    });

    this.collection = new PersonList(options.collection);

    var router = new Router({
      app: this,
      collection: this.collection
    });  // 2

    Backbone.history.start();  // 3
  },

  listNames: function() {
    var view = new IndexView({
      app: this,
      collection: this.collection
    });

    this.regions.get('root').show(view);
  },

  displayName: function(model) {
    var view = new PersonView({
      app: this,
      model: model
    });

    this.regions.get('root').show(view);
  }
});

var app = new App();
app.start({collection: listData});
```

The changes are as follows:

  1. We import our Router.
  2. We pass our app into the `router` so we can access views when routing.
  3. We start the `Backbone.history` which will trigger the route-handling.

### The views

For the purposes of this tutorial, we'll skip over the templates themselves and
focus on the view handling. Let's start with our list index in
`views/personlist.js`:

```js
var Marionette = require('backbone.marionette');


var PersonView = Marionette.LayoutView.extend({
  tagName: 'li',
  template: require('../templates/person/item.html'),

  triggers: {
    'click': 'select:person'
  }
});


var PersonList = Marionette.CollectionView.extend({
  childView: PersonView,
  tagName: 'ul',
  template: require('../templates/person/list.html'),

  onChildviewSelectPerson: function(child, options) {
    this.options.app.displayName(child.model);  // 1
    Backbone.history.navigate('name/' + model.id);  // 2
  }
})


module.exports = PersonList;
```

There are two interesting parts in our `PersonList`:

  1. Since we passed the `app` into our view, we can its methods directly.
  2. We then navigate to the new URL.

### What does `Backbone.history.navigate` do?

The `navigate` method is a simple way to update the fragment with the new URL.
It's important to note that we don't execute any extra code using this. Its only
purpose is to update the page state so we can either retrieve it, or access it
with the back/forward buttons on the browser. We could forcibly execute that
code using `navigate` but it's best not to if we can avoid it.

### The detail view

Our detail view is pretty straightforward and just contains a way to return to
the list. Let's open up `views/person.js` and get started:

```js
var Marionette = require('backbone.marionette');


var PersonView = Marionette.LayoutView.extend({
  template: require('../templates/person/detail.html'),

  ui: {
    back: '.back'
  },

  triggers: {
    'click @ui.back': 'click:back'
  },

  onClickBack: function() {
    this.options.app.listNames();
    Backbone.history.navigate('');
  }
});


module.exports = PersonView;
```

As an aside, you can quickly see the advantages of using the `ui` hash - we
don't know what exactly `back` looks like in HTML but we can decide that later
and not worry about having to make lots of niggly fixes later.
