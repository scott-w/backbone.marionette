# Collections and advanced views

Now we're comfortable rendering templates from model attributes, let's start
rendering lists of information from Backbone Collections.

## What is a Collection?

The [Backbone Collection](https://backbonejs.org/#Collection) is a list of
Backbone Models with some helper methods to allow us to abstract over the items
in the collection.

Like models, Collections can be bound to views for rendering information.
Marionette includes a special class of views called `CollectionView` which
iterates over Collections and renders a `childView` for each model.

## Rendering a list

Let's get a list of items displayed. We'll take the `Person` model from our
[earlier tutorial on models](./models.md) and render a list of people. Let's
start by defining a Collection in `apps/collections/person.js`:

```js
var Backbone = require('backbone');

var Person = require('../models/person');


var PersonCollection = Backbone.Collection.extend({
  model: Person
});

module.exports = PersonCollection;
```

We now want a `CollectionView` that will render a message for each person in
`app/personlist.js`:

```js
var Marionette = require('backbone.marionette');

var PersonView = Marionette.LayoutView.extend({
  tagName: 'li',
  template: require('./templates/person.html')
});


var PersonList = Marionette.CollectionView.extend({
  tagName: 'ul',
  childView: PersonView
});
```

We just need a template to render in `templates/person.html`:

```html
Hi <%- first_name %>!
```

Now let's open up our `app.js`, create a list that we'll render, and attach our
new `CollectionView`:

```js
require('backbone').$ = require('jquery');
var Marionette = require('backbone.marionette');

var Person = require('./models/person');
var PersonList = require('./collections/person');

var HelloView = require('./layout');

var modelData = {
  first_name: 'John',
  last_name: 'Smith'
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

    var hello = new HelloView({
      collection: collectionData,
      model: modelData
    });

    regions.get('hello').show(hello);
  }
});

var app = new App();
app.start({model: modelData, collection: listData});
```

Now let's go into our `HelloView` and `hello.html` to put the last pieces into
place to render everything:

```js
var PersonView = require('./personlist');


var HelloView = Marionette.LayoutView.extend({
  template: require('./hello.html'),

  regions: {
    goodbye: '#goodbye-hook',
    greetings: '#greeting-hook'
  },

  onShow: function() {
    var goodbyeView = new GoodbyeView();
    var personView = new PersonView({collection: this.collection});

    this.goodbye.show(goodbyeView);
    this.greetings.show(personView);
  }
});
```

```html
<p>Hello, <%- first_name %> <%- last_name %>!</p>
<div id="greeting-hook"></div>
<div id="goodbye-hook"></div>
```

Now your template should render a list of greetings to people in the list.
