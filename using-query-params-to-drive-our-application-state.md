# Driving our application state through the URL

In JSConf EU 2013, [Tom Dale](https://twitter.com/tomdale) gave a talk
called:
[Stop Breaking the Web](http://2013.jsconf.eu/speakers/tom-dale-stop-breaking-the-web.html).

Tom talks about the importance of the URL and how we should give it a
higher priority in our applications, ideally the URL should be able to
reflect our application state in such way that we can reference it
easily, bookmark it or share it with others.

Some of us have probably experienced a lot of frustration when
visiting a website which has some kind of search functionality but
loses our selections between page reloads or doesn't allows to share
easily what we are seeing with others.

A common example of such websites are airlines, the following image is
a search on Delta's website where we searched for flights for the next
EmberConf.

![Search on Delta](images/delta.png)

The URL after the search is
http://www.delta.com/air-shopping/findFlights.action which doesn't
really tell us anything about the screen we are visiting, if we copy
and paste the URL in other browser, we'll get a bunch of
errors and not the search we originally did.

Now let's do a search on [hipmunk](https://www.hipmunk.com), they give
a great deal of importance to URLs

![hipmunk search](images/hipmunk.png)

The URL for the search above has the following
[flights/MDE-to-PDX#!dates=Aug23,Aug31&pax=1](https://www.hipmunk.com/flights/MDE-to-PDX#!dates=Aug23,Aug31&pax=1),
isn't that beautiful? Just reading the URL we have an idea where are
we trying to travel and the dates, also just clicking it will take us
to the search we are seeing in the image, suppose we want someone to
buy the ticket for us, we can just share the URL and be done with it.

Ember also gives a lot of importance to the URL, in fact our
applications are driven by URLs which we specify in **app/router.js**,
it doesn't mean we are free of building bad applications which don't
respect the URL, but at least it gives us the tools to avoid this
issues and invite us to think better about our URLs.

## Sorting friends.

When visiting the friends index, we want to be able to sort them
clicking on the **Name** or **Articles** column, and then toggle
ascending or descending between clicks.


Ember includes a mixin called
[SortableMixin](http://emberjs.com/api/classes/Ember.SortableMixin.html)
which extends arrays to have sorting functionality, by default
[Ember.ArrayController](http://emberjs.com/api/classes/Ember.ArrayController.html)
which is the class of our `Friends Index Controller` extends this class
so we can make it sort its content easily.

To use the mixing we only need specify the property `sortProperties`
and optionally we can specify `sortAscending`.

Let's try it creating the `Friends Index Controller`:

{title="app/controllers/friends/index.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.ArrayController.extend({
  sortAscending: true,
  sortProperties: ['fullName']
});
~~~~~~~~

If we visit http://localhost:4200/friends, we'll see our friends
sorted by `fullName` in ascending order, if we change `sortAscending`
to `false` and refresh, we'll see them sorted in descending order.

### Changing the sort field dynamically

To change our sort field dynamically we are going to create an
action `setSortBy` which will receive as parameter the field we want to sort our
properties by and make `sortProperties` a computed property which
depends on `sortBy`.

We'll also toggle the property `sortAscending` every time we call the
action `setSortBy` such as if it's `true` then it becomes `false` and
the other way around.

{title="app/controllers/friends/index.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.ArrayController.extend({
  sortAscending: false,

  //
  // We'll use sortBy to hold the name of the field we want to sort by.
  //
  sortBy: 'fullName',

  // We are making sortProperties a computed property
  // If we change the value for sortBy then the property will be
  // recalculated.

  sortProperties: Ember.computed('sortBy', function() {
    return [this.get('sortBy')];
  }),
  actions: {
    //
    // The setSortBy function receives the name of the function and
    // toggle `sortAscending`. The function `toggleProperty`  comes from the
    // [Observable Mixin](http://emberjs.com/api/classes/Ember.Observable.html)
    // it switches a boolean property between false and true.
    //
    setSortBy: function(fieldName) {
      this.set('sortBy', fieldName);
      this.toggleProperty('sortAscending');

      return false;
    }
  }
});

~~~~~~~~

Now we need to call the `setSortBy` action in the
**app/templates/friends/index.hbs**

{title="app/templates/friends/index.hsb", lang="handlebars"}
~~~~~~~~
<table class="primary">
  <thead>
    <tr>
      <th {{action "setSortBy" "fullName"}}> Name</th>
      <th {{action "setSortBy" "totalArticles"}}>Articles</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    {{#each}}
      <tr>
        <td>{{link-to fullName "articles" this}}</td>
        <td>{{totalArticles}}</td>
        <td><a href="#" {{action "delete" this}}>delete</a></td>
      </tr>
    {{/each}}
  </tbody>
</table>
~~~~~~~~

If we go to http://localhost:4200/friends and click on **Name** or
**Articles** then the list of friends will get sorted.

We got sorting working! But we just built one of those sites which
doesn't allow us to reproduce our app state from the URL, suppose we
want someone to have a look at our friends list sorted by the total
number of articles they have, we can't do it right now.

Ember has an useful feature called
[Query Parameters](http://emberjs.com/guides/routing/query-params/) which
allow us to persist application state in the URL as parameters like
`/friends?sortBy=name&sortAscending=true`.


To use them we only need to specify a property called `queryParams` in
the controller associated with the route we are visiting and list
every property which should be persisted as query parameter.

In our scenario we'll modify the controller as follows:

{title="app/controllers/friends/index.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.ArrayController.extend({
  queryParams: ['sortBy', 'sortAscending'],
  sortAscending: true,
  sortBy: 'fullName',
  sortProperties: Ember.computed('sortBy', function() {
    return [this.get('sortBy')];
  }),
  actions: {
    // omitted
  }
~~~~~~~~

If we visit http://localhost:4200/friends the URL won't have any
query parameters but as soon as we click any of the headers then the query
parameters will change, the query parameters are only included when
the default value for the property change, in our case that would be
when `sortAscending` changes to something different to `true` and
`sortBy` to something different to `fullName`.

Now we can refresh the browser or copy the URL into a new tab and
we'll see the same! We are driving our application state through the
URL.

We can also use query params with the `link-to` helper, so if we want
a link to the  friends index sorted by `totalArticles` we can write it
like: `{{#link-to 'friends' (query-params
sortBy="totalArticles")}}Friends{{/link-to}}`

## Refreshing the model when query parameters changes

By default the model hook won't be called if any of the query parameters
change, but there are scenarios where this can be the desired
behavior, for example when we are using pagination and we don't have
all the users in the store, under that scenario we'll ask the API for the
users sorted by a given field in ascending or descending order.

Supposing our API supports the parameters `sortBy` and
`sortAscending`, we can make the route make a full transition when any
of the `queryParams` change, to do so we'll need to specify in the
route a property called `queryParams` where we explicitly mark
the parameters which we want to cause a full transition.

{title="app/routes/friends/index.js", , lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  queryParams: {
    sortBy: {
      refreshModel: true
    },
    sortAscending: {
      refreshModel: true
    }
  },
  model: function(params) {
    return this.store.find('friend', params);
  }
});
~~~~~~~~

Now every time we change `sortBy` or `sortAscending` the model hook
for **app/routes/friends/index.js** will be called, making requests to
the API similar `/api/v4/friends?sortBy=fullName&sortAscending=true`.

## Further Reading

Query parameters is one of the best documented features on Ember, we
recommend the official guides for more information [http://emberjs.com/guides/routing/query-params/](http://emberjs.com/guides/routing/query-params/).

X>## Tasks
X>Use query parameters on articles index to show or hide articles
X>depending on their state, if the query parameter `showReturned` is
X>true then all the articles are displayed otherwise show the only the
X>ones in state `borrowed`.
X>
X> Tip: Use `this.store.filter` to select the articles.
