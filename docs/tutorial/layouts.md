# Applications, initializers and regions

Now that we've built a simple view, let's start constructing a slightly more
complex application by nesting our views. This will let us show and hide
different views based on user input, location in the application, or any other
criteria you could come up with.

## Creating an Application

Taking our initial outline from
[creating our first view](./firstview.md#project-outline), let's move things
around a little. First, create a new file called `app/layout.js` with the
following:

```js
var Marionette = require('backbone.marionette');

var HelloView = Marionette.LayoutView.extend({
  el: '#view-hook',
  template: require('./hello.html')
});

module.exports = HelloView;
```

Your `hello.html` can remain unchanged. You'll notice that this is just the text
from `app.js` moved into a separate file. We'll modify `app.js` to look like:

```js
require('backbone').$ = require('jquery');
var Marionette = require('backbone.marionette');

var HelloView = require('./layout');


var App = Marionette.Application.extend({
  onStart: function(options) {
    var hello = new HelloView():
    hello.render();
  }
});

var app = new App();
app.start({});
```

### What have we achieved?

When you open your `index.html` file in your browser, you'll see exactly the
same output as before. So what have we achieved here?

We started by wrapping our application start inside our application. Notice the
empty object literal passed to `app.start({})` - this is where we can inject
options into our application from the template. A couple of examples would be
initial data or URLs to download the initial data to render.

Our `onStart` method is an event handler that is called when `start()` gets
executed. We also have access to an `onBeforeStart` method which runs before
anything in `app.start()` is executed.

Notice how we called our file `layout` instead of `view`: this reflects the
purpose of our `LayoutView` - a central hook to render all of our application's
templates and data.

## Laying out our views

The key component of a `LayoutView` is that it lets us attach and render
different views and different types of views. Let's create a new view and render
it from our `LayoutView`. Create a file called `goodbye.js`:

```js
var Marionette = require('backbone.marionette');


var GoodbyeView = Marionette.LayoutView.extend({
  template: require('./goodbye.html')
});


module.exports = GoodbyeView;
```

And create a template file called `goodbye.html`:

```html
<p>Goodbye, world! I'll miss you!</p>
```

Let's go back to our `hello.html` template and tell it where we're going to want
to render the `goodbye.html` template:

```html
<p>Hello, world!</p>
<div id="goodbye-hook"></div>
```

Finally, we need to tell our `LayoutView` to actually render the `GoodbyeView`
when it is rendered itself. Open up `layout.js` and change `HelloView` to look
like:

```js
var GoodbyeView = require('./goodbye');


var HelloView = Marionette.LayoutView.extend({
  el: '#view-hook',
  template: require('./hello.html'),

  regions: {
    goodbye: '#goodbye-hook'
  },

  onRender: function() {
    var goodbyeView = new GoodbyeView();
    this.goodbye.show(goodbyeView);
  }
});
```

This might seem like a lot of code to just render an extra `<p>` - but we can
use this to render lots of different types of views. For example, we can embed
lists built from collections of data, or bind different types of views to
different data models. We'll discover more about
[tying Backbone models to views](./backbone.md) shortly.
