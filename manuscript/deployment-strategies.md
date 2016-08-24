# Deploying Ember.js applications

In this chapter we'll explore different alternatives to deploy our
Ember.js applications. We'll talk about S3 and Pagefront based
deployments where our application is completely separated from our API.

## Deploying to S3

In order to host our application in S3, we'll need to change our
application adapter so it hits our CORS enabled API and then generate
a production build.

To consume the API without using **ember-cli**'s proxy feature, we
need to set the property
[host](https://guides.emberjs.com/v2.6.0/models/customizing-adapters/#toc_host-customization)
in the application adapter.

To do so, let's add a configuration property called `host` in
`config/environment.js` and then read it from there.

{title="Adding host to config/environment.js", lang="JavaScript"}
~~~~~~~~
/* jshint node: true */

module.exports = function(environment) {
  var ENV = {
    host: 'https://api.ember-101.com',
    // ...
~~~~~~~~

Now we can use it in the adapter. Let's create the application adapter
running `ember g adapter application` and then add the following:

{title="app/adapters/application.js", lang="JavaScript"}
~~~~~~~~
import JSONAPIAdapter from 'ember-data/adapters/json-api';
import config from '../config/environment';

export default JSONAPIAdapter.extend({
  host: config.host
});
~~~~~~~~

We also need to change `app/routes/index.js` to use the host:

{title="app/routes/index.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';
import config from '../config/environment';

export default Ember.Route.extend({
  //
  // Check the following for more information about services
  // https://guides.emberjs.com/v2.5.0/applications/services/
  //
  ajax: Ember.inject.service(),
  model()  {
    return this.get('ajax').request(`${config.host}/friends`).then(function(data){
      return {
        friendsCount: data.data.length
      };
    });
  }
});
~~~~~~~~

Now we can stop the server and run it again without the option
`--proxy`.

Next we need to generate the production build using the command `ember
build`.

When we run `ember server`, we always run a `build` and add
some extra stuff so that we can run our project in development, but we
don't need the same files in production.

When we do `ember build`, the output goes by default to the directory
`dist`. Let's check that:

{title="ember build", lang="bash"}
~~~~~~~~
borrowers $ ember build
ember build
Building
Built project successfully. Stored in "dist/".
~~~~~~~~

Inspecting the `dist` directory, we'll see the following contents:

~~~~~~~~
dist/
|-- assets
|-- |-- app.scss
|-- |-- borrowers.css
|-- |-- borrowers.js
|-- |-- borrowers.map
|-- |-- ember-basic-dropdown.scss
|-- |-- ember-power-select
|-- |-- |-- themes
|-- |-- |-- |-- bootstrap.scss
|-- |-- |-- variables.scss
|-- |-- ember-power-select.scss
|-- |-- failed.png
|-- |-- font
|-- |-- |-- fontello.eot
|-- |-- |-- fontello.svg
|-- |-- |-- fontello.ttf
|-- |-- |-- fontello.woff
|-- |-- |-- fontello.woff2
|-- |-- passed.png
|-- |-- test-loader.js
|-- |-- test-support.css
|-- |-- test-support.js
|-- |-- test-support.map
|-- |-- tests.js
|-- |-- tests.map
|-- |-- vendor.css
|-- |-- vendor.js
|-- |-- vendor.map
|-- crossdomain.xml
|-- index.html
|-- robots.txt
|-- testem.js
|-- tests
    |-- index.html
~~~~~~~~

I> Remember we can see the options for a command passing the option
I> `--help` like `ember build --help`.

Let's talk about the assets directory first. All our JavaScript and
stylesheet files will end in this directory. We can also put
other kinds of assets, such as images or fonts, under `public/assets` and
they will be merged into this directory. If we had the image
`public/assets/images/foo.png` we could reference it in our stylesheets like
`images/foo.png`.

What about those test files? They are used for testing and only
included in development or test environments. If we go to
[http://localhost:4200/tests](http://localhost:4200/tests) and inspect the network tab, we'll see
that those files are being used.

The tests directory is the entry point for running tests. `testem.js`
is used by default when we do `ember test`. It uses **Testem** to run
the test with **PhantomJS**.

If we run the build command but we specify production environment (e.g.,
`ember build --environment production`) we'll see a very different
output:

~~~~~~~~
dist/
|-- assets
|-- |-- app.scss
|-- |-- borrowers-0434066470763b4f6d79622e2bb60ffb.css
|-- |-- borrowers-b01655dbaa5eaca990dcb5f786c3a2b9.js
|-- |-- ember-basic-dropdown.scss
|-- |-- ember-power-select
|-- |-- |-- themes
|-- |-- |-- +-- bootstrap.scss
|-- |-- +-- variables.scss
|-- |-- ember-power-select.scss
|-- |-- font
|-- |-- |-- fontello.eot
|-- |-- |-- fontello.svg
|-- |-- |-- fontello.ttf
|-- |-- |-- fontello.woff
|-- |-- +-- fontello.woff2
|-- |-- vendor-6d05d317665a43d23c85eafd86e90654.css
|-- +-─ vendor-8422fb84936d9c467a1a18d2ad95d6c5.js
|-- crossdomain.xml
|-- index.html
+-─ robots.txt
~~~~~~~~

We have fewer files this time. Nothing related with testing is
included because that is only a development/tests concern. Our assets
files were fingerprinted and minified. If we open `dist/index.html`
we'll see that the references to them were updated as well:

~~~~~~~~
<link rel="stylesheet" href="assets/vendor-6d05d317665a43d23c85eafd86e90654.css">
<link rel="stylesheet" href="assets/borrowers-0434066470763b4f6d79622e2bb60ffb.css">
~~~~~~~~

Fingerprinting is achieved using
[broccoli-asset-rev](https://github.com/rickharrison/broccoli-asset-rev).
This allows us the option to select the format of the files we want to
fingerprint and to append an URL to every asset.

Now we are ready to deploy to an S3 bucket. We need to create
the bucket and enable static website hosting. Let's set up an
index document, `index.html`.

The following guide explains how to set up your S3 bucket:
[Hosting a Static Website on Amazon S3](http://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html)

Once the bucket is set up, we can run `ember build --environment
production` and then manually upload all the files under `dist`. The
following is an example of the site working on S3:
[http://ember-101.s3-website-us-east-1.amazonaws.com/](http://ember-101.s3-website-us-east-1.amazonaws.com/)

It is very important that we set our bucket as public. To do this, we
can use the following bucket policy:

{title="S3 policy", lang="JavaScript"}
~~~~~~~~
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "AddPerm",
			"Effect": "Allow",
			"Principal": "*",
			"Action": "s3:GetObject",
			"Resource": "arn:aws:s3:::REPLACE-WITH-REAL-BUCKET-NAME/*"
		}
	]
}
~~~~~~~~

The following tutorial explains how to achieve a setup using custom
routing and Cloudfront: [Hosting a Static Website on Amazon Web Services](http://docs.aws.amazon.com/gettingstarted/latest/swh/website-hosting-intro.html)

If we decide to use Cloudfront, we need to prepend the URL to our
assets. To do this, we simply pass the option in the Brocfile as follows:

{title="Brocfile.js", lang="JavaScript"}
~~~~~~~~
var app = new EmberApp({
  fingerprint: {
    prepend: 'https://d29sqib8gy.cloudfront.net/'
  },
});
~~~~~~~~

If we run `ember build --environment production` and open
`dist/index.html`, we'll notice the URL in our assets.

~~~~~~~~
<script src="https://d29sqib8gy.cloudfront.net/assets/vendor-b29ae2f2e402c33a5d9c683aac4e0f8e.js"></script>
<script src="https://d29sqib8gy.cloudfront.net/assets/borrowers-c459411ce1cc8332ef795be81d96d1b6.js"></script>
~~~~~~~~

A better approach for deployments would be to use the ember addon
[ember-cli-deploy](http://ember-cli-deploy.com). Their website has a
complete guide on how to use their addon.

## Deploying to Pagefront

[Pagefront](https://www.pagefronthq.com/) is a PaaS for ember. This is
probably the easiest way to deploy ember applications today. They have
an ember addon which makes the whole process super simple.

First let's install the addon with `ember install`, we'll use
`ember-101-GITHUB-HANDLE` to identiy the application:

~~~~~~~~
ember install ember-pagefront --app=ember-101-abuiles
~~~~~~~~

After installing **ember-pagefront**, a new file `config/deploy.js`
will be created with the required data to deploy to pagefront.

Next we can run `ember deploy production` and that's it. Our
application will be deployed to
[https://ember-101-abuiles.pagefrontapp.com](https://ember-101-abuiles.pagefrontapp.com)


## ember-cli-deploy

[Ember CLI deploy](http://ember-cli-deploy.com/) has become the
default solution to manage deployments with ember. Pagefront use it
under the hood, and there are many other plugins available, which
allow us to deploy to different platforms or just develop our own
solutions.

We can find all the available options visiting the plugins section in
their website [http://ember-cli-deploy.com/docs/v0.6.x/plugins](http://ember-cli-deploy.com/docs/v0.6.x/plugins).
