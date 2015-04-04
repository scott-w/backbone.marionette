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
