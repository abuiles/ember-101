# Ember Data
In this chapter we'll cover some of the public methods from the
[DS.Store](http://emberjs.com/api/data/classes/DS.Store.html) and also
learn how to load relationships asynchronously.

## DS.Store Pubic API

The Store is the main interface we'll be using to interact with our
records and the backend,  when we create, load or delete a record it
is managed and saved in the store, it then take cares of replicating
any change to the backend.

We won't cover all of the functions but the ones more used and their
gotchas.

#### all

**store.all** is similar to **store.find** but instead of making a
request to the backend it will return all the records already loaded
in the store, the result of this method is a **live array** meaning it
will update its content if more records are loaded into the store for
the given type.

Let's study this with the inspector navigating to
http://localhost:4200 and clicking refresh, then we'll grab an
instance of the application route and run the following commands in the
console:

~~~~~~~~
friends = $E.store.all('friend')
friends.get('length')
> 0
friends.mapBy('firstName')
> []
~~~~~~~~

We stored the result in a variable called friends, and we'll noticed
that the length of the collection is zero, it makes sense since we
haven't loaded any **friends** yet, if we click on the friends link
and then run

~~~~~~~~
friends.get('length')
> 3
friends.mapBy('firstName')
> ["zombo Wamba", "Pizza", "Loading-this"]
~~~~~~~~

We'll see that the result is different from zero, when we
navigated to the friends route a request to the backend was made and
some records were loaded into the store, as we mentioned the result
from **all** is a **live array**, that's why our **friends** variable
got updated without requiring us to do anything extra.


### filter

The **filter** function behaves similar to **find** but it takes an additional
parameter which is a **filter function**, it will call the function for
every record on the result and the return the ones for which the
function returns true.

By default filter we'll work against elements already loaded into the
store (like **all**) but if we want to force a request to the backend
we can pass an object and it will make a request with every property
on the object as a parameter.

Again, let's see this working on our application navigating to
http://localhost:4200/friends and putting the following in the
console:

~~~~~~~~
friends = $E.store.filter('friend', function(friend){
   return friend.get('totalArticles') % 2 == 0
})
friends.mapBy('firstName')
~~~~~~~~

The previous call to filter will take every friend already loaded in
the store and then select the ones who have borrowed an even number of
articles.

Let's suppose our API supports a parameter **hasArticles** which
return only the friends who have currently an article and that we want
to filter them by the ones who have an even number of articles.

We could write the filter as follows:

~~~~~~~~
friends = $E.store.filter('friend', {hasArticles: true}, function(friend){
   return friend.get('totalArticles') % 2 == 0
})
> GET "http://localhost:4200/api/v2/friends?hasArticles=true".
~~~~~~~~

If we inspect our network tab, we'll see that the **GET** request
**http://localhost:4200/api/v2/friends?hasArticles=true** was made to
the server.

Once the records are loaded, it will apply the filter and return the
values returning true for the given filter function.

The result from a filter function is a **live array** too, in our
example if any of our friends borrow a new article and the total
number of articles is even, then it will disappear from our result.

I> XHR login in the console is a great way to debug our applications, we can
I> enable it using the setting in Chrome's DevTools, see  [https://www.igvita.com/slides/2012/devtools-tips-and-tricks/#4](https://www.igvita.com/slides/2012/devtools-tips-and-tricks/#4)

### find

We have already used **find** to load records, **find** behaves
differently based on the arguments we pass and the data available in
the store, there are 2 scenarios one for loading a collection of
records and the other one for a single record.

#### Scenario #1: Loading a collection

If we call **find** only with a model name then it will make a request
to load a list of records of that type, the following is an example of that:

~~~~~~~~
friends =  $E.store.find('friend')

XHR finished loading: GET "http://localhost:4200/api/v2/friends".
~~~~~~~~

If we want to send query parameters with our request, then we can pass
an object as second argument and every key  on the object will
be included as parameter:

~~~~~~~~
friends =  $E.store.find('friend', {hasArticles: true, sort_by: 'created_at'})

XHR finished loading: GET "http://localhost:4200/api/v2/friends?hasArticles=true&sort_by=created_at".
~~~~~~~~

In the previous request we asked **find** to load all the articles,
sending as parameters the key **hasArticles** and **sort_by**.

Like **all** and **filter** the result from **find** is a **live
array** too. When called it will make a request to the server and then
the collection will be updated if more records are added or removed
from the store.

#### Scenario #2: Loading a single record

We can also use **find** to load an specific record, to do that we'll
only need to pass the record **id** as second argument:

~~~~~~~~
$E.store.find('friend', 15)
XHR finished loading: GET "http://localhost:4200/api/v2/friends/15".
~~~~~~~~

In the previous example we are loading the friend with id 15, the
**store** will only make a request to the server if the friend is not
available in the store, to understand this, let's go to
http://localhost:4200/friends and then on the console try the following:

~~~~~~~~
id = $E.store.all('friend').get('firstObject').id
$E.store.find('friend', id)
~~~~~~~~

If we open our network tab, we'll see that the store didn't make any request
this time, the reason is that we asked for a friend which was already
loaded into the store.

Is important to mention that **find**, **all** and **filter** return
promises, when testing on the browser's console we don't have to worry
about it, but if we want to use the result in our application then we
need to keep this in mind.

### getById

We can use **store.getById('friend', 15)** to fetch an user directly
from the store, unlike **find** the behavior of this function is
synchronous, it will return null if the record is available or null otherwise.

### metadataFor

If our API includes a "meta" key with a response we can access such
edata with the **metadataFor** function. This is very useful when we
are implementing things like pagination.

Suppose the response from our API is something like the following when
fetching all the friends:

~~~~~~~~
{
  friends: [ ... ],
  meta: { total: 30}
}
~~~~~~~~

We can then read the meta key doing **this.store.metadataFor('friend')**;


### createRecord

We are already familiar with **createRecord**, it is used when we want
to create a new record for a given type, we have used previously like:

~~~~~~~~
this.store.createRecord('friend', {attrs..});
~~~~~~~~

We can also use createRecord through a relationship, suppose we are in
the context of a friend, we know they have an **articles** property
which represents all the articles belonging to a friend, if we want to
add a new article we could also do it using the following syntax:

~~~~~~~~
friend.get('articles').createRecord({attrs...});
~~~~~~~~

The previous won't work if the relationship is **async**

## Loading relationships

We already covered how to specify relationships between models, if we
are defining a relationship of type has many then we use the keyword
**hasMany** and if we want a belongs to we use **DS.belongsTo**.

We also switch to use v2 of the API which side-loads all the articles
records for our friend but didn't stop to understand what was
happening.


There are two ways to work with relationships in Ember, the first one
is working with records pre-loaded into store and the second one is
to load them on demand.

With the first strategy on the payload we'll specify the ids of the
records the model is related with, Ember-Data will look for those
records and fill up the association automatically, under this model
the records which are part of the association need to be loaded into
the **Store** or be **side-loaded** with the parent.

If we inspect the payload for friends coming for the version 2 of
the API, they look something like the following:

{title="", lang="JavaScript"}
~~~~~~~~
{
  id: 48,
  first_name: "zombo",
  last_name: "Pombo",
  email: "zombo@pombo.com",
  twitter: "zombo",
  total_articles: 2,
  article_ids: [
    40,
    41
  ]
}
~~~~~~~~

It includes the model's attributes and then there is a key called
**article_ids** this is what Ember-Data uses to bind the models.

Ember-Data expects the **articles** with ids 40 and 41 to be into the
**Store**, if we do **store.getById('article', 40)** it has to
return a value or it expects to have a key in the response called
**articles** which includes the **articles** with id 40 and 41.

If the records are not present then we'll get an error like the
following:

~~~~~~~~
route: articles.index Assertion Failed: You looked up the 'articles'
relationship on a 'friend' with id 48 but some of the associated
records were not loaded. Either make sure they are all loaded together
with the parent record, or specify that the relationship is async
(`DS.hasMany({ async: true })`)
~~~~~~~~


The **payload** when side-loading a relationship looks like the
following:

~~~~~~~~
{
  friend: {
    id: 48,
    first_name: "Wamba",
    last_name: "Pombo",
    email: "zombo@pombo.com",
    twitter: "zombo",
    total_articles: 2,
    article_ids: [
      41
    ]
  },
  articles: [
    {
      id: 41,
      created_at: "2014-10-17T16:04:51.884Z",
      description: "Pombo Set",
      state: "borrowed",
      notes: null,
      friend_id: 48
    }
  ]
}
~~~~~~~~

The response above is bringing the **friend** record with id **48** and
then including all their articles.


This strategy for loading records works well if we know that all the
records the association depends on are already on the store or if
the number of records to side-load are not a lot.

But what if we want to load thousands of relationships and on top of
that we want to implement strategies like pagination or search? Enter
**async** relationships.

On the error thrown by ember-data when it couldn't find the records
the following was included: **specify that the relationship is async
(DS.hasMany({ async: true })**.

## Working with async relationships in Ember-Data

Ember-Data offers support to work with relationships in
an asynchronous way only marking the attribute as async, then we can
include the ids or an URL to load the records from.

First it loads the parent record and then it will only load the
records in the relationship when we explicitly call the attribute,
that would be when we do something like **friend.get('articles')**, in
that moment Ember-Data will check if the **articles** are already
loaded and if not then it will make a **GET** request, if the ids in
the relationships are 40 and 41, then the **GET** requests is going to
be something like **/api/friends?ids=40,41**.


Let's try this on our applications, first, we'll update our
**application adapter** to use **v3** of the api, let's change
**app/adapters/application.js**:

{title="Using borrowers API V3", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';

export default DS.ActiveModelAdapter.extend({
  namespace: 'api/v3'
});
~~~~~~~~

If we check the response from
[http://api.ember-cli-101.com/api/v3/friends.json](http://api.ember-cli-101.com/api/v3/friends.json)
we'll notice that this time the articles are not being side-loaded.

Next we need to update our friend model, we'll add the object
**{async: true)** as second argument to the **hasMany** attribute for
articles:

{title="Specifying articles as async", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';
import Ember from 'ember';

export default DS.Model.extend({
  articles:      DS.hasMany('articles', {async: true}),
  email:         DS.attr('string'),
  firstName:     DS.attr('string'),
  lastName:      DS.attr('string'),
  totalArticles: DS.attr('number'),
  twitter:       DS.attr('string'),
  fullName: Ember.computed('firstName', 'lastName', function() {
    return this.get('firstName') + ' ' + this.get('lastName');
  })
});
~~~~~~~~

With that we have switch our model from working with side-load
relationships to **async**.

Let's explore how **async** relations behave, let's navigate to
http://localhost:4200/friends, click in any of our friends and open
the console, we'll see something like the following:

~~~~~~~~
XHR finished loading: GET "http://localhost:4200/api/v3/articles/34".
XHR finished loading: GET "http://localhost:4200/api/v3/articles/35".
XHR finished loading: GET "http://localhost:4200/api/v3/articles/36".
XHR finished loading: GET "http://localhost:4200/api/v3/articles/16".
~~~~~~~~

This time we didn't get the error because our articles were not
loaded, instead Ember-Data made a **GET** request for every of our
friends.

Will Ember-Data make 10000 request to our API if our friend has
10000 items? No, we can tell Ember-Data to coalesce all those calls
into a single request setting the adapter's property
**coalesceFindRequests** to true, let's change
**app/adapters/application.js** to the following:

{title="Enable coalesceFindRequests", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';

export default DS.ActiveModelAdapter.extend({
  namespace: 'api/v3',
  coalesceFindRequests: true
});
~~~~~~~~

If we refresh the route, we'll see this time the following GET
request:

~~~~~~~~
XHR finished loading: GET
"http://localhost:4200/api/v3/articles?ids%5B%5D=34&ids%5B%5D=35&ids%5B%5D=36&ids%5B%5D=16".
~~~~~~~~

Now it is making a single request to the API passing the query
parameter **ids** with all the articles that it needs to load.

I> In the following commit we can check the backend implementation to load a list of articles if the
I> ids parameter is present: [abuiles/borrowers-backend- Add version 3](https://github.com/abuiles/borrowers-backend/commit/857cb40e654b8243b6e842a2bc78408cd50a9f4d#diff-b4f73470ac000871615a9c310e2537fcR5)



### Using links instead of ids.

There is another way of loading relationships asynchronously in
Ember-Data without specifying the ids, we can return a property called
**links** with an object including an URL for every of the
relationships to load asynchronously, then Ember-Data will make a
request to the URL when we ask for the relationship records.

We'll move to version 4 of our API which specifies the relationships using links.

To try in our application, we'll update our **application adapter** to
use **v4** of the api, let's change **app/adapters/application.js**:


{title="Using borrowers API V4", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';

export default DS.ActiveModelAdapter.extend({
  namespace: 'api/v4',
  coalesceFindRequests: true
});
~~~~~~~~

If we look at the payload for v4
http://api.ember-cli-101.com/api/v4/friends.json, we'll noticed that
they look like the following:

{title="JSON payload with links", lang="JavaScript"}
~~~~~~~~
{
  id: 48,
  first_name: "Zombo",
  last_name: "Pombo",
  email: "zombo@pombo.com",
  twitter: "zombo",
  total_articles: 2,
  links: {
    articles: "/api/v4/articles?friend_id=48"
  }
}
~~~~~~~~

This time we don't have ids, but an URL where Ember-Data can load the
relationship from. If we go to a friend profile, we'll see the request
**GET "http://localhost:4200/api/v4/articles?friend_id=48"** in the
network tab.

There is an important thing to mention and is that the request to load
**async** data will only happen once, if we visit a friend profile, go
back to the friends index and then visit that friend profile again, we
won't see a request to fetch the articles because Ember-Data we'll
identify such request as already fulfilled.

If we always want to load the records from the model hook on the
**Articles Index Route** then we can put a guard, check if the request
is fulfilled and if that's the case then force a reload, we can use
something like the following:

{title="app/routes/articles/index.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  model: function() {
    var articles = this.modelFor('friends/show').get('articles');

    //
    // The return value from an async relationship is a PromiseArray
    // the property isFulfilled will become true when the proxied
    // promise has been fulfilled, in this case that would when we
    // get a response from the API.
    //

    if (articles.get('isFulfilled')) {
      articles.reload();
    }

    return articles;
  },
  actions: {
    save: function(model) {
      model.save();
      return false;
    }
  }
});
~~~~~~~~~

If we try again, we'll see that a request is always made to the API to
fetch the articles whenever we navigate to a friend profile.

The property **isFulfilled** is part of a set of properties included
in the **PromiseArray** via the
[(Ember.PromiseProxyMixin](http://emberjs.com/api/classes/Ember.PromiseProxyMixin.html#property_isFulfilled)

It has the following properties that we can use to guide the flow of
our application.


|isFulfilled |
|isPending   |
|isRejected  |
|isSettled   |



## What to use then?

So many options, what should we use then? it depends on our scenarios
and how do we want to load our data, side-loading works perfectly when
we are not fetching many records, but it can make your API really
slow if you are returning a lot of relationships and a lot of records.

Async helps us alleviate the issue when we have a lot of records and
can help us keep our end-points lighter, but it might add some kind of
overhead when getting all the ids in a relationship.

The faster option from an API point of view would be to use links
since it won't require the parent to know anything about its children
but then you lose other benefits.

One example of things you lose when using links instead of ids is the
following, when using ids Ember-Data will only load from the server
the records which are not yet available in the store, but if some of
the records are loaded, then it won't make that request. With links
you lose that benefits because Ember-Data doesn't have any information
so it will make the request and load data which you might have already
available.

Again is a matter of trade-offs and finding what works best for us, we
need to measure and experiment with different strategies and then go
with the one which gives us the best performance.
