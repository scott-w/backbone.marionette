# Collections and Views

The `Marionette.CollectionView` is used to render elements in a
`Backbone.Collection`. Unlike a `View`, the `CollectionView` isn't able to
render a template on its own - instead rendering a `childView` for each `model`
in `collection`.

## Document Index

  1. Rendering collections
  2. Callback methods
  3. Migrating from Marionette 2 `CompositeView`

## 1. Rendering collections

To render a collection in a `CollectionView`, define a `childView` attribute
defining your class:

```javascript
var Marionette = require('backbone.marionette');

var Child = Marionette.View.extend({
  template: require('template.html')
});

var List = Marionette.CollectionView.extend({
  childView: Child
});
```

To render a collection, we instantiate the view with a collection attribute:

```javascript
var myList = new List({
  collection: new Backbone.Collection([{key: 'val'}, {key: 'val'}]);
});

region.show(myList);
```

## 2. Callback methods

<!-- Todo -->

## 3. Migrating from Marionette 2 `CompositeView`

Users upgrading from Marionette 2.x will notice that CompositeView is now
deprecated and will be removed in Marionette 4. The reason for this is that
Regions now allow us to overwrite the Element instead of following the classic
Backbone behavior of adding an extra `el`.

### Upgrading

Upgrading from `CompositeView` requires adding a parent `View` layout and
rendering in a region.

Take the following CompositeView from Marionette 2:

```javascript
var Marionette = require('backbone.marionette');

var Child = Marionette.LayoutView.extend({
  template: require('child.html')
});

var List = Marionette.CompositeView.extend({
  template: require('layout.html'),
  childView: Child,
  childViewContainer: 'tbody'
});
```

The equivalent in Marionette 3:

```javascript
var Marionette = require('backbone.marionette');

var Child = Marionette.View.extend({
  template: require('child.html')
});

var List = Marionette.CollectionView.extend({
  childView: Child
});

var Layout = Marionette.View.extend({
  template: require('layout.html'),

  regions: {
    list: {
      selector: 'tbody',
      replaceElement: true
    }
  },

  onRender: function() {
    this.showChildView('list', new List({collection: this.collection}));
  }
});
```

While this approach requires more code to set up, it provides a far more
flexible approach to rendering complex layout schemes. For instance, it is now
possible to render multiple lists inside a single layout container such as a
table.
