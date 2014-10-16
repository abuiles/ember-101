# Components and Addons

## Web Components

Web Components are a new mechanism which allows us to extend the DOM
with our own elements, instead of limiting ourselves to traditional
tags we can define our own ones wrapping up all the display logic in a
single bundle and then reuse it between different applications, we'll
use the component as any other tag and the browser will
understand how to render it based on its definition.

Let's examine how a share on twitter button works, currently we need
to include some JavaScript  and then create an anchor tag which
will be transformed by the JavaScript snippet:

{title="Twitter Share Button", lang="html"}
~~~~~~~~
<a class="twitter-share-button"
  href="https://twitter.com/share">
Tweet
</a>
<script type="text/javascript">
window.twttr=(function(d,s,id){var t,js,fjs=d.getElementsByTagName(s)[0];if(d.getElementById(id)){return}js=d.createElement(s);js.id=id;js.src="https://platform.twitter.com/widgets.js";fjs.parentNode.insertBefore(js,fjs);return window.twttr||(t={_e:[],ready:function(f){t._e.push(f)}})}(document,"script","twitter-wjs"));
</script>
~~~~~~~~


The previous code is going to be the same independently of the
application we are developing, sounds like a good candidate for a
component since it's a chunk of code which can be reused across
applications, a possible twitter-button component could look something
like the following:

{title="Twitter Share Button", lang="html"}
~~~~~~~~
<twitter-button>
  Ember.js Rocks!
</twitter-button>
~~~~~~~~

And then have all the implementation details hidden in its definition,
as consumers we wouldn't need to worry about how it is done, since we
are just interested in the final output.

Web Components are a great tool which we can use to write more
expressive applications and avoid code repetition within projects but
unfortunately it is not supported yet in all the browser.

To tackle this problem Ember introduced the concept of Components,
it's basically an API which allows us write components today and
following a specification as closely as possible to the one specified
by the **W3C**, the day components become widely available we'll be
able to switch without problems.

I> The official name for Web Components in the **W3C** is
I> (Custom Elements)[http://w3c.github.io/webcomponents/spec/custom/#about]

## ember-cli addons

**ember-cli** has a built-in mechanism which allows us augment **ember-cli**
functionality or share code easily between different applications,
such mechanism is known as addons.

Using addons we can easily write Ember Components and share them with
others using npm, let's create our first component which will help us
grab an image to use as placeholder in our friends profiles.

## ember-cli-fill-murray

http://www.fillmurray.com is a service we can use to get random images
of Bill Murray to use as placeholders, let's write an addon so when
include it we can do something like the following in any of our
templates:

{title="Fill Murray Component", lang="handlebars"}
~~~~~~~~
{{fill-murray width=300 length=300}}
~~~~~~~~

First we need to create the addon, **ember-cli** has a command to help us
with this, outside of our borrowers directory let's run the following
command:

{title="Creating an ember-cli addon", lang="bash"}
~~~~~~~~
$ ember addon ember-cli-fill-murray
version: 0.1.1
installing..
  create .bowerrc
  create .editorconfig
  create .ember-cli
  create tests/dummy/
  ...
~~~~~~~~

The **addon** command creates a directory very similar to the one
created by **new** but the former is done in a way that it can be
distributed as an addon.

If we go to the directory **ember-cli-fill-murray**, it will look like
 the following:

{title="Addon directory", lang="bash"}
~~~~~~~~
.
|--  Brocfile.js
|--  README.md
|--  addon
|--  app
|--  bower.json
|--  bower_components
|--  config
|--  index.js
|--  node_modules
|--  package.json
|--  testem.json
|--  tests
└--  vendor
~~~~~~~~

If we open **package.json** we'll see the following section:

{title="", lang="json"}
~~~~~~~~
  "keywords": [
    "ember-addon"
  ],
~~~~~~~~

That's how **ember-cli** detects the presence of an **addon**, when we
include the library in an **ember-cli** project, it will transverse
the dependencies and then identify as **addons** the ones with the
keyword **ember-addon**, we'll also use **package.json** to specify
any dependency our library might have.

Next we have **index.js** which is the entry point for loading our
**addon**, if we need to add any extra configuration for our addon
we'll specify it in this file, for now let's work with the basic one
which looks as the following:

{title="", lang="JavaScript"}
~~~~~~~~
module.exports = {
  name: 'ember-cli-fill-murray'
};
~~~~~~~~

Next we have to the directories **app** and **addon** this is where
the code for our **addons** will live.

Whatever we put into **app** will be merged into our the application
namespace, meaning we consume it just as if it were inside our
**ember-cli**'s project.

For example if we had an **app/model/friend-base.js** file in our
**addon**, we could consume in any of the models in our our
**borrowers** app like:

{title="Consuming an addon's app/model/friend-base.js", lang="JavaScript"}
~~~~~~~~
import FriendBase from './friend-base';

...
~~~~~~~~


If we had put **friend-base.js** into the **addon** directory, instead
of getting merge into the consuming application namespace it would be
kept under the **addon namespace**, with the previous example if our
**addon** was called **borrowers-base** and we had
**addon/models/friend-base.js**, then we would have consumed it like
the following:

{title="Consuming modules from addon's namespace", lang="JavaScript"}
~~~~~~~~
import FriendBase from 'borrowers-base/models/friend-base';

...
~~~~~~~~

Going back to our **ember-cli-murray addon**, we have the directory in
place and we want to distribute a component called **fill-murray**,
inside the directory we can also use **ember-cli** generators, so
we'll do that to create the component:


{title="Bill Murray Component", lang="bash"}
~~~~~~~~
$ ember generate component fill-murray
version: 0.1.1
installing
  create app/components/fill-murray.js
  create app/templates/components/fill-murray.hbs
installing
  create tests/unit/components/fill-murray-test.js
~~~~~~~~

In **app/components/fill-murray.js** we can specify the properties for
our component:

{title="app/components/fill-murray.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Component.extend({
  length: 100, // Default length and width would 100
  width: 100,

  //
  // The following computed property will give us the url for
  // fill-murray, in this case it depends on the properties length and width.
  //

  src: Ember.computed('length', 'width', function() {
    var base = 'http://www.fillmurray.com/';
    return base + this.get('width') +  '/' + this.get('length');
  })
});
~~~~~~~~

