# Ember Data
In this chapter we'll cover some of the public methods from the
[DS.Store](http://emberjs.com/api/data/classes/DS.Store.html) and
learn how to load relationships asynchronously.

## DS.Store Pubic API

The store is the main interface we'll use to interact with our
records as well as the backend.  When we create, load, or delete a record, it
is managed and saved in the store. It then takes care of replicating
any change to the backend.

We won't cover all of the functions, but we'll go over the more common ones and their
gotchas.

#### all

**store.all** is similar to **store.find**, but instead of making a
request to the backend it returns all the records already loaded
in the store. The result of this method is a **live array**, which means it
will update its content if more records are loaded into the store for
the given type.

Let's study this with the inspector by navigating to
http://localhost:4200 and clicking refresh. Next we'll grab an
instance of the application route and run the following commands in the
console:

~~~~~~~~
friends = $E.store.all('friend')
friends.get('length')
> 0
friends.mapBy('firstName')
> []
~~~~~~~~

We stored the result in a variable called friends, and we'll notice
that the length of the collection is zero. This makes sense because we
haven't loaded any **friends** yet. Click on the friends link
and run the following:

~~~~~~~~
friends.get('length')
> 3
friends.mapBy('firstName')
> ["zombo Wamba", "Pizza", "Loading-this"]
~~~~~~~~

We'll see that the result is no longer zero. When we
navigated to the friends route, a request to the backend was made and
some records were loaded into the store. As we mentioned, the result
from **all** is a **live array**. That's why our **friends** variable
was updated without requiring any additional steps.


### filter

The **filter** function behaves similarly to **find**, but it takes an additional
parameter known as the **filter function**. This will call the function for
every record in the result and return the ones for which the
function returns true.

By default, filter will work against elements already loaded into the
store (like **all**). If we want to force a request to the backend,
we can pass an object and it will make a request with every property
on the object as a parameter.

Again, let's see this working on our application by navigating to
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

Let's suppose our API supports a parameter **hasArticles**, which
return only the friends who currently have an article. Let's say we want
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

The result from a filter function is a **live array** as well. In our
example, if any of our friends borrow a new article and the total
number of articles is even, then it will disappear from our result.

I> XHR login in the console is a great way to debug our applications. We can
I> enable it using the setting in Chrome's DevTools. See  [https://www.igvita.com/slides/2012/devtools-tips-and-tricks/#4](https://www.igvita.com/slides/2012/devtools-tips-and-tricks/#4).

### find

We have already used **find** to load records. **find** behaves
differently based on the arguments we pass and the data available in
the store. There are two scenarios. One for loading a collection of
records and the other for a single record.

#### Scenario #1: Loading a collection

If we call **find** only with a model name, then it will make a request
to load a list of records of that type. The following is an example:

~~~~~~~~
friends =  $E.store.find('friend')

XHR finished loading: GET "http://localhost:4200/api/v2/friends".
~~~~~~~~

If we want to send query parameters with our request, then we can pass
an object as second argument and every key on the object will
be included as parameter:

~~~~~~~~
friends =  $E.store.find('friend', {hasArticles: true, sort_by: 'created_at'})

XHR finished loading: GET "http://localhost:4200/api/v2/friends?hasArticles=true&sort_by=created_at".
~~~~~~~~

In the previous request we asked **find** to load all the articles,
sending as parameters the keys **hasArticles** and **sort_by**.

Like **all** and **filter**, the result from **find** is a **live
array**. When called, it makes a request to the server and
the collection is updated when more records are added to or removed
from the store.

#### Scenario #2: Loading a single record

We can also use **find** to load a specific record. To do that, we'll
only need to pass the record **id** as second argument:

~~~~~~~~
$E.store.find('friend', 15)
XHR finished loading: GET "http://localhost:4200/api/v2/friends/15".
~~~~~~~~

In the previous example, we loaded the friend with id 15. The
**store** will only make a request to the server if the friend is not
available in the store. To understand this, let's go to
http://localhost:4200/friends and try the following in the console:

~~~~~~~~
id = $E.store.all('friend').get('firstObject').id
$E.store.find('friend', id)
~~~~~~~~

If we open our network tab, we'll see that the store didn't make any requests
this time. This is because we asked for a friend who was already
loaded into the store.

It's important to mention that **find**, **all**, and **filter** return
promises. When testing on the browser's console, we don't have to worry
about it, but if we want to use the result in our application then we
need to keep this in mind.

### getById

We can use **store.getById('friend', 15)** to fetch a user directly
from the store. Unlike **find**, the behavior of this function is
synchronous. It will return null if the record is available, or null otherwise.

### metadataFor

If our API includes a "meta" key with a response, we can access such
edata with the **metadataFor** function. This is useful when we
implement things like pagination.

Suppose the response from our API is something like the following when
we fetch all of our friends:

~~~~~~~~
{
  friends: [ ... ],
  meta: { total: 30}
}
~~~~~~~~

We can then read the meta key doing **this.store.metadataFor('friend')**;


### createRecord

We are already familiar with **createRecord**, which is used when we want
to create a new record for a given type. For example:

~~~~~~~~
this.store.createRecord('friend', {attrs..});
~~~~~~~~

We can also use createRecord via a relationship. Suppose we are in
the context of a friend and we know they have an **articles** property
that represents all the articles belonging to another friend. If we want to
add a new article, we can do it using the following syntax:

~~~~~~~~
friend.get('articles').createRecord({attrs...});
~~~~~~~~

This won't work if the relationship is **async**.

## Loading relationships

We already covered how to specify relationships between models. If we
are defining a relationship of type has many, then we use the keyword
**hasMany**. If we want a belongs to, we use **DS.belongsTo**.

We switch to v2 of the API, which side-loads all the articles
records for our friend but doesn't stop to understand what is
happening.


There are two ways to work with relationships in Ember. The first
is working with records pre-loaded into the store, and the second is
to load them on demand.

With the first strategy on the payload we'll specify the ids of the
records the model is related to. Ember-Data looks for those
records and fills up the association automatically. Under this model,
the records that are part of the association need to be loaded into
the **Store** or **side-loaded** with the parent.

If we inspect the payload for friends arriving in version 2 of
the API, the results look something like the following:

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

This includes the model's attributes and a key called
**article_ids**. This is what Ember-Data uses to bind the models.

Ember-Data expects the **articles** with ids 40 and 41 to be in the
**Store**. If we do **store.getById('article', 40)**, it
returns a value or else expects to have a key in the response called
**articles** that includes the **articles** with id 40 and 41.

If the records are not present, we'll get the following error:

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

The response above brings the **friend** record with id **48** and
includes all of their articles.


This strategy for loading records works well if we know that all the
records the association depends on are already in the store, or if
there is a low number of records to side-load.

What if we want to load thousands of relationships in addition to implementing strategies like pagination or search? Enter **async** relationships.

In the previous error thrown by ember-data, the following was included: 
**specify that the relationship is async
(DS.hasMany({ async: true })**.

## Working with async relationships in Ember-Data

Ember-Data offers support for working with asynchronous relationships. All we have to do is mark the attribute as async. Then we can
include the ids or an URL from which to load the records.

First it loads the parent record. Then it will load the
records in the relationship, but only when we explicitly call the attribute.
For example, if we call **friend.get('articles')**, Ember-Data will check if the **articles** are already
loaded. If they are not, it will make a **GET** request. If the ids in
the relationships are 40 and 41, then the **GET** request is going to
be something like **/api/friends?ids=40,41**.


Let's try this on our applications. First we'll update our
**application adapter** to use **v3** of the API. Let's change
**app/adapters/application.js**:

{title="Using borrowers API V3", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';

export default DS.ActiveModelAdapter.extend({
  namespace: 'api/v3'
});
~~~~~~~~

If we check the response from
[http://api.ember-cli-101.com/api/v3/friends.json](http://api.ember-cli-101.com/api/v3/friends.json),
we'll notice that this time the articles are not being side-loaded.

Next we need to update our friend model. We'll add the object
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

We just switched our model from working with side-load
relationships to **async**.

Let's explore how **async** relations behave. If we navigate to
http://localhost:4200/friends, click on any of our friends, and open
the console, we'll see something like the following:

~~~~~~~~
XHR finished loading: GET "http://localhost:4200/api/v3/articles/34".
XHR finished loading: GET "http://localhost:4200/api/v3/articles/35".
XHR finished loading: GET "http://localhost:4200/api/v3/articles/36".
XHR finished loading: GET "http://localhost:4200/api/v3/articles/16".
~~~~~~~~

This time we didn't get the error because our articles were not
loaded. Instead, Ember-Data made a **GET** request for each of our
friends.

Will Ember-Data make 10,000 requests to our API if our friend has
10,000 items? No. We can tell Ember-Data to coalesce all those calls
into a single request by setting the adapter's property
**coalesceFindRequests** to true. Let's change
**app/adapters/application.js** to the following:

{title="Enable coalesceFindRequests", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';

export default DS.ActiveModelAdapter.extend({
  namespace: 'api/v3',
  coalesceFindRequests: true
});
~~~~~~~~

If we refresh the route, this time we'll see the following GET
request:

~~~~~~~~
XHR finished loading: GET
"http://localhost:4200/api/v3/articles?ids%5B%5D=34&ids%5B%5D=35&ids%5B%5D=36&ids%5B%5D=16".
~~~~~~~~

Now it makes a single request to the API by passing the query
parameter **ids** with all the articles that it needs to load.

I> In the following commit, we can check the backend implementation to load a list of articles if the
I> ids parameter is present: [abuiles/borrowers-backend- Add version 3](https://github.com/abuiles/borrowers-backend/commit/857cb40e654b8243b6e842a2bc78408cd50a9f4d#diff-b4f73470ac000871615a9c310e2537fcR5).



### Using links instead of ids.

There is another way to load relationships asynchronously in
Ember-Data without specifying the ids. We can return a property called
**links** with an object including an URL for each of the
relationships to load asynchronously. Ember-Data will then make a
request to the URL when we ask for the relationship records.

We'll move to version 4 of our API, which specifies the relationships using links.

To try this in our application, we'll update **application adapter** to
use **v4** of the API. Let's change **app/adapters/application.js**:


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
it looks like the following:

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

This time we don't have ids but an URL from which Ember-Data can load the
relationship. If we go to a friend's profile, we'll see the request
**GET "http://localhost:4200/api/v4/articles?friend_id=48"** in the
network tab.

One important item to mention is that the request to load
**async** data will only happen once. If we visit a friend's profile, go
back to the friends index, and then visit that friend profile again, we
won't see a request to fetch the articles because Ember-Data will
identify such a request as already fulfilled.

If we always want to load the records from the model hook on the
**Articles Index Route**, then we can put a guard, check if the request
is fulfilled, and if that's the case then force a reload. We can use
something like the following:

{title="app/routes/articles/index.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  model: function() {
    var articles = this.modelFor('friends/show').get('articles');

    //
    // The return value from an async relationship is a PromiseArray.
    // The property isFulfilled will become true when the proxied
    // promise has been fulfilled. In this case, that would be when we
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

If we try again, we'll see that navigating to a friend's profile will always cause a request to the API to
fetch the articles.

The property **isFulfilled** is part of a set of properties included
in the **PromiseArray** via the
[(Ember.PromiseProxyMixin](http://emberjs.com/api/classes/Ember.PromiseProxyMixin.html#property_isFulfilled).

It has the following properties that we can use to guide the flow of
our application.


|isFulfilled |
|isPending   |
|isRejected  |
|isSettled   |



## What to use?

So many options. What should we use? It depends on our scenarios
and how we want to load our data. Side-loading works perfectly when
we are not fetching many records, but it can make your API really
slow if you are returning a lot of relationships and a lot of records.

Async helps us alleviate the issue when we have a lot of records. This
can help us keep our end-points lighter, but it might add some
overhead when getting all the ids in a relationship.

The faster option from an API point of view would be to use links.
This won't require the parent to know anything about its children,
but then we lose other benefits.

For example, when using ids, Ember-Data will only load records from the server
that are not yet available in the store. However, if some of
the records are loaded, it won't make that request. With links,
you lose that benefit because Ember-Data doesn't have any information.
It will make the request and load data that you might already have
available.

Again, it's a matter of weighing risks and benefits and finding what works best for us. We
need to measure and experiment with different strategies before choosing
the one that gives us the best performance.
