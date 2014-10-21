# Working with JavaScript plugins

In this chapter we'll learn how to write Ember helpers that can be
consumed in our templates, to do so we'll write a helper called
**formatted-date** which we are going to use to show the date when an
article was borrowed, so instead of showing **Sun Sep 28 2014 04:58:30
GMT-0500**, we'll see **September 28, 2014**.

We'll implement **formatted-date** using
[Momentjs](http://momentjs.com), a library which facilitates working
with dates in JavaScript.

## Installing moment

Remember that ember-cli uses Bower to manage frontend dependencies,
here we'll be using the same pattern used to install **picnicss**:
we'll add **moment** to bower and then use **app.import** in our
**Brocfile.js**.

I> We can also install front-end dependencies via npm if they are
I> packed as addons, we'll learn more about on the addons chapter.


First, we install **moment**:

~~~~~~~~
$ bower install moment --save
~~~~~~~~

The option `--save` adds the dependency to our **bower.json**, we
should find something similar to **"moment": "~2.8.3"** (the version
might be different).

Next, let's import **moment**, to know which file to import we can go
to **bower_components/moment/** and we'll see that it has a
**moment.js** file which is the non-minified version of the library or
we can point to any of the version under the directory **min/**, for
now let's use the non-minified.

I> Moment site also includes information when [consuming via bower](http://momentjs.com/docs/#/use-it/bower/.)

Let's add the following to our **Brocfile.js**:

~~~~~~~~
app.import('bower_components/moment/moment.js');
~~~~~~~~

Next, if we navigate to
[http://localhost:4200](http://localhost:4200), open the console and
type "moment", then we should have access to the **moment** object.

With that we have successfully included our first JavaScript plugin,
but there are some gotchas we need to be aware of.

## It's a global!

At the beginning of the book we mention that one of the things
ember-cli gives you is support to work with **ES6 Modules** instead of
globals, so it feels like giving a step backwards if we add a library
and then use it through its global, right?.

The sad news is that not all libraries are written in such way that
they can be consumed easily via a modules loader, and even so if there is
an **AMD** definition included in the library not all of them are
compatible with the modules loader used by **ember-cli**.


For example, **moment** includes an **AMD** version

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

But unfortunately the modules loader which ember-cli is using, doesn't
support that yet.

Other libraries do the following:

{title="Anonymous module",lang="JavaScript"}
~~~~~~~~
define([], function() {
	return lib;
});
~~~~~~~~

That is known as an anonymous module and even though its syntax is
valid, the loader doesn't support that either since it expects named
modules.

I> In a near future people will be able to use **moment** or other
I> JavaScript libraries via **import** but the integration is not ready
I> yet. See issue [#2177](https://github.com/stefanpenner/ember-cli/issues/2177) for
I> more info.


This whole problem is not really ember-cli's fault but the fact
everyone is building their libraries in different formats making hard
for consumers to use.

But what can we do about it?

## Wrapping globals

Instead of consuming globals directly let's wrap then in a helper
module which will help us foster the use of modules and also update or
replace easily **moment** once we have a way to load it via the module
loader.

First, let's create an utils file called **date-helpers**:

{title="", language="JavaScript"}
~~~~~~~~
$ ember g util date-helpers
installing
  create app/utils/date-helpers.js
installing
  create tests/unit/utils/date-helpers-test.js
~~~~~~~~

Replace **app/utils/date-helpers.js** with the following:

{title="Wrapping globals", language="JavaScript"}
~~~~~~~~
function formatDate(date, format) {
  return window.moment(date).format(format);
}

export default {
  formatDate
}
~~~~~~~~


Here we are wrapping the call to **moment#format** in the function
**formatDate** which we can consume doing **import { formatDate } from
'utils/date-helpers';**, with that we are back to our idea of using
modules, and also we'll have the facility to update easily **moment**
when our loader is ready to load it.

Also if we decide to stop using **moment** and replace it with any
other library which does the same, we don't need to change our
consuming code since it doesn't care how **format-date** is being
implemented.

## Writing an Ember helper: formatted-date.

Helpers are pieces of code that help us augment our templates, in this
case we want to write a helper to have a a date as a formatted string.

**ember-cli** includes a generator for helpers too, so let's create
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

{title="Using formatted-date", lang="handlebars"}
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
        <td>{{description}}</td>
        <td>{{notes}}</td>
        <td>{{formatted-date createdAt 'LL'}}</td>
        <td>{{view Ember.Select content=states selection=state}}</td>
        <td>
          {{#if isSaving}}
            <p>Saving ...</p>
          {{/if}}
        </td>
      </tr>
    {{/each}}
  </tbody>
</table>
~~~~~~~~

Now, when we visit any our friends profile, we should see the dates
with a nicer format.

![Articles using formatted-date](images/articles-with-format.png)

## Working with libraries with named AMD distributions.

Before the addons system exist the easiest way to distribute
JavaScript libraries to be consume in ember-cli was to have a built
with a named-AMD version, importing the library using **app.import**
and whitelisting the library's exports.

Let's study
[ic-ajax](https://github.com/instructure/ic-ajax/tree/v2.0.1/lib) an
"Ember-friendly **jQuery.ajax** wrapper", if we navigate to the
[lib/main.js](https://github.com/instructure/ic-ajax/blob/master/lib/main.js)
we'll notice that the source of the application is written with
**ES6** syntax but it is
[distributed](https://github.com/instructure/ic-ajax/tree/v2.0.1/dist)
in different formats so you can consume it like a global or in a
module format.

As mentioned previously **loader.js** doesn't work with anonymous AMD
distributions so if we want to include **ic-ajax** we need to use the
**named-amd** output, let's try **ic-ajax** in our project for a first
sketch of the dashboard.

First we need to remove **ember-cli-ic-ajax** from our
**package.json** running the following command:

{title="Uninstalling a npm package", lang="bash"}
~~~~~~~~
npm uninstall ember-cli-ic-ajax --save-dev
~~~~~~~~

The library we just removed wraps all the steps we are about to do but
we won't be using it since we are interested in learning how things
are working under the hood and what are we gaining when using the
addon.

Next we need to add the library to bower, we can do so with **bower
install ic-ajax --save**, once installed let's import it in our
**Brocfile.js** as follows:

{title="Importing ic-ajax",lang="JavaScript"}
~~~~~~~~
app.import('bower_components/ic-ajax/dist/named-amd/main.js');
~~~~~~~~

**ic-ajax** default export is the **request** function which allows us
to make petitions and manage them as if they were promises, let's use
that to create a "dashboard" object.

We'll present dashboard as the home page of our application, so when
we navigate to the root url we'll see the reports, we already have the
template but let's create the route to load the required data, create
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
<h2>Total Friends: {{friendsCount}}</h2>
~~~~~~~~

The previous code is correct but we'll see the following error when
running **ember server**:

{title="Error when importing ic-ajax", lang="bash"}
~~~~~~~~
$ ember server --proxy http://api.ember-cli-101.com
version: 0.1.2
Proxying to http://api.ember-cli-101.com
Livereload server on port 35729
Serving on http://0.0.0.0:4200
ENOENT, no such file or directory '/borrowers/tmp/tree_merger-tmp_dest_dir-KIfHrFRc.tmp/ic-ajax.js'
Error: ENOENT, no such file or directory '/borrowers/tmp/tree_merger-tmp_dest_dir-KIfHrFRc.tmp/ic-ajax.js'
...
~~~~~~~~

At the beginning of this chapter we mentioned that part of the process
on consuming named AMD libraries was to use **app.import** and
**whitelist** the library's exports without explaining what we meant
with the later.

During the build process all our files under **app/** go through a
transformation step where the ES6 modules are converted to AMD format,
when something like the following is found **import request from
'ic-ajax';** internally the tool in charge of transpiling the code,
checks if that is something already registered in the module system
and if not it tries to find the module and convert it to the proper
format, in the previous scenario it will try to find a file called
**ic-ajax.js**, but since it is a library we are including externally
such file doesn't exist hence causing the build to fail.

Whitesting in this context means telling the tool in charge of
transforming our ES6 files to AMD that whenever **import request from
'ic-ajax'** is found, then assume is already included so it doesn't
try to resolve it.

To do so we pass an option called exports to **app.import** which
whitelist **ic-ajax** and and its **exports**.

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

If we run **ember server** we'll see that everything works and we'll
see the friends count in our dashboard visiting
[http://localhost:4200/](http://localhost:4200/)

### ember-cli-ic-ajax

We started the chapter by removing **ember-cli-ic-ajax** which is an
addon wrapping the call to import and include the exports for us, if
we inspect the
[index file in the addon](https://github.com/rwjblue/ember-cli-ic-ajax/blob/master/index.js#L18),
we'll notice that it has almost the same things we added to our
**Brocfile.js**.

Now that we understand how importing named-amd libraries work, we can
remove the **import** for **ic-ajax** from the **Brocfile.js** and use
it via the addon, let's run the following commands and then stop and
start the server, everything should work:

{title="", lang="bash"}
~~~~~~~~
$ bower uninstall ic-ajax --save
$ npm i ember-cli-ic-ajax --save
~~~~~~~~

T> **npm i** is an alias fro **npm install**


### A temporary replacement for moment.js

Let's consume a simple named-amd library which takes a date and return
its value after calling **.toDateString()**, this will be a simple
example just to practice one more importing named-amd modules.

The name of the library is **borrowers-dates** and is located in [https://github.com/abuiles/borrowers-dates](https://github.com/abuiles/borrowers-dates)

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

The library export a function called **format**, let's consume it via
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
**app/utils/date-helpers** instead of moment:

{title="Using borrowers-dates", lang="JavaScript"}
~~~~~~~~
import { format } from 'borrowers-dates';


function formatDate(date) {
  return format(date);
}

export {
  formatDate
};
~~~~~~~~

Now if we visit the profile for any our friends with articles, we'll
see the dates rendering differently since we are not longer using
moment.

X> ## Tasks
X>
X> Remove borrowers-dates and go back to using  moment.


## Wrapping up

In this chapter we have learned how to work with JavaScript plugins
both as globals and consuming named-amd plugins.

We didn't learn how to write reusable plugins to be consumed with
ember-cli, since that's what addons are used for and we'll be talking
about them in the next chapter.

I> The API for consuming third-party plugins is not 100% finished in
I> ember-cli and this chapter might change to adjust to it. The story
I> is still a work in progress but the idea is to make easier to work
I> with any plugin independently of the format in which is written.
