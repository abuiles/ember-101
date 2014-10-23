# Computed Properties and Observers

We already covered computed properties and use them in different parts
of our applications, one of them was on the friend model:

{title="app/models/friend.js", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';
import Ember from 'ember';

export default DS.Model.extend({
  // ...
  fullName: Ember.computed('firstName', 'lastName', function() {
    return this.get('firstName') + ' ' + this.get('lastName');
  })
});
~~~~~~~~

With the code above we created a new property on the model called
**fullName** which depends on **firstName** and **lastName**, the
computed properties will be called once at the beginning and then the result will be
cached until any of the dependent properties change.

Next we'll like to talk about a couple of features and things to keep
in mind when defining computed properties.

## An alternative syntax for computed properties

Ember extends the **function** prototype with the function
**property** to allow us to specify computed properties in a
different way, instead of using `Ember.computed` we can specify a
computed property like the following:

{title="CP in app/models/friend.js via .property function", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';
import Ember from 'ember';

export default DS.Model.extend({
  // ...
  fullName: function() {
    return this.get('firstName') + ' ' + this.get('lastName');
  }.property('firstName', 'lastName');
});
~~~~~~~~

We achieve the same result using the `prototype` extension or `Ember.computed`, what to use is
just a matter of taste, some people like `Ember.computed` and others
`.property`.

We need to keep in mind though that we can use `.property` only
because prototype extensions are enabled by default, if we decide to
turn them off then we wouldn't be able to use this functionality.

I> Ember guides have a section about disabling prototype
I> extensions, if we are thinking about turning them off we should give
I> it a read and understand the implications: [Disabling prototype extensions](http://emberjs.com/guides/configuring-ember/disabling-prototype-extensions).

## Computed Property function signature

The functions we have been using to declare a computed property have
looked like the following:

{title="Computed Property Function", lang="JavaScript"}
~~~~~~~~
fullName: function() {
  return this.get('firstName') + ' ' + this.get('lastName');
}.property('firstName', 'lastName');
~~~~~~~~

Using the previous signature in the **function** we passed to
`Ember.computed` or `.property` we get computed properties working,
but we can optionally specify it like the following:

{title="Computed Property Function", lang="JavaScript"}
~~~~~~~~
fullName: function(key, value, oldValue) {
  return this.get('firstName') + ' ' + this.get('lastName');
}.property('firstName', 'lastName');
~~~~~~~~

With that we can add support for setting the value of a computed
property and handle  how it should behave, the following is extracted
from the Ember documentation where they use **firstName** and
**lastName** too:

{title="Computed Property with set support, lang="JavaScript"}
~~~~~~~~
fullName: function(key, value, oldValue) {
  if (arguments.length === 1) {
    //
    // Works as getter
    //

    return this.get('firstName') + ' ' + this.get('lastName')
  } else {

    //
    // Works as setter
    //

    var name = value.split(' ');

    this.set('firstName', name[0]);
    this.set('lastName', name[1]);

    return value;
  }
}.property('firstName', 'lastName')
~~~~~~~~

I> For the curious the following class has the implementation for [computed property](https://github.com/emberjs/ember.js/blob/v1.7.0/packages/ember-metal/lib/computed.js#L78).


Why did we avoid mentioning that we can use a computed property as
setter? The reason is that this is a very uncommon scenario which tends to
cause a lot of confusion on people, ideally we should use computed
properties as Read-Only. In a later version of Ember this might be the
default, there is an issue created by
[Stefan Penner](https://twitter.com/stefanpenner) which aims to make
computed properties Read-Only by default: [default readOnly
CP #9290](https://github.com/emberjs/ember.js/issues/9290).


## Computed Properties gotchas

Computed properties and observers are normally fired whenever we call
set on the property they depend on, the downside of this is that they
will be recalculated even if the value is the same, also if we are
using version 1.7 of Ember there are some bugs that cause computed
properties and observers to misbehave under some circumstances.

Some of the bugs have been fixed in  the upcoming version of
Ember (1.8) but computed properties and observes are still being
called even if the property didn't change.


Fortunately for us, [Gavin Joyce](https://twitter.com/gavinjoyce)
wrote an **ember-cli-addon** called
[ember-computed-change-gate](https://github.com/GavinJoyce/ember-computed-change-gate)
which offers an alternative function to define computed properties
and fixes observers such that they'll be only called if the property
they depend on have changed.

We can install the addon with `npm i ember-computed-change-gate
--save-dev` and use it in our friends model like the following:

{title="Using ember-computed-change-gate in app/models/friend.js", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';
import Ember from 'ember';
import changeGate from 'ember-computed-change-gate/change-gate';

export default DS.Model.extend({
  //
  // Currently changeGate only support one property
  //
  capitalizedFirstName: changeGate('firstName', function(firstName) {
    return Ember.String.capitalize(firstName);
  })
});
~~~~~~~~

With that our computed property `capitalizedFirstName` will be called
only when the value of the dependent key has actually changed to a
different value.

## Observers

Ember has a built-in implementation of the
[Observer pattern](http://en.wikipedia.org/wiki/Observer_pattern),
which allows us to keep track of changes in any property or
computed property.

We used observers to implement auto saving in the articles item controller
with the following:

{title="controllers/articles/item.js", lang="JavaScript"}
~~~~~~~~
  isDirtyChanged: function() {
    if (this.get('isDirty') && !this.get('isSaving')) {
      Ember.run.once(this, this.autoSave);
    }
  }.on('init').observes('isDirty')
~~~~~~~~

We define an observer with the prototype extension `.observes` which
receive any number of properties to observe, then when any of the
properties is set the function is called automatically.

The pattern `.on('init').observes('propertyName')` is very common to
make sure the observer is enabled, by default observers are not
switched on until the function where they are being defined is called,
if we define the observer as follows:

{title="controllers/articles/item.js", lang="JavaScript"}
~~~~~~~~
  isDirtyChanged: function() {
    if (this.get('isDirty') && !this.get('isSaving')) {
      Ember.run.once(this, this.autoSave);
    }
  }.observes('isDirty')
~~~~~~~~

Then the observer won't have any effect until the function
`isDirtyChanged` is called, to make sure the observer is enable we use
`on('init')` which calls the function as soon as the object where the
function is define get's created. In our example that would be when an
instance of `controllers/articles/item.js` is created.


We can also create an observer using `addObserver` from
[Ember.Observable](http://emberjs.com/api/classes/Ember.Observable.html),
we could have define the `isDirtyChanged` observer like:

{title="controllers/articles/item.js", lang="JavaScript"}
~~~~~~~~
  setObserver: function() {
    this.addObserver('isDirty', this, this.isDirtyChanged);
  }.on('init'),
  isDirtyChanged: function() {
    if (this.get('isDirty') && !this.get('isSaving')) {
      Ember.run.once(this, this.autoSave);
    }
  }
~~~~~~~~

## Observing collections

Ember adds two convenient properties to collections which we can use
if we want to observer changes any of the member's properties
properties or if we want to do something every time an element is
added or removed.

The first property is
[.[]](http://emberjs.com/api/classes/Ember.Array.html#property__)
which is just a special handler which changes every time the
collection content change.

The second one is
[@.each](http://emberjs.com/api/classes/Ember.Array.html#property__each)
which allows to observer properties on every of the items in the
collection.

We can use the previous function in our articles index to call a
function if we add a new articles and then other one when we change
the state of an article:


{title="app/articles/index.js" lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.ArrayController.extend({
  contentDidChange: function() {
    console.log('Called when we add or removed an article.');
  }.observes('model.[]'),
  stateDidChange: function() {
    console.log('Called when the state property change for any of the articles.');
  }.observes('model.@each.state')
});
~~~~~~~~

If we visit any of our friends profile and change the state for any
article or add a new one, we'll see the messages in the browser's
console.
