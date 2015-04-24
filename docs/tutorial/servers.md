# Integrating with a Web Server

After understanding how we can build applications that we can navigate, interact
with and list data through, it's time to pull it all together by integrating
with a web server.

## Server API

Before going any further, we need to define an imaginary API that we'll use.
For the uninitiated, a web service API is a way of communicating with our web
application so we can save state and access it from other machines connected to
the internet. Common tools for building web application APIs are
[Ruby on Rails](http://rubyonrails.org/) and
[Django REST Framework](http://django-rest-framework.org/), though there are
plenty available.

We'll define two endpoints and assume we're serving the API from the same host
as our web application:

`/people` returns the following JSON list:

```js
[
  {
    "id": 1,
    "first_name": "Steve",
    "last_name": "Jones",
    "url": "/people/1"
  },
  {
    "id": 4,
    "first_name": "James",
    "last_name": "Haskell",
    "url": "/people/4"
  }
]
```

In addition, we can issue a POST request to this URL with the following JSON
object to create a new person:

```js
{
  "first_name": "Jack",
  "last_name": "Doe"
}
```

`/people/:id` returns a JSON object matching the ID, if passed 1:

```js
{
  "id": 1,
  "first_name": "Steve",
  "last_name": "Jones",
  "url": "/people/1"
}
```

We can issue a PUT request against any endpoint with the following JSON
structure to update the identified person:

```js
{
  "first_name": "Jack",
  "last_name": "Doe"
}
```

## Integrating our Application

Now we have our API defined, we can start to work with it! Now, instead of
pre-defining a list when we load our page, we can just define our URLs on our
collections and Backbone will work everything out for us.

Our application structure is identical to before, so we'll start by amending our
`collections/person.js`:

```js
var Backbone = require('backbone');

var Person = require('../models/person');


var PersonCollection = Backbone.Collection.extend({
  model: Person,

  url: '/people'
});

module.exports = PersonCollection;
```

Now we'll change our `app.js` file to remove references to existing data:

```js
require('backbone').$ = require('jquery');
var Marionette = require('backbone.marionette');

var Router = require('./router');


var App = Marionette.Application.extend({
  onStart: function(options) {
    var router = new Router();

    Backbone.history.start();
  }
});

var app = new App();
app.start();
```

Finally we'll amend our `router.js` so we get the data from the server:

```js
var Marionette = require('backbone.marionette');

var PersonList = require('./collections/person');

var IndexView = require('./views/personlist');
var PersonView = require('./views/person');


var Controller = Marionette.Object.extend({
  initialize: function() {
    this.regions = new Marionette.RegionManager({
      regions: {
        root: '#view-hook'
      }
    });

    this.collection = new PersonList();  // 1
    this.collection.fetch();  // 2
  },

  listNames: function() {
    var view = new IndexView({
      collection: this.collection,
      controller: this
    });

    this.regions.get('root').show(view);  
  },

  displayName: function(id) {
    var model = this.collection.get(id);

    var view = new PersonView({
      controller: this,
      model: model
    });

    this.regions.get('root').show(view);
  }
});

var Router = Marionette.AppRouter.extend({
  controller: new Controller(),  // 3

  appRoutes: {
    '': 'listNames',
    'name/:id': 'displayName'
  }
});

module.exports = Router;
```

We have made a few changes to simplify this:

  1. We no longer pass any initial data into the Collection.
  2. We execute `fetch()` to pull this data from the server.
  3. We can just define a `new Controller()` directly on the `AppRouter`.

When we load our application, the data will be fetched from the server and our
list view will automatically render when the data is fetched. Also notice how
the URLs on the router don't need to be associated in any way with the router's
URL scheme. We can easily use multiple collections/models and blend the data
together in different views under our application hierarchy.

## Navigating directly to a person

If we navigate directly to `#name/1` you'll notice it doesn't work at all! This
is because, when the controller executes, the collection is empty, so
`this.collection.get(1);` returns `undefined` and we just get a cascade of
errors.

Let's have the view always grab the data from the server before it attempts to
render as a first solution:

```js
var Controller = Marionette.Object.extend({
  initialize: function() {
    this.regions = new Marionette.RegionManager({
      regions: {
        root: '#view-hook'
      }
    });

    this.collection = new PersonList();  
    this.collection.fetch();  
  },

  listNames: function() {
    var view = new IndexView({
      collection: this.collection,
      controller: this
    });

    this.regions.get('root').show(view);  
  },

  displayName: function(id) {
    var model = new Person({id: id});

    var view = new PersonView({
      controller: this,
      model: model
    });

    this.listenTo(model, 'sync', function() {
      this.regions.get('root').show(view);
    });
    model.fetch();
  }
});
```

We just need to amend our `models/person.js`:

```js
var Backbone = require('backbone');


var Person = Backbone.Model.extend({
  urlBase: '/person'
});

module.exports = Person;
```

With this in place, navigating to `#name/1` won't display anything until the
model has loaded from the server. There is a downside to this, however, in that
we no longer attempt to get the model from the collection, even though we might
have all its data sitting in memory ready to use!

## A more effective solution

Let's use the model defaults and events to get the right balance between
performance and correct behavior.

We'll go back to our `models/person.js` to start:

```js
var Backbone = require('backbone');


var Person = Backbone.Model.extend({
  defaults: {
    first_name: '',
    last_name: ''
  }
  urlBase: '/person'
});

module.exports = Person;
```

With our defaults in place, we can safely render our view with an empty `Person`
and backfill its details later. Let's open up `router.js` and do this:

```js
var Controller = Marionette.Object.extend({
  initialize: function() {
    this.regions = new Marionette.RegionManager({
      regions: {
        root: '#view-hook'
      }
    });

    this.collection = new PersonList();  
    this.collection.fetch();  
  },

  listNames: function() {
    var view = new IndexView({
      collection: this.collection,
      controller: this
    });

    this.regions.get('root').show(view);  
  },

  displayName: function(id) {
    var model = this.collection.get(id);
    if (_.isUndefined(model)) {
      model = new Person({id: id});  // 1
      model.fetch();
    }

    var view = new PersonView({
      controller: this,
      model: model
    });

    this.regions.get('root').show(view);
  }
});
```

  1. Our only major change is to try to grab the item before creating it from
  scratch

Our view will now, most likely, be rendered as empty and never change. Let's
open up `views/person.js` and add our event handler so it will re-render when
`fetch()` completes:

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

  modelEvents: {
    'sync': 'render'
  },

  onClickBack: function() {
    Backbone.history.navigate('');
    this.options.controller.listNames();
  }
});


module.exports = PersonView;
```

Now, when the `fetch()` completes, `render()` will be called and the view will
be redrawn with the data from the server. And, even better, if we navigated
there from the root URL, the page load will be instant! We could even move the
`model.fetch()` call outside the `if()` block and have our view refreshed with
the latest data if we think it'll change.

## Saving user data

Now let's start recording user-entered data with the server. For this section,
we'll look at sample template fragments that we could use.
