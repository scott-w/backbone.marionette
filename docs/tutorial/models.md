# Tying your views to Backbone Models

Now that we know how to build complex, nested view chains, it's time to render
information based on data set on our models.

## Creating a Model

There are two ways to do this:
  1. If we don't want any special behavior, just create an instance of a model
  2. If we want to include some business logic, extend the model

We'll start by just using the Backbone Model itself in our view. First we'll
need some data, so open up `app.js`:

```js
require('backbone').$ = require('jquery');
var Marionette = require('backbone.marionette');

var HelloView = require('./layout');

var appData = {
  first_name: 'John',
  last_name: 'Smith'
};


var App = Marionette.Application.extend({
  onStart: function(options) {
    var regions = new Marionette.RegionManager({
      regions: {
        hello: '#view-hook'
      }
    });

    var modelData = new Backbone.Model(options.appData);

    var hello = new HelloView({model: modelData});

    regions.get('hello').show(hello);
  }
});

var app = new App();
app.start({data: appData});
```

Now, without changing our `HelloView`, let's update our `hello.html` template
to take advantage of these new fields:

```html
<p>Hello, <%- first_name %> <%- last_name %>!</p>
<div id="goodbye-hook"></div>
```

Now when we build and load our app, you'll see the name set from the data. The
default template language we have set is
[underscore](https://underscorejs.org#template). Though Marionette supports most
template languages, we'll stick with underscore for this tutorial, as it is one
of Backbone's dependencies anyway.

Feel free to play around with different values and template variables. You'll
see that the names correspond exactly to the fields set on the model.

### What if we have no data?

You'll notice that if you reference a field that's not set then you won't be
able to compile the template or the app.

We can mitigate this a couple of ways:
  1. Model defaults
  2. Template helpers

#### Model defaults

Let's first do it at the model level by setting some defaults. Create a file at
`app/models/person.js`:

```js
var Backbone = require('backbone');

var Person = Backbone.Model.extend({
  defaults: {
    first_name: '',
    last_name: '',
    nickname: 'Nobody'
  }
});

module.exports = Person;
```

Now let's go back to our `app.js` file and use the new `Person` model:

```js
var Person = require('./models/person');

var HelloView = require('./layout');

var appData = {
  first_name: 'John',
  last_name: 'Smith'
};


var App = Marionette.Application.extend({
  onStart: function(options) {
    var regions = new Marionette.RegionManager({
      regions: {
        hello: '#view-hook'
      }
    });

    var modelData = new Person(options.appData);

    var hello = new HelloView({model: modelData});

    regions.get('hello').show(hello);
  }
});
```

Just to prove it works, we'll modify our template to refer to people by their
nickname:

```html
<p>Hello, <%- nickname %>!</p>
<div id="goodbye-hook"></div>
```

Now you can see the default value set from our Model. Where default fields match
data attributes, obviously the data set takes priority.

#### Using template helpers

Alternatively, we could use a template helper to check for the existence of an
attribute and return a default value if it doesn't exist. Let's open our
`layout.js` file and modify the `HelloView`:

```js
var HelloView = Marionette.LayoutView.extend({
  template: require('./hello.html'),

  regions: {
    goodbye: '#goodbye-hook'
  },

  onShow: function() {
    var goodbyeView = new GoodbyeView();
    this.goodbye.show(goodbyeView);
  },

  templateHelpers: function() {
    var model = this.model;

    return {
      get: function(name, default) {
        return model.get(name) || default || '';
      }
    };
  }
});
```

Now let's modify our `hello.html` template to call the helper:

```html
<p>Hello, <%- get(nickname, 'Nobody') %>!</p>
<div id="goodbye-hook"></div>
```

This gives us the same output as using a model default.
