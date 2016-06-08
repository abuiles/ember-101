# Working with JavaScript plugins

In this chapter we'll learn how to write Ember helpers that can be
consumed in our templates. To do so, we'll write a helper called
**format-date** that will show the date when an
article was borrowed. Instead of showing **Sun Sep 28 2014 04:58:30
GMT-0500**, we'll see **September 28, 2014**.

We'll implement **format-date** using
[Momentjs](http://momentjs.com), a library that facilitates working
with dates in JavaScript.

## Installing moment

Remember that ember-cli uses Bower to manage frontend dependencies.
Here we'll use the same pattern used to install **picnicss**: we'll
add **moment** to Bower and then use **app.import** in our
**ember-cli-build.js**.

I> We can also install front-end dependencies via npm if they are
I> packed as addons. We'll learn more about this in a later chapter.


First, we install **moment**:

~~~~~~~~
$ bower install moment --save
~~~~~~~~

The option `--save` adds the dependency to our **bower.json**. We
should find something similar to **"moment": "~2.13.0"** (the version
might be different).

Next, let's import **moment**. To find out which file to import, let's go
to **bower_components/moment/**. We'll see that it contains a
**moment.js** file that is the non-minified version of the library.
We can also point to any of the versions under the directory **min/**. For
now, let's use the non-minified.

I> Moment site also includes information when [consuming via bower](http://momentjs.com/docs/#/use-it/bower/.)

Let's add the following to our **ember-cli-build.js**:

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

This issue is not entirely the fault of ember-cli, but in fact results
from everyone building their libraries in different formats, making it
difficult for consumers to use.

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

## Writing an Ember helper: format-date.

Helpers are pieces of code that help us augment our templates. In this
case, we want to write a helper to create a date as a formatted string.

**ember-cli** includes a generator for helpers. Let's create
**format-date** with the command **ember g helper format-date**, and
then modify **app/helpers/format-date.js** so it consumes our format
function.

{title="Format date helper app/helpers/format-date.js ", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

// We are consuming the function defined in our utils/date-helpers.
import { formatDate  } from '../utils/date-helpers';

export default Ember.Helper.helper(function([date, format]) {
  return formatDate(date, format);
});
~~~~~~~~

Once we have our helper defined, we can use it in the component
**app/templates/components/loans/loan-row.hbs**:

{title="Using format-date in app/components/loans/loan-row.hbs", lang="handlebars"}
~~~~~~~~
<td>{{loan.article.name}}</td>
<td>{{loan.notes}}</td>
<td>{{format-date loan.createdAt "LL"}}</td>
<td>
  {{input type="checkbox" checked=loan.returned click=(action save loan)}}
</td>
<td>
  {{#if loan.isSaving}}
    <p>Saving...</p>
  {{/if}}
</td>
~~~~~~~~

Now, when we visit any of our friends' profiles, we should see the dates
in a more attractive format.

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

Let's add the library to bower with `bower
install ic-ajax --save`. If we are asked to select a version of ember,
then we need to select the one which is higher. Once it's installed,
let's import it into our **ember-cli-build.js** as follows:

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
  model()  {
    return request('/friends').then(function(data){
      return {
        friendsCount: data.data.length
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

Next, if we run **ember server**, we'll see that everything works. We
can see the friends count in our dashboard by visiting
[http://localhost:4200/](http://localhost:4200/).

### ember-ajax

We installed `ic-ajax` only for demonstration purposes but this library
is not longer the recommended way to wrap ajax request in our ember
applications. Instead, there is now a library called
[ember-ajax](https://github.com/ember-cli/ember-ajax) which is the one
officially supported by the community.

The library `ember-ajax` is consumed a bit differently than `ic-ajax`
since it requires us to use a service.

Now that we understand how importing named AMD libraries works, ew can
remove the **import** for **ic-ajax** from the **ember-cli-build.js** and use `ember-ajax`. Let's run the following commands and then stop and
start the server.

{title="", lang="bash"}
~~~~~~~~
$ bower uninstall ic-ajax --save
$ ember install ember-ajax
~~~~~~~~

If we navigate to the dashboard, we'll see an error like `Uncaught
Error: Could not find module `ic-ajax` imported from
`borrowers/routes/index`. We need to change the route to consume
`ember-ajax` instead, it should look like the following:

{line-numbers=off, title="app/routes/index.js", lang="javascript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  //
  // Check the following for more information about services
  // https://guides.emberjs.com/v2.5.0/applications/services/
  //
  ajax: Ember.inject.service(),
  model()  {
    return this.get('ajax').request('/friends').then(function(data){
      return {
        friendsCount: data.data.length
      };
    });
  }
});
~~~~~~~~

After changing the route with the code above, our application should
work fine again.


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

And then import it through our **ember-cli-build.js**:

{title="Consuming borrowers-dates", lang="JavaScript"}
~~~~~~~~
app.import('bower_components/borrowers-dates/index.js');
~~~~~~~~

With the library included, let's consume it in
**app/utils/date-helpers.js** instead of moment:

{title="Using borrowers-dates in app/utils/date-helpers.js", lang="JavaScript"}
~~~~~~~~
import { format as borrowersDate } from 'borrowers-dates';


function formatDate(date, format) {
  return borrowersDate(date, format);
}

export {
  formatDate
};
~~~~~~~~

Now when we visit the profile for any our friends with articles, we'll
see the dates rendered differently. This is because we are no longer
using moment.

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
$ ember install ember-browserify
~~~~~~~~

Once the addon has been installed, we are going to use it to consume
MomentJS from npm in our `date-helpers` file.

Before doing that, let's remove moment with `bower uninstall moment
--save` and also remove
`app.import('bower_components/moment/moment.js');` from
`ember-cli-build.js`.

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
I> ember-cli, and this chapter might change along with its
I> development. The story is still a work in progress, but the goal is
I> to make it easier to work with any plugin regardless of the format in
I> which it was written.
