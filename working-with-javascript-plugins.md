# Working with JavaScript plugins

In this chapter we'll learn how to write Ember helpers that can be
consumed in our templates. To do so, we'll write a helper called
**formatted-date** that will show the date when an
article was borrowed. Instead of showing **Sun Sep 28 2014 04:58:30
GMT-0500**, we'll see **September 28, 2014**.

We'll implement **formatted-date** using
[Momentjs](http://momentjs.com), a library that facilitates working
with dates in JavaScript.

## Installing moment

Remember that ember-cli uses Bower to manage frontend dependencies.
Here we'll use the same pattern used to install **picnicss**: we'll
add **moment** to Bower and then use **app.import** in our
**Brocfile.js**.

I> We can also install front-end dependencies via npm if they are
I> packed as addons. We'll learn more about this in a later chapter.


First, we install **moment**:

~~~~~~~~
$ bower install moment --save
~~~~~~~~

The option `--save` adds the dependency to our **bower.json**. We
should find something similar to **"moment": "~2.8.3"** (the version
might be different).

Next, let's import **moment**. To find out which file to import, let's go
to **bower_components/moment/**. We'll see that it contains a
**moment.js** file that is the non-minified version of the library.
We can also point to any of the versions under the directory **min/**. For
now, let's use the non-minified.

I> Moment site also includes information when [consuming via bower](http://momentjs.com/docs/#/use-it/bower/.)

Let's add the following to our **Brocfile.js**:

~~~~~~~~
app.import('bower_components/moment/moment.js');
~~~~~~~~

Next, if we navigate to
[http://localhost:4200](http://localhost:4200), open the console, and
type "moment" we should have access to the **moment** object.

We have successfully included our first JavaScript plugin,
but we need to be aware of some gotchas.

## It's a global!

At the beginning of the book, we mentioned that one of the things
ember-cli gives you is support to work with **ES6 Modules** rather than
globals. It feels like taking a step backward if we add a library
and then use it through its global, right?

The sad news is that not all libraries are written in such a way that
they can be consumed easily via a modules loader. Even so, if there is
an **AMD** definition included in the library, not all of them are
compatible with the module loader used by **ember-cli**.


For example, **moment** includes an **AMD** version:

{title="moment AMD definition",lang="JavaScript"}
~~~~~~~~
// ...
} else if (typeof define === 'function' && define.amd) {
    define('moment', function (require, exports, module) {
        if (module.config && module.config() && module.config().noGlobal === true) {
            // release the global variable
            globalScope.moment = oldGlobalMoment;
        }

        return moment;
    });
~~~~~~~~

Unfortunately, the module loader ember-cli is using doesn't support
that yet.

Other libraries do the following:

{title="Anonymous module",lang="JavaScript"}
~~~~~~~~
define([], function() {
	return lib;
});
~~~~~~~~

This is known as an anonymous module. Although its syntax is
valid, the loader doesn't support this either because it expects named
modules.

I> In the near future people will be able to use **moment** or other
I> JavaScript libraries via **import**, but the integration is not yet ready
I> yet. See issue [#2177](https://github.com/stefanpenner/ember-cli/issues/2177) for
I> more info.


This issue is not entirely the fault of ember-cli, but in fact results from
everyone building their libraries in different formats, making it difficult
for consumers to use.

What can we do about it?

## Wrapping globals

Instead of consuming globals directly, let's wrap them in a helper
module that will allow us to foster the use of modules and to easily
update or replace **moment** once we have a way to load it via the
module loader.

First, let's create a utils file called **date-helpers**:

{title="", language="JavaScript"}
~~~~~~~~
$ ember g util date-helpers
installing
  create app/utils/date-helpers.js
installing
  create tests/unit/utils/date-helpers-test.js
~~~~~~~~

Replace **app/utils/date-helpers.js** with the following:

{title="Wrapping globals: app/utils/date-helpers.js", language="JavaScript"}
~~~~~~~~
function formatDate(date, format) {
  return window.moment(date).format(format);
}

export {
  formatDate
};
~~~~~~~~

Here we are wrapping the call to **moment#format** in the function
**formatDate**, which we can consume doing **import { formatDate } from
'utils/date-helpers';**. With this, we are back to our idea of using
modules. We'll also have the facility to easily update **moment**
when our loader is ready to load it.

If we decide to stop using **moment** and replace it with any
other similar library, we won't need to change our
consuming code since it doesn't care how **format-date** is
implemented.

## Writing an Ember helper: formatted-date.

Helpers are pieces of code that help us augment our templates. In this
case, we want to write a helper to create a date as a formatted string.

**ember-cli** includes a generator for helpers. Let's create
**formatted-date** with the command **ember g helper formatted-date**, and
then modify **app/helpers/formatted-date** so it consumes our format
function.

{title="Formatted Date helper", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

// We are consuming the function defined in our utils/date-helpers.
import { formatDate } from '../utils/date-helpers';

export default Ember.Handlebars.makeBoundHelper(function(date, format) {
  return formatDate(date, format);
});
~~~~~~~~

Once we have our helper defined, we can use it in **app/templates/articles/index.hbs**:

{title="Using formatted-date in app/templates/articles/index.hbs", lang="handlebars"}
~~~~~~~~
<table class="primary">
  <thead>
    <tr>
      <th>Description</th>
      <th>Notes</th>
      <th>Borrowed since</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    {{#each itemController='articles/item'}}
      <tr>
        <td>{{model.description}}</td>
        <td>{{model.notes}}</td>
        <td>{{formatted-date model.createdAt 'LL'}}</td>
        <td>{{view "select" content=states selection=model.state}}</td>
        <td>
          {{#if model.isSaving}}
            <p>Saving ...</p>
          {{/if}}
        </td>
      </tr>
    {{/each}}
  </tbody>
</table>
~~~~~~~~

Now, when we visit any of our friends' profiles, we should see the dates
in a more attractive format.

![Articles using formatted-date](images/articles-with-format.png)

## Working with libraries with named AMD distributions.

Before the addons system existed, the easiest way to distribute
JavaScript libraries to be consumed in ember-cli was to have a build
with a named AMD version, importing the library using **app.import**,
and whitelisting the library's exports.

Let's study
[ic-ajax](https://github.com/instructure/ic-ajax/tree/v2.0.1/lib), an
"Ember-friendly **jQuery.ajax** wrapper." If we navigate to the
[lib/main.js](https://github.com/instructure/ic-ajax/blob/master/lib/main.js),
we'll notice that the source of the application is written with
**ES6** syntax, but it is
[distributed](https://github.com/instructure/ic-ajax/tree/v2.0.1/dist)
in different formats. This allows us to consume it in either global or
module formats.

As mentioned previously, **loader.js** doesn't work with anonymous AMD
distributions. If we want to include **ic-ajax**, we need to use the
**named AMD** output. Let's try **ic-ajax** in our project for a first
sketch of the dashboard.

First we need to remove **ember-cli-ic-ajax** from our
**package.json** by running the following command:

{title="Uninstalling a npm package", lang="bash"}
~~~~~~~~
npm uninstall ember-cli-ic-ajax --save-dev
~~~~~~~~

The library we just removed wraps all the steps we are about to perform, but
we won't be using it. We are interested in learning how things
work under the hood and what we gain when we use the
addon.

Next we need to add the library to Bower. We can do so with `bower
install ic-ajax --save`. Once it's installed, let's import it into our
**Brocfile.js** as follows:

{title="Importing ic-ajax",lang="JavaScript"}
~~~~~~~~
app.import('bower_components/ic-ajax/dist/named-amd/main.js');
~~~~~~~~

**ic-ajax**'s default export is the **request** function, which allows us
to make petitions and manage them as if they were promises. Let's use
this to create a "dashboard" object.

We'll present dashboard as the home page of our application, so when
we navigate to the root url we'll see the reports. We already have the
template, but let's create the route to load the required data. Create
`app/routes/index.js` with the following content:

{title="app/routes/index.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';
import request from 'ic-ajax';

export default Ember.Route.extend({
  model: function()  {
    return request('/api/friends').then(function(data){
      return {
        friendsCount: data.friends.length
      };
    });
  }
});
~~~~~~~~

And then replace `app/templates/index.hbs` so it uses
**friendsCount**:

{title="", lang="handlebars"}
~~~~~~~~
<h1>Dashboard</h1>
<hr/>
<h2>Total Friends: {{model.friendsCount}}</h2>
~~~~~~~~

The previous code is correct, but we'll see the following error when
running **ember server**:

{title="Error when importing ic-ajax", lang="bash"}
~~~~~~~~
$ ember server --proxy http://api.ember-cli-101.com
version: 0.1.5
Proxying to http://api.ember-cli-101.com
Livereload server on port 35729
Serving on http://0.0.0.0:4200
ENOENT, no such file or directory '/borrowers/tmp/tree_merger-tmp_dest_dir-KIfHrFRc.tmp/ic-ajax.js'
Error: ENOENT, no such file or directory '/borrowers/tmp/tree_merger-tmp_dest_dir-KIfHrFRc.tmp/ic-ajax.js'
...
~~~~~~~~

At the beginning of this chapter, we mentioned that part of the process
of consuming named AMD libraries is to use **app.import** and
**whitelist** the library's exports. We didn't explain what we meant
by the latter.

During the build process, all our files under **app/** go through a
transformation step where the ES6 modules are converted to AMD format.
When something like **import request from
'ic-ajax';** is found internally, the tool in charge of transpiling the code
checks if that is something already registered in the module system.
If not, it tries to find the module and convert it to the proper
format. In the previous scenario, it will try to find a file called
**ic-ajax.js**, but because it is a library we are including externally,
such a file doesn't exist. This causes the build to fail.

Whitelisting in this context means telling the tool in charge of
transforming our ES6 files to AMD that whenever **import request from
'ic-ajax'** is found, it is to assume its inclusion and refrain from
resolving it.

To do so, we pass an option called exports to **app.import** that
whitelists **ic-ajax** and its **exports**.

In the **Brocfile.js**, let's replace the call to **import** with the
following:

{title="Importing ic-ajax with exports",lang="JavaScript"}
~~~~~~~~
app.import('bower_components/ic-ajax/dist/named-amd/main.js', {
        exports: {
          'ic-ajax': [
            'default',
            'defineFixture',
            'lookupFixture',
            'raw',
            'request',
          ]
        }
      });
~~~~~~~~

If we run **ember server**, we'll see that everything works. We can
see the friends count in our dashboard by visiting
[http://localhost:4200/](http://localhost:4200/).

### ember-cli-ic-ajax

We started this chapter by removing **ember-cli-ic-ajax**, an
addon that wraps the call to import and include exports for us. If
we inspect the
[index file in the addon](https://github.com/rwjblue/ember-cli-ic-ajax/blob/master/index.js#L18),
we'll notice that it has almost the same things we added to our
**Brocfile.js**.

Now that we understand how importing named AMD libraries works, we can
remove the **import** for **ic-ajax** from the **Brocfile.js** and use
it via the addon. Let's run the following commands and then stop and
start the server. Everything should work:

{title="", lang="bash"}
~~~~~~~~
$ bower uninstall ic-ajax --save
$ npm i ember-cli-ic-ajax --save
~~~~~~~~

T> **npm i** is an alias fro **npm install**


### A temporary replacement for moment.js

Let's consume a simple named AMD library that takes a date and returns
its value after calling **.toDateString()**. This will be a simple
example just to practice another module for importing named AMD.

The name of the library is **borrowers-dates** and it is located in [https://github.com/abuiles/borrowers-dates](https://github.com/abuiles/borrowers-dates).

The following is the content of the library:

{title="borrowers-dates library", lang="JavaScript"}
~~~~~~~~
define("borrowers-dates", ["exports"], function(__exports__) {
  "use strict";
  function format(date) {
    return date.toDateString();
  }

  __exports__.format = format;
});
~~~~~~~~

The library exports a function called **format**. Let's consume it via
bower:

{title="", lang="bash"}
~~~~~~~~
bower install borrowers-dates --save
~~~~~~~~

And then import it through our **Brocfile.js**:


{title="Consuming borrowers-dates", lang="JavaScript"}
~~~~~~~~
app.import('bower_components/borrowers-dates/index.js', {
  exports: {
    'borrowers-dates': [
      'format'
    ]
  }
});
~~~~~~~~

With the library included, let's consume it in
**app/utils/date-helpers.js** instead of moment:

{title="Using borrowers-dates in app/utils/date-helpers.js", lang="JavaScript"}
~~~~~~~~
import { format } from 'borrowers-dates';


function formatDate(date) {
  return format(date);
}

export {
  formatDate
};
~~~~~~~~

Now when we visit the profile for any our friends with articles, we'll
see the dates rendered differently. This is because we are no longer using
moment.

X> ## Tasks
X>
X> Remove borrowers-dates and go back to using moment.

## ember-browserify

[Browserify](http://browserify.org/) is a Node library which allows us
to consume other Node libraries in the Browser using CommonJS (which
is Node's module system), what this means is that we can install
libraries like MomentJS using npm and then consume them in the browser
via browserify. But wait, to use Browserify we actually need to
install the library and create a "bundle" with our
dependencies, normally we'll run something like `browserify main.js -o
bundle.js` and then use `bundle.js` via a script tag `<script
src="bundle.js"></script>`.

As we can imagine this can get tricky and hard to manage in our
ember-cli application, but thanks to
[Edward Faulkner](https://github.com/ef4) there is addon which allow
us to consume libraries from npm with browserify without needing us to
worry about the bundling process, it is called [ember-browserify](https://github.com/ef4/ember-browserify).

### Using ember-browserify

First we need to install the addon, which we can do running `ember
install`:

{title="", lang="bash"}
~~~~~~~~
$ ember install:addon ember-browserify
~~~~~~~~

Once the addon has been installed, we are going to use it to consume
MomentJS from npm in our `date-helpers` file.

Before doing that, let's make sure we have removed `moment` from our
`bower.json` and also that we have removed
`app.import('bower_components/moment/moment.js');` from the
`Brocfile`.

Next, let's install moment via npm, which we can do with `npm install
moment --save-dev`.

Once it has been installed we can consume it from npm thanks to
ember-browserify just doing `import moment from 'npm:moment';`.

Let's use it in our `date-helpers` so `formatDate` uses `moment`.

{title="app/utils/date-helpers.js", lang="JavaScript"}
~~~~~~~~
import moment from 'npm:moment';

function formatDate(date, format) {
  return moment(date).format(format);
}

export {
  formatDate
};
~~~~~~~~

And that's it, we are now consuming MomentJS via browserify just as if
it was other module in our application.

## Wrapping up

In this chapter we have covered how to work with JavaScript plugins
both as globals, consuming named AMD plugins and via ember-browserify.

We didn't cover how to write reusable plugins to be consumed with
ember-cli. This is what addons are used for, and we'll talk
about them in the next chapter.

I> The API for consuming third-party plugins is not 100% finished in
I> ember-cli, and this chapter might change along with its development. The story
I> is still a work in progress, but the goal is to make it easier to work
I> with any plugin regardless of the format in which it was written.
