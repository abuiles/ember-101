# Testing Ember.js applications

In this we'll cover the basis of unit and acceptance testing in
Ember.js applications and recommend a couple of resources that can
helps us expand our knowledge on this area.

## Unit Testing

When we run the generators by default they will create unit test files
too, we can see all the generated unit test if we go to `tests/unit`:

{title="Unit tests", lang="bash"}
~~~~~~~~
$ ls tests/unit/ adapters
helpers routes controllers models utils
~~~~~~~~

By default tests are grouped by type, if we open the unit test for our
model friend, we'll see the following:

{title="tests/unit/model/friend-tests.js", lang="JavaScript"}
~~~~~~~~
import { test, moduleForModel } from 'ember-qunit';

moduleForModel('friend', 'Friend', { needs: ['model:article'] });

test('it exists', function() { var model = this.subject(); ok(model);
});
~~~~~~~~

At the beginning of the test we are importing a set of helpers from
[ember-qunit](https://github.com/rwjblue/ember-qunit), which is
library wrapping a bunch of functions to facilitate testing with
**QUnit**.

`moduleForModel` received the name of the model we are testing, a
description and then some options, in our scenario we are specifying
that the tests need a model called **article** because of the existing
relationship between them.

Then it includes a basic assertion that the model exist,
`this.subject()` would be an instance of a `friend`.

We have 2 ways of running test, the first one is via the browser while
we are running the development server, we can navigate to
[http://localhost:4200/tests](http://localhost:4200/tests) and our
tests will be run and the second one is using a tests runner, at the
moment **ember-cli** has built-in support for **Testem** with
[PhantomJS](http://phantomjs.org/) which we can use to run our test on
a CI server, to run the test in this mode we only need to do `npm
test`.

Let's write two more test for our model friend, we want to check that
the computed property `fullName` behaves as expected and that the
relationship articles is properly set.

{title="tests/unit/model/friend-tests.js", lang="JavaScript"}
~~~~~~~~
import { test, moduleForModel } from 'ember-qunit';
import Ember from 'ember';

moduleForModel('friend', 'Friend', {
  needs: ['model:article']
});

test('it exists', function() {
  var model = this.subject();
  ok(model);
});

test('fullName concats first and last name', function() {
  var model = this.subject({firstName: 'Syd', lastName: 'Barrett'});

  equal(model.get('fullName'), 'Syd Barrett');

  Ember.run(function() {
    model.set('firstName', 'Geddy');
  });

  equal(model.get('fullName'), 'Geddy Barrett', 'Updates fullName');
});

test('articles relationship', function() {
  var klass  = this.subject({}).constructor;

  var relationship = Ember.get(klass, 'relationshipsByName').get('articles');

  equal(relationship.key, 'articles');
  equal(relationship.kind, 'hasMany');
});
~~~~~~~~

We can run our tests going directly to the following URL
[http://localhost:4200/tests?module=Friend](http://localhost:4200/tests?module=Friend).

The first tests verifies that `fullName` is being calculated
correctly, we have to wrap `model.set('firstName', 'Geddy');` in
`Ember.run` since it has an asynchronous behavior, it we modify the
implementation for `fullName` such as it doesn't return first and last
name the tests will fail.

The second test is just checking that we have setup the proper
relationship to `articles` something similar could go in the articles
model tests. If we call `constructor` on an instance to a model, that
would give us access to the class they are an instance of.


Let's add other unit test for `app/utils/date-helpers`:

{title="tests/unit/utils/date-helpers-test.js", lang="JavaScript"}
~~~~~~~~
import { formatDate } from 'borrowers/utils/date-helpers';

module('Utils: formatDate');

test('formats a date object', function() {
  var date = new Date("11-3-2014");
  var result = formatDate(date);

  equal(result, 'Mon Nov 03 2014', 'returns a readable string');
});
~~~~~~~~

We are importing the function we want to test and then we check that
it returns the date as a readable string, we can run the test going to
[http://localhost:4200/tests?module=Utils%3A%20formatDate](http://localhost:4200/tests?module=Utils%3A%20formatDate).

## Acceptance Tests

With acceptance tests we can verify workflows in our application, for
example making sure that we can add a new friend, that if we visit the
friend index then a list is rendered, etc. It basically emulates a
real user using our application.


Ember has a set of helper that make super simple writing this kind of
tests, there are
[synchronous](http://emberjs.com/guides/testing/test-helpers/#toc_wait-helpers)
and
[asynchronous](http://emberjs.com/guides/testing/test-helpers/#toc_asynchronous-helpers)
helpers , we use the former ones for tests that don't have any kind of
side-effect like checking if an element is present in page and the
latter for tests that fire some kind of side-effect for example
clicking a link or saving a model.

Let's write an acceptance test to verify that we can add new friends
to our application we can generate an acceptance test with the
generator **acceptance-test**

~~~~~~~~
$ ember g acceptance-test friends/new
installing
  create tests/acceptance/friends/new-test.js
~~~~~~~~

If we visit the generated test we'll see the following:

{title="tests/acceptance/friends/new-test.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';
import startApp from 'borrowers/helpers/start-app';

var App;

module('Acceptance: FriendsNew', {
  setup: function() {
    App = startApp();
  },
  teardown: function() {
    Ember.run(App, 'destroy');
  }
});

test('visiting /friends/new', function() {
  visit('/friends/new');

  andThen(function() {
    equal(currentPath(), 'friends/new');
  });
});
~~~~~~~~

We need to replace `import startApp from '../helpers/start-app';` with
`import startApp from '../../helpers/start-app';` and then make the
assertion of `currentPath` to look for `friends.new` instead of
`friends/new`.

We can then run our tests visiting
[http://localhost:4200/tests](http://localhost:4200/tests) or if we
want to run only the acceptance tests for Friends New
[http://localhost:4200/tests?module=Acceptance%3A%20FriendsNew](http://localhost:4200/tests?module=Acceptance%3A%20FriendsNew)

Let's add two more test but this time starting from the index URL, we
want to validate that we can navigate to new and then check that it
redirects to the correct place after creating a new user.

{title="Tests new friend: tests/acceptance/friends/new-test.js", lang="JavaScript"}
~~~~~~~~
test('Creating a new friend', function() {
  visit('/');
  click('a[href="/friends/new"]');
  andThen(function() {
    equal(currentPath(), 'friends.new');
  });
  fillIn('input[placeholder="First Name"]', 'Johnny');
  fillIn('input[placeholder="Last Name"]', 'Cash');
  fillIn('input[placeholder="email"]', 'j@cash.com');
  fillIn('input[placeholder="twitter"]', 'jcash');
  click('input[value="Save"]');

  //
  // Clicking save will fire an async event
  // We can use andThen which will called once the promises above
  // have been resolved
  //

  andThen(function() {
    equal(
      currentRouteName(),
      'friends.show.index',
      'Redirects to friends.show after create'
    );
  });

});
~~~~~~~~

The second test we want to add is that the application stays on the
new page if we click save without adding any field and that an error
message is displayed:

{title="Tests new friend: tests/acceptance/friends/new-test.js", lang="JavaScript"}
~~~~~~~~
test('Clicking save without filling fields', function() {
  visit('/friends/new');
  click('input[value="Save"]');
  andThen(function() {
    equal(
      currentRouteName(),
      'friends.new',
      'Stays on new page'
    );
    equal(
      find("h2:contains(You have to fill all the fields)").length,
      1,
      "Displays error message"
    );
  });

});
~~~~~~~~

### Mocking the API response

On the previous tests we are hitting the API but this is not a common
scenario, normally we'd like to mock the interactions with the API, to
do so we have different alternatives, one is to use
[Pretender](https://github.com/trek/pretender), a library which allows
us to mock request with a simple DSL.

Other alternative is to use the
[built-in mock generator](http://www.ember-cli.com/#mocks-and-fixtures)
in **ember-cli**, it basically takes advantage of the Express server
used for development and extend it to capture request to our API
end-points, with that we can control what do we want to return for
each request.

As an example let's create a mock for `api/articles`:

~~~~~~~~
$ ember g http-mock articles
installing
  create server/.jshintrc
  create server/index.js
  create server/mocks/articles.js
  install package connect-restreamer
~~~~~~~~

If open the generated file `server/mocks/articles.js`, we'll see the
following:

{title="server/mocks/articles.js", lang="JavaScript"}
~~~~~~~~
module.exports = function(app) {
  var express = require('express');
  var articlesRouter = express.Router();
  articlesRouter.get('/', function(req, res) {
    res.send({"articles":[]});
  });
  app.use('/api/articles', articlesRouter);
};

~~~~~~~~~

It intercepts the calls to any request starting with `/api/articles`
and then if it is a GET to `/` it will return `{"articles":[]}`.

Suppose we want to mock the request for a particular article, we can
add the following:

{title="server/mocks/articles.js", lang="JavaScript"}
~~~~~~~~
module.exports = function(app) {
  var express = require('express');
  var articlesRouter = express.Router();
  articlesRouter.get('/', function(req, res) {
    res.send({"articles":[]});
  });
  articlesRouter.get('/articles/74', function(req, res) {
    res.send({
      "article":{
        "id":74,
        "created_at":"2014-11-03T21:30:47.869Z",
        "description":"foo",
        "state":"borrowed",
        "notes":"bar",
        "friend_id":153
      }
    });
  });
  app.use('/api/articles', articlesRouter);
};
~~~~~~~~

it will intercept any GET request to `/articles/74` and return the
mocked article.

## Further Reading

During EmberConf 2014 [Eric Berry](https://twitter.com/coderberry)
gave a great talk called
[The Unofficial, Official Ember Testing Guide](http://www.confreaks.com/videos/3310-emberconf2014-the-unofficial-official-ember-testing-guide)
where he walk us through testing in Ember.js, Eric also contributed an
excellent guide for testing which is now the official in the Ember.js
website, we recommend the official guide since they do a complete
overview from unit to acceptance  testing:
[http://emberjs.com/guides/testing/](http://emberjs.com/guides/testing).

To know more about using mocks and fixtures we recommend the following
presentation:
[Real World Fixtures](https://speakerdeck.com/cball/real-world-fixtures)
by [Chris Ball](https://twitter.com/cball_).
