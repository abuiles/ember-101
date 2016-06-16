# Testing Ember.js applications

In this chapter we'll cover the basics of unit and acceptance testing in
Ember.js applications and recommend a couple of resources that can
help us expand our knowledge in this area.

## Unit Testing

When we run the generators, they create unit test files by default.
We can view all the generated unit tests if we go to `tests/unit`:

{title="Unit tests", lang="bash"}
~~~~~~~~
$ ls tests/unit/
adapters	controllers	models		utils
components	helpers		routes
~~~~~~~~

Tests are automatically grouped by type. If we open the unit test for our
friend model, we'll see the following:

{title="tests/unit/models/friend-test.js", lang="JavaScript"}
~~~~~~~~
import { test, moduleForModel } from 'ember-qunit';

moduleForModel('friend', 'Friend', { needs: ['model:article'] });

test('it exists', function(assert) {
  var model = this.subject();
  assert.ok(model);
});
~~~~~~~~

At the beginning of the test we import a set of helpers from
[ember-qunit](https://github.com/rwjblue/ember-qunit), which is a
library that wraps a bunch of functions to facilitate testing with
**QUnit**.

`moduleForModel` received the name of the model we are testing, a
description, and some options. In our scenario, we specify
that the tests need a model called **article** because of the existing
relationship between them.

Next, the test includes a basic assertion that the model exists.
`this.subject()` would be an instance of a `friend`.

We have two ways of running tests. The first one is via the browser while
we run the development server. We can navigate to
[http://localhost:4200/tests](http://localhost:4200/tests) and our
tests will be run. The second method is using a tests runner. At the
moment **ember-cli** has built-in support for **Testem** with
[PhantomJS](http://phantomjs.org/), which we can use to run our tests on
a CI server. To run tests in this mode, we only need to do `ember
test`.

I> We can also run tests with the command `npm test` which is aliased
I> to `ember test` in `package.json`.

Let's write two more tests for our friend model. We want to check that
the computed property `fullName` behaves as expected and that the
relationship articles is properly set.

{title="tests/unit/models/friend-test.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';
import { moduleForModel, test } from 'ember-qunit';

moduleForModel('friend', 'Unit | Model | friend', {
  // Specify the other units that are required for this test.
  needs: ['model:loan']
});

test('it exists', function(assert) {
  let model = this.subject();
  // let store = this.store();
  assert.ok(!!model);
});

test('fullName joins first and last name', function(assert) {
  var model = this.subject({firstName: 'Syd', lastName: 'Barrett'});

  assert.equal(model.get('fullName'), 'Syd Barrett');

  Ember.run(function() {
    model.set('firstName', 'Geddy');
  });

  assert.equal(model.get('fullName'), 'Geddy Barrett', 'Updates fullName');
});

test('loans relationship', function(assert) {
  var klass  = this.subject({}).constructor;

  var relationship = Ember.get(klass, 'relationshipsByName').get('loans');

  assert.equal(relationship.key, 'loans');
  assert.equal(relationship.kind, 'hasMany');
});
~~~~~~~~

We can run our tests by going directly to the following URL:
[http://localhost:4200/tests?module=Friend](http://localhost:4200/tests?module=Friend).

The first test verifies that `fullName` is calculated
correctly. We have to wrap `model.set('firstName', 'Geddy');` in
`Ember.run` because it has an asynchronous behavior. If we modify the
implementation for `fullName` such that it doesn't return first and last
names, the tests will fail.

The second test checks that we have set up the proper
relationship to `articles`. Something similar could go in the articles
model tests. If we call `constructor` on an instance to a model, that
will give us access to the class of which it is an instance.


Let's add other unit test for `app/utils/date-helpers`:

{title="tests/unit/utils/date-helpers-test.js", lang="JavaScript"}
~~~~~~~~
import dateHelpers from 'borrowers/utils/date-helpers';
import { module, test } from 'qunit';

module('Unit | Utility | date helpers');

test('formats a date object', function(assert) {
  var date = new Date("11-3-2014");
  var result = dateHelpers.formatDate(date, 'ddd MMM DD YYYY');

  assert.equal(result, 'Mon Nov 03 2014', 'returns a readable string');
});
~~~~~~~~

We import the function we want to test and then check that
it returns the date as a readable string. We can run the test by going to
[http://localhost:4200/tests?module=Unit%20%7C%20Utility%20%7C%20date%20helpers](http://localhost:4200/tests?module=Unit%20%7C%20Utility%20%7C%20date%20helpers).

## Acceptance Tests

With acceptance tests we can verify workflows in our application. For
example, making sure that we can add a new friend, that if we visit the
friend index a list is rendered, etc. An acceptance test basically emulates a
real user's experience of our application.


Ember has a set of helpers to simplify writing these kinds of
tests. There are
[synchronous](http://emberjs.com/guides/testing/test-helpers/#toc_wait-helpers)
and
[asynchronous](http://emberjs.com/guides/testing/test-helpers/#toc_asynchronous-helpers)
helpers. We use the former for tests that don't have any kind of
side-effect, such as checking if an element is present on a page, and the
latter for tests that fire some kind of side-effect. For example,
clicking a link or saving a model.

Let's write an acceptance test to verify that we can add new friends
to our application. We can generate an acceptance test with the
generator **acceptance-test**.

~~~~~~~~
$ ember g acceptance-test friends/new
installing
  create tests/acceptance/friends/new-test.js
~~~~~~~~

If we visit the generated test, we'll see the following:

{title="tests/acceptance/friends/new-test.js", lang="JavaScript"}
~~~~~~~~
import { test } from 'qunit';
import moduleForAcceptance from 'borrowers/tests/helpers/module-for-acceptance';

moduleForAcceptance('Acceptance | friends/new');

test('visiting /friends/new', function(assert) {
  visit('/friends/new');

  andThen(function() {
    assert.equal(currentURL(), '/friends/new');
  });
});
~~~~~~~~

Now we can run our tests by visiting
[http://localhost:4200/tests](http://localhost:4200/tests) or, if we
want to run only the acceptance tests for Friends New,
[http://localhost:4200/tests?module=Acceptance%20%7C%20friends%2Fnew](http://localhost:4200/tests?module=Acceptance%20%7C%20friends%2Fnew).

Let's add two more tests but this time starting from the index URL. We
want to validate that we can navigate to new and then check that it
redirects to the correct place after creating a new user.

{title="Tests new friend: tests/acceptance/friends/new-test.js", lang="JavaScript"}
~~~~~~~~
test('Creating a new friend', function(assert) {
  visit('/');
  click('a[href="/friends/new"]');
  andThen(function() {
    assert.equal(currentPath(), 'friends.new');
  });
  fillIn('input[placeholder="First Name"]', 'Johnny');
  fillIn('input[placeholder="Last Name"]', 'Cash');
  fillIn('input[placeholder="email"]', 'j@cash.com');
  fillIn('input[placeholder="twitter"]', 'jcash');
  click('input[value="Save"]');

  //
  // Clicking save will fire an async event.
  // We can use andThen, which will be called once the promises above
  // have been resolved.
  //

  andThen(function() {
    assert.equal(
      currentRouteName(),
      'friends.show.index',
      'Redirects to friends.show after create'
    );
  });

});
~~~~~~~~

The second test we want to add checks that the application stays on
the new page if we click save, without adding any fields, and that an
error message is displayed:

{title="Tests new friend: tests/acceptance/friends/new-test.js", lang="JavaScript"}
~~~~~~~~
test('Clicking save without filling fields', function(assert) {
  visit('/friends/new');
  click('input[value="Save"]');
  andThen(function() {
    assert.equal(
      currentRouteName(),
      'friends.new',
      'Stays on new page'
    );
    assert.equal(
      find("h2:contains(You have to fill all the fields)").length,
      1,
      "Displays error message"
    );
  });

});
~~~~~~~~

### Mocking the API response

On the previous tests we hit the API, but this is not a common
scenario. Normally we'd like to mock the interactions with the API.
To do so the community recommendation is to use
[ember-cli-mirage](http://www.ember-cli-mirage.com/), a library that
allows us to mock servers with a simple DSL.

Let's stop the server and install mirage running `ember install
ember-cli-mirage`.

Next, let's disable `mirage` if the environment is development, since
we only want to use it for testing.

{line-numbers=off, title="config/environment.js", lang=""}
~~~~~~~~
if (environment === 'development') {
  ENV['ember-cli-mirage'] = {
    enabled: false
  };
}
~~~~~~~~


If we go to http://localhost:4200/tests we'll see an error in the
acceptance test like:

{line-numbers=off, title="", lang="text"}
~~~~~~~~
Error: Mirage: Your Ember app tried to GET '/friends',
    but there was no route defined to handle this request.
    Define a route that matches this path in your
    mirage/config.js file. Did you forget to add your namespace?@ 269 ms
~~~~~~~~

That's because we didn't add the end-point friends to our mirage
server.

Let's tell `mirage` about the routes for friends, adding the following
lines to the file `mirage/config.js`:

{line-numbers=off, title="mirage/config.js", lang="javascript"}
~~~~~~~~
this.get('/friends');
this.post('/friends');
this.get('/friends/:id');
this.patch('/friends/:id');
this.delete('/friends/:id')
~~~~~~~~

After refreshing the tests page, we'll see a new error like the following:

```
Error: Pretender intercepted GET /friends but encountered an error:
Mirage: The route handler for /friends is trying to access the friend
model, but that model doesn't exist. Create it using 'ember g
mirage-model friend'.
```

Mirage is telling us that there is a handler for `friends` but it doesn't know about that model, we need to create the model in mirage with the command `ember g mirage-model friend`.

If we refresh again, the acceptance tests will pass. Mirage also help
us if we want to create fake data in the server. To do so, we need to
create a factory and then fill the server with fake data before
running the test. Let's explore that next.

We can create factories with the `mirage-factory` generator, to create
one for friends we can run the following command `ember g
mirage-factory friend` and then let's edit the factory to declare the
attributes:

{line-numbers=off, title="", lang=""}
~~~~~~~~
//
// Mirage ships with Fajer.js which help us to create fake data
//
// See https://github.com/marak/Faker.js/
//
import { Factory, faker } from 'ember-cli-mirage';

export default Factory.extend({
  firstName() {
    return faker.name.firstName();
  },
  lastName() {
    return faker.name.lastName();
  },
  email() {
    return faker.internet.email();
  },
  twitter: '@someone'
});
~~~~~~~~

Once we have defined the factory, we can use it in our acceptance
test. Let's create one for the friends index route and then assert
that the all the friend are rendered.

First, let's create the acceptance test running `ember g acceptance-test friends`

{line-numbers=off, title="tests/acceptance/friends-test.js", lang=""}
~~~~~~~~
import { test } from 'qunit';
import moduleForAcceptance from 'borrowers/tests/helpers/module-for-acceptance';

moduleForAcceptance('Acceptance | friends', {
  beforeEach() {
    //
    // Mirage makes the variable server avaialble in our tests.
    //
    // We can use server.create('model-name') to create 1 entry in the
    // mock server or use createList to create many.
    //
    //
    server.createList('friend', '10');
  }
});

test('visiting /friends', function(assert) {
  visit('/friends');

  andThen(function() {
    assert.equal(currentURL(), '/friends');

    //
    // This will fail since we are creating 10 friends, fix it :)
    //
    assert.equal(
      find('table tbody tr').length,
      9,
      'assertion');

  });
});
~~~~~~~~

With this we can have some idea of what mirage allow us to do, we can
work with relationships and use mirage for development, but that's out
of the scope of this book, for more information check out the
documentation
[http://www.ember-cli-mirage.com/docs/v0.2.x/](http://www.ember-cli-mirage.com/docs/v0.2.x/).

## Further Reading

During EmberConf 2014, [Eric Berry](https://twitter.com/coderberry)
gave a great talk called
[The Unofficial, Official Ember Testing Guide](http://www.confreaks.com/videos/3310-emberconf2014-the-unofficial-official-ember-testing-guide)
where he walked us through testing in Ember.js. Eric also contributed an
excellent guide for testing that is now the official guide on the Ember.js
website. We recommend the official guide, which provides a complete
overview from unit to acceptance testing:
[http://emberjs.com/guides/testing/](http://emberjs.com/guides/testing).

To know more about using mocks and fixtures, we recommend the following
presentation:
[Real World Fixtures](https://speakerdeck.com/cball/real-world-fixtures)
by [Chris Ball](https://twitter.com/cball_).