Next we need to specify the body of our component in
**app/templates/components/fill-murray.hbs**:

~~~~~~~~
<img {{bind-attr src=src}}>
~~~~~~~~

When rendering the component it will insert an **img** tag reading the
source from the computed property we specified.

With that our **addon** is ready, the next step is to distribute it
via npm, to do that first let's change the name in **package.json**
because **ember-cli-fill-murray** is already taken, in this case
we'll use **ember-cli-fill-murray-your-github-nickname** and also set
version to **0.1.0**, it would look something as the following:

~~~~~~~~
  "name": "ember-cli-fill-murray-your-github-nickname",
  "version": "0.1.0",
~~~~~~~~

With the previous values in place we can just do **npm publish** and
our **addon** will be ready to be consumed.

## Consuming fill-murray in borrowers

Once our package is in **npm**, we can add it to our application
running the following command:

{title="", lang="bash"}
~~~~~~~~
$ npm install --save ember-cli-fill-murray-your-github-nickname
~~~~~~~~

Once it has been installed we can consume the component in any of our
templates as follows:

{title="", lang="handlebars""}
~~~~~~~~
{{fill-murray width=150 length=150}}
~~~~~~~~

Let's use it in our friend template, first we want to add some styling
for our friends template in **app/styles/app.css**:

{title="", lang="css"}
~~~~~~~~
.friend-profile{
  text-align: left;
}

.friend-profile img {
  float:left;
  margin:1em 2em 1em 1em;
}

.friend-profile .checkbox {
  clear:both;
  display:block;
  margin:1em;
}

.friend-profile-links li {
  display: inline;
  list-style-type: none;
  padding-right: 20px;
}
~~~~~~~~

And then modify **app/templates/friends/show.hbs** to look like the
following:

{title="Consuming fill-murray in app/templates/friends/show.hbs", lang="handlebars"}
~~~~~~~~
<div class="friend-profile" class="row">
  <div class="friend-info full">
    {{fill-murray width=300 length=300}}
    <div>
      <p>{{fullName}}</p>
      <p>{{email}}</p>
      <p>{{twitter}}</p>
      <ul class="friend-profile-links">
        <li>{{link-to 'Edit info' 'friends.edit' this}}</li>
        <li>{{link-to 'Lend article' 'articles.new'}}</li>
        <li><a href="#" {{action "delete" this}}>Delete</a></li>
      </ul>
    </div>
  </div>
</div>

<div class="articles-container">
  {{outlet}}
</div>
~~~~~~~~

After editing the template, if we visit a friend's profile we'll see a
Bill Murray placeholder.
