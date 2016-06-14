# Ember Data
In this chapter we'll cover some of the public methods from the
[DS.Store](http://emberjs.com/api/data/classes/DS.Store.html) and
learn how to load relationships asynchronously.

## DS.Store Public API

The store is the main interface we'll use to interact with our records
as well as the backend.  When we create, load, or delete a record, it
is managed and saved in the store. The store then takes care of
replicating any change to the backend.

We won't cover all of the functions, but we'll go over the more common
ones and their gotchas.

#### peekAll

**store.peekAll** is similar to **store.findAll**, but instead of
making a request to the backend it returns all the records already
loaded in the store. The result of this method is a **live array**,
which means it will update its content if more records are loaded into
the store for the given type.

Let's study this with the inspector by navigating to
http://localhost:4200 and clicking refresh. Next we'll grab an
instance of the application route and run the following commands in the
console:

~~~~~~~~
friends = $E.store.peekAll('friend')
friends.get('length')
> 0
friends.mapBy('firstName')
> []
~~~~~~~~

We stored the result in a variable called friends, which is a
collection with zero element. This makes sense because we haven't
loaded any **friends** yet. If we click on the friends link and run the
following:

~~~~~~~~
friends.get('length')
> 3
friends.mapBy('firstName')
> ["zombo Wamba", "Pizza", "Loading-this"]
~~~~~~~~

We'll see that the result is no longer zero. When we navigated to the
friends route, a request to the backend was made and some records were
loaded into the store. As we mentioned, the result from **peekAll** is
a **live array**. That's why our **friends** variable was updated
without requiring any additional steps.

I> XHR logging in the console is a great way to debug our applications. We can
I> enable it using the setting in Chrome's DevTools. See slide #4 in
I> the presentation [Wait, DevTools could do THAT? by Ilya Grigorik](https://www.igvita.com/slides/2012/devtools-tips-and-tricks/#4](https://www.igvita.com/slides/2012/devtools-tips-and-tricks/#4).

### findAll

If we call **findAll** with a model name, then it will make a request
to load a list of records of that type. The following is an example:

~~~~~~~~
friends =  $E.store.findAll('friend')

XHR finished loading: GET "http://localhost:4200/api/v2/friends".
~~~~~~~~

If we want to send query parameters with the request, then we should
use `store.query`, it receives the name of the model and an object
as second argument, every key on the object will be included as
parameter:

~~~~~~~~
friends =  $E.store.query('friend', {sort: 'first-name'})

XHR finished loading: GET "http://localhost:4200/api/v2/friends?&sor=first-name".
~~~~~~~~

In the previous request we asked **query** to load all the articles,
sending as parameters the key **sort**.

Like **peekAll** and **filter**, the result from **findAll** is a
**live array**. When called, it makes a request to the server and the
collection is updated when more records are added to or removed from
the store.

## findRecord: Loading a single record

If we want to load a single record then we should use
[store.findRecord](http://emberjs.com/api/data/classes/DS.Store.html#method_findRecord) . To do that, we use the name of the model and
the record's **id** as second argument:

~~~~~~~~
$E.store.findRecord('friend', 15)
XHR finished loading: GET "http://localhost:4200/api/v2/friends/15".
~~~~~~~~

### peekRecord

We can use **store.peekRecord('friend', 15)** to fetch a user directly
from the store. Unlike findRecord, query or findAll, the behavior of
this function is synchronous. It will return the record if it is
available, or `null` otherwise.

### createRecord

We are already familiar with **createRecord**, which is used when we
want to create a new record for a given type. For example:

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

We already covered how to create relationships between models. If we
are defining a relationship of type "has many", then we use the keyword
**hasMany**. If we want a "belongs to", we use **DS.belongsTo**.

There are two ways to work with relationships. The first is working
with records pre-loaded into the store, and the second is to load them
on demand.

As we know, our API follows JSON API, which gives us different
strategies to load the records associated with a relationship. In our
API, we use the
[links strategy](http://jsonapi.org/format/#document-resource-object-relationships).
Ember data follows those links to fill up the association
automatically.

If we inspect the payload for friends, the results look something like
the following:

{title="", lang="JavaScript"}
~~~~~~~~
{
  "attributes": {
    "created-at": "2016-06-12T13:47:02.427Z",
    "email": "arya@got.com",
    "first-name": "Arya",
    "last-name": "Stark",
    "twitter": "@noone"
  },
  "id": "64",
  "links": {
    "self": "https://api.ember-101.com/friends/64"
  },
  "relationships": {
    "loans": {
      "links": {
        "related": "https://api.ember-101.com/friends/64/loans",
        "self": "https://api.ember-101.com/friends/64/relationships/loans"
      }
    }
  },
  "type": "friends"
}
~~~~~~~~

This includes the model's attributes and a key called `relationships`
which includes the links where the related attributes can be fetched.

Ember data then follows that link to get the list of loans for a
friend.

We can see how this works if we visit `http://localhost:4200/friends`
and then click on a friend, we'll see that there are several request
after that one to fetch the related loans and then more request to
fetch the related articles.

We can avoid all those request sideloading the related records,
JSONAPI give us a way to do so, which is using the parameter `include`
and then a path for the things we want to include.

Let's modify the index route so it include the query parameter `include` with the value `loans`:

{line-numbers=off, title="app/routes/friends/index.js", lang=""}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    //
    // We use now store.query and pass include in the options
    //

    return this.store.query('friend', {include: 'loans'});
  }
});
~~~~~~~~

Now if we go to to index, we'll see that the request for the "loans"
relationship is not happening, but there are request for articles.

We can sideload the articles too, but we need to ask for that in the
`include` query params. Let's modify our route once more so it looks
like the following:

{line-numbers=off, title="app/routes/friends/index.js", lang=""}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    //
    // We use now store.query and pass include in the options
    //

    return this.store.query('friend', {include: 'loans,loans.article'});
  }
});
~~~~~~~~

If we go once more to friends and then click on one of them, we won't
see extra request happening.

We can see the difference in the payload visiting the following page
[https://api.ember-101.com/friends?include=loans](https://api.ember-101.com/friends?include=loans)
and then adding or removing things to the query param `include`.

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

For example, when using ids, ember data will only load records from
the server that are not yet available in the store. However, if some
of the records are loaded, it won't make that request. With links, you
lose that benefit because ember data doesn't have any information.  It
will make the request and load data that you might already have
available.

Again, it's a matter of weighing risks and benefits and finding what
works best for us. We need to measure and experiment with different
strategies before choosing the one that gives us the best performance.
