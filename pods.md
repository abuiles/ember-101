# PODS
Until now we have organized our project files by type, so we have all the models under
`app/models`, controllers under `app/controllers`, and so on.

As [we mentioned](#pods-adapter) in the section on
adapters, **ember-cli** allows us to group things that are logically
related under a single directory. Such a structure is known as "pods".

The following shows us how the resolver tries to find the friend
adapter:

{title="Resolving the friend adapter"}
~~~~~~~~
[ ] adapter:friend .............borrowers/friend/adapter
[ ] adapter:friend .............undefined
[ ] adapter:friend .............borrowers/adapters/friend
[ ] adapter:friend .............undefined
~~~~~~~~

First it tries to find the module `adapter` under the namespace
`friend` and then moves to the namespace `adapters`.

We are currently able to structure our projects using pods or by grouping items
by their time, but the way forward is to start using pods. Ember.js
2.0 introduces the concept of Routeable Components, and it will
expect us to place some files following a pod convention.

I> For changes coming in Ember 2.0, read:
I> [The Road to Ember 2.0 RFC](https://github.com/emberjs/rfcs/pull/15)

## Using pods

Let's change our routes, controllers, and templates related to a
friend so that they are located in the pod called `app/friends`.

One easy way to find out where we should place our files is to look at the
resolver log. We can enable it by setting the property
`ENV.APP.LOG_RESOLVER` to `true` in `app/environment.js`.

The following is the lookup log for objects related to a friend:

{title="pods lookup"}
~~~~~~~~
[ ] template:friends ............. borrowers/friends/template

[ ] route:friends/index .......... borrowers/friends/index/route
[ ] controller:friends/index ..... borrowers/friends/index/controller
[ ] template:friends/index ....... borrowers/friends/index/template

[ ] route:friends/new ............ borrowers/friends/new/route
[ ] controller:friends/new ....... borrowers/friends/new/controller
[ ] template:friends/new ......... borrowers/friends/new/template

[ ] route:friends/show ........... borrowers/friends/show/route
[ ] controller:friends/show ...... borrowers/friends/show/controller
[ ] template:friends/show ........ borrowers/friends/show/template
~~~~~~~~

We can start by creating a directory called friends followed by the child
directories new, edit, index, and show.

~~~~~~~~
$ mkdir app/friends
$ mkdir app/friends/new
$ mkdir app/friends/show
$ mkdir app/friends/index
$ mkdir app/friends/edit
~~~~~~~~

With the directories in place, we can start by moving the routes:

~~~~~~~~
$ mv app/routes/friends/index.js app/friends/index/route.js
$ mv app/routes/friends/show.js app/friends/show/route.js
$ mv app/routes/friends/edit.js app/friends/edit/route.js
$ mv app/routes/friends/new.js app/friends/new/route.js
$ rm -rf app/routes/friends
~~~~~~~~

If we run our acceptance tests for friends
[http://localhost:4200/tests?module=Acceptance%3A%20FriendsNew](http://localhost:4200/tests?module=Acceptance%3A%20FriendsNew),
everything should work.

Next let's move the templates:

~~~~~~~~
$ mv app/templates/friends/index.hbs app/friends/index/template.hbs
$ mv app/templates/friends/show.hbs app/friends/show/template.hbs
$ mv app/templates/friends/edit.hbs app/friends/edit/template.hbs
$ mv app/templates/friends/new.hbs app/friends/new/template.hbs
~~~~~~~~

On the edit and new template, we are using a partial called "form." The
pods loookup for partials expects the template to be located in the
following file: `borrowers/friends/-form/template`. Let's move it there:

~~~~~~~~
$ mkdir app/friends/-form
$ mv app/templates/friends/-form.hbs app/friends/-form/template.hbs
$ rm -rf app/templates/friends
~~~~~~~~

Now the controllers:

~~~~~~~~
$ mv app/controllers/friends/edit.js app/friends/edit/controller.js
$ mv app/controllers/friends/new.js app/friends/new/controller.js
$ mv app/controllers/friends/base.js app/friends/base-controller.js
~~~~~~~~

Notice that we moved the base controller to
`app/friends/base-controller.js`. We need to update the references in
`app/friends/edit/controller.js` and `app/friends/new/controller.js`.


Instead of:

~~~~~~~~
import FriendsBaseController from './base';
~~~~~~~~

We need:

~~~~~~~~
import FriendsBaseController from '../base-controller';
~~~~~~~~

If we update the browser, everything should work. We are now
using pods for our friends.

X> ## Tasks
X> Change the structure for articles so that everything is under the pod
X> `articles`.
