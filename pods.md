# PODS
Until know we have been organizing the files in our project in such
way that they are grouped by type, so we have all the models under
`app/models`, controllers under `app/controllers` and so on.

As [we mentioned](#pods-adapter) in the chapter where we talked about
adapters, **ember-cli** allows us to group things which are logically
related under a single directory, such structure is known as "pods".

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

Currently we can structure our projects using pods or grouping things
by their time, but the way forward is to start using pods, Ember.js
2.0 will introduce the concept of Routeable Components and it will
expect us to place some files following a pod convention.

I> For changes coming on Ember 2.0 read:
I> [The Road to Ember 2.0 RFC](https://github.com/emberjs/rfcs/pull/15)

## Using pods

Let's change our routes, controller and templates related with a
friend so they are located in the pod called `app/friends`.

An easy way to know where should we place our files is to look at the
resolver log, we can enable it setting the property
`ENV.APP.LOG_RESOLVER` to `true` in `app/environment.js`.

The following is the lookup log for things related with a friend:

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

We can start by creating the directory friends and create the child
directories new, edit, index, and show.

~~~~~~~~
$ mkdir app/friends
$ mkdir app/friends/new
$ mkdir app/friends/show
$ mkdir app/friends/index
$ mkdir app/friends/edit
~~~~~~~~

With the directories in place we can start by moving the routes:

~~~~~~~~
$ mv app/routes/friends/index.js app/friends/index/route.js
$ mv app/routes/friends/show.js app/friends/show/route.js
$ mv app/routes/friends/edit.js app/friends/edit/route.js
$ mv app/routes/friends/new.js app/friends/new/route.js
$ rm -rf app/routes/friends
~~~~~~~~

If we run our acceptance tests for friends
[http://localhost:4200/tests?module=Acceptance%3A%20FriendsNew](http://localhost:4200/tests?module=Acceptance%3A%20FriendsNew)
everything should work.

Next let's move the templates:

~~~~~~~~
$ mv app/templates/friends/index.hbs app/friends/index/template.hbs
$ mv app/templates/friends/show.hbs app/friends/show/template.hbs
$ mv app/templates/friends/edit.hbs app/friends/edit/template.hbs
$ mv app/templates/friends/new.hbs app/friends/new/template.hbs
~~~~~~~~

On the edit and new template we are using a partial called "form", the
pods loookup for partials expect the template to be located in the
following file: `borrowers/friends/-form/template`, let's move it there.

~~~~~~~~
$ mkdir app/friends/-form
$ mv app/templates/friends/-form.hbs app/friends/-form/template.hbs
$ rm -rf app/templates/friends
~~~~~~~~

And then the controllers

~~~~~~~~
$ mv app/controllers/friends/edit.js app/friends/edit/controller.js
$ mv app/controllers/friends/new.js app/friends/new/controller.js
$ mv app/controllers/friends/base.js app/friends/base-controller.js
~~~~~~~~

Notice that we moved the base controller to be in
`app/friends/base-controller.js`, we need to update the reference in
`app/friends/edit/controller.js` and `app/friends/new/controller.js`.


Instead of:

~~~~~~~~
import FriendsBaseController from './base';
~~~~~~~~

We need to import:

~~~~~~~~
import FriendsBaseController from '../base-controller';
~~~~~~~~

Now if we update the browser, everything should work and we are now
using pods for our friends.

X> ## Tasks
X> Change the structure for articles so everything is under the pod
X> `articles`.
