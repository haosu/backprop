Backprop
========
[![Build Status](https://secure.travis-ci.org/af/backprop.png)](http://travis-ci.org/af/backprop)
[![NPM version](https://badge.fury.io/js/backprop.png)](http://badge.fury.io/js/backprop)

A small Backbone plugin that lets you use [ECMAScript 5 properties][ES5props] on your Backbone models.
Instead of writing:

```js
mymodel.set('name', 'Bob');
console.log(mymodel.get('name'));   // prints 'Bob'
```

with Backprop you can write this instead (and it will have the same effect):

```js
mymodel.name = 'Fred';
console.log(mymodel.name);              // prints 'Fred'
```

Backbone's `get()` and `set()` will still work if you need them. For example, you could use `set()` for it's `{validate: true}` option, although Backprop also provides a `setProperties()` method that accepts the same options as `set()` (see below).

You can install Backprop from npm with `npm install backprop`.

[ES5props]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty


Usage
-----

To use the plugin, add the following line to your app's initial config:

```js
Backprop.extendModel(Backbone.Model);
```

Then use Backprop.Model.extend (rather than Backbone's version) to create
your models, with declarative properties in the model definition. For example:

```js
var User = Backprop.Model.extend({
    name: Backprop.String(),
    numFollowers: Backprop.Number({ default: 0 })
});
```

Also, Backprop is CommonJS-friendly, so you can use it in Node or with client-side
module systems like Browserify:

```js
var Backprop = require('backprop');
```

Defining Properties
-------------------

Backprop ships with a number of functions that can be used to create properties
in your model definition:

* `Backprop.String` => ensures the property value is a string
* `Backprop.Boolean` => ensures the property value is a boolean
* `Backprop.Number` => ensures the property value is a Javascript Number
* `Backprop.Integer` => uses parseInt(x, 10) to ensure the value is an integer.
                        Note that Javascript can only safely represent [integers
                        between -2^53 and 2^53](http://www.2ality.com/2013/10/safe-integers.html).
* `Backprop.Date` => calls `new Date(x)`, passing in the assigned value as x.
                     With these properties you can assign a Date instance, Unix
                     timestamp (in milliseconds), or date string.
* `Backprop.Generic` => Performs no type coercion (you can provide your own `coerce` though)

**Note!** Previous versions of Backprop used `Backbone.property()` for defining
properties. This is no longer supported– `Backprop.Generic()` is a drop-in
replacement for these older property definitions.


Property Arguments
------------------

Each property function takes an optional hash as its only argument. The following
keys are supported to pre-filter data and make dealing with properties a bit more pleasant:

##### `default`
Lets you specify a default value for the property. This will override anything that
was set in the model's `defaults` hash for this attribute name. It's basically just a convenient
shorthand so you can keep your default value close to the property definition.

##### `coerce`
Specify a function that transforms the property’s value before it is set.
You could pass in a browser built-in like `encodeURIComponent`, or define
your own function. If you defined your property with a type-coercing function
like `Backprop.String()`, note that the type coercion will happen before your
`coerce` function sees the value.

For example:

```
var Cat = Backprop.Model.extend({
    lives: Backprop.Number({ coerce: function(x) { return x*9; })
});

var c = new Cat;
c.lives = '2';
console.log(c.lives === 18)      // prints true
```

##### `trim`
If true, calls a trim method on the value before setting the attribute. This is
handy for removing leading/trailing whitespace from strings. For obvious reasons,
you probably only want to use this with `Backprop.String()`.


##### `max` and `min`
Specify values that the value must be less than/greater than (these can be used separately
or together). Most useful for numbers, but will work with any values that work with `<` and `>`.

```js
var Beer = Backprop.Model.extend({
    milliliters = Backprop.Number({ min: 330, max: 1000 })
});
var b = new Beer;

b.milliliters = 100;
console.log(b.milliliters);     // prints 330

b.milliliters = 2000;
console.log(b.milliliters);     // prints 1000
```


##### `choices`
Specify an array with an enumeration of acceptable values for the property. If a value
not in the array is assigned to the property, the value set to the model will be:

    a) the property's current value, if one was set previously (ie. the assignment will fail)
    b) the property's `default` value (if one was speficied)
    c) `undefined`

```js
var Beer = Backprop.Model.extend({
    style: Backprop.String({ choices: ['IPA', 'stout', 'ESB'], default: 'IPA' }),
    type: Backprop.String({ choices: ['on_tap', 'bottle'] })
});

var b = new Beer;

b.style = 'asdf';
console.log(b.style);           // prints 'IPA'

b.type = 'foooo';
console.log(b.type);           // prints undefined

b.type = 'on_tap';
console.log(b.type);           // prints 'on_tap'

b.type = 'foooo';
console.log(b.type);           // prints 'on_tap'
```

The setProperties method
------------------------

Backprop also adds a `setProperties()` method to model instances. This
method accepts two positional arguments: the first is a hash of properties to set, and the
second is an options object that is passed to Backbone's `set()` method behind the scenes.
The second argument can be used to pass `{ validate: true }` or `{ silent: true }` for your
properties, while still having the data passed through Backprop's pre-filters (eg. `choices`,
`max`, `min`, etc).

```
var b = new Beer;
b.setProperties({ style: 'ESB', type: 'on_tap' }, { silent: true });
```

Note that if you pass any keys in `setProperties()`'s first hash that are not defined as
properties, they will be set on the model as regular Backbone attributes.


Compatibility
---------------

Any browser with decent ES5 support should work fine, but IE8 and below do not fit in that category.
For more complete support info see [this table](http://kangax.github.io/es5-compat-table/#Object.defineProperty).

The plugin has been tested with Backbone v1.0.0, not sure if previous releases will work correctly.


Running tests
-------------

```
git clone git://github.com/af/backprop.git
cd backprop
npm test
```


Changelog
---------
* `0.4.0` - Removed Backbone.property() and Backprop.monkeypatch(). Use
            Backprop.extendModel(Backbone.Model) instead of the latter.
* `0.3.0` - Added property shorthands like Backprop.Number, Backprop.String, etc
* `0.2.0` - Added Backbone.Model.prototype.setProperties()
* `0.1.0` - Initial release


Credits
-------

Partial inspiration came from this gist:
https://gist.github.com/dandean/1292057
