# Deploying Ember.js applications

In this chapter we'll explore different alternatives to deploy our
Ember.js applications. We'll talk about S3 and Divshot based
deployments where our application is completely separated from our API.
Then we'll cover how to do a deployment on Heroku using the
`heroku-buildpack-ember-cli`, which allows us to proxy requests to our API.
Finally, we'll talk about Redis based deployments in Ruby on Rails
and Node.js.

## Deploying to S3

In order to host our application in S3, we'll need to change our
application adapter so it hits our CORS enabled API and then generate
a production build.

To consume the API without using **ember-cli**'s proxy
feature, we need to set the property `host` in the application
adapter.

To do so, let's add a configuration property called `host` in
`config/environment.js` and then read it from there.

{title="Adding host to config/environment.js", lang="JavaScript"}
~~~~~~~~
/* jshint node: true */

module.exports = function(environment) {
  var ENV = {
    host: 'http://api.ember-cli-101.com',
    // ...
~~~~~~~~

Now we can use it in the application adapter as follows:

{title="app/adapters/application.js", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';
import config from '../config/environment';

export default DS.ActiveModelAdapter.extend({
  host: config.host,
  namespace: 'api/v4',
  coalesceFindRequests: true
});
~~~~~~~~

We also need to change `app/routes/index.js` to use the host:

{title="app/routes/index.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';
import request from 'ic-ajax';
import config from '../config/environment';

export default Ember.Route.extend({
  model: function()  {
    var host = config.host || '';

    return request(host + '/api/friends').then(function(data){
      return {
        friendsCount: data.friends.length
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
version: 0.1.5
Building...
Built project successfully. Stored in "dist/".
~~~~~~~~

Inspecting the `dist` directory, we'll see the following contents:

~~~~~~~~
|- assets
|-- |- borrowers.css
|-- |- borrowers.js
|-- |- failed.png
|-- |- passed.png
|-- |- test-loader.js
|-- |- test-support.css
|-- |- test-support.js
|-- |- vendor.css
|-- |- vendor.js
|- crossdomain.xml
|- font
|-- |- fontello.eot
|-- |- fontello.svg
|-- |- fontello.ttf
|-- |- fontello.woff
|- index.html
|- robots.txt
|- testem.js
|- tests
    |- index.html
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
.
|-- assets
|- |-- borrowers-97a85d25222a06c4a39d475c7ad27a73.js
|- |-- borrowers-985aabef341eea2a8b20d3e9e685d6b0.css
|- |-- images
|- |-- vendor-9877b53c34630081b26b7b9fd19d4bb8.css
|- |-- vendor-b29ae2f2e402c33a5d9c683aac4e0f8e.js
|-- crossdomain.xml
|-- font
|- |-- fontello.eot
|- |-- fontello.svg
|- |-- fontello.ttf
|- |-- fontello.woff
|-- index.html
|-- robots.txt
~~~~~~~~

We have fewer files this time. Nothing related with testing is
included because that is only a development/tests concern. Our assets
files were fingerprinted and minified. If we open `dist/index.html`
we'll see that the references to them were updated as well:

~~~~~~~~
<link rel="stylesheet" href="assets/vendor-9877b53c34630081b26b7b9fd19d4bb8.css">
<link rel="stylesheet" href="assets/borrowers-985aabef341eea2a8b20d3e9e685d6b0.css">
~~~~~~~~

Fingerprinting is achieved using
[broccoli-asset-rev](https://github.com/rickharrison/broccoli-asset-rev).
This allows us the option to select the format of the files we want to
fingerprint and to append an URL to every asset.

All our assets should ideally be kept under the directory `/assets`, so
let's make sure our fonts are put in there as well. To do this, we need to
modify our `Brocfile` and the references to the fonts in
`vendor/fontello/fontello.css`.

To accomplish the first part we simply need to specify `assets/fonts` as
the `destDir` for our imported fonts:

{title="Brocfile.js", lang="JavaScript"}
~~~~~~~~
app.import('vendor/fontello/font/fontello.ttf', {
  destDir: 'assets/fonts'
});
app.import('vendor/fontello/font/fontello.eot', {
  destDir: 'assets/fonts'
});
app.import('vendor/fontello/font/fontello.svg', {
  destDir: 'assets/fonts'
});
app.import('vendor/fontello/font/fontello.woff', {
  destDir: 'assets/fonts'
});
~~~~~~~~

If we run `ember build --environment production`, we'll find our
fonts under `assets/fonts`.

{title="Putting fonts under assets directory", lang="bash"}
~~~~~~~~
.
|-- assets
|-- |-- borrowers-985aabef341eea2a8b20d3e9e685d6b0.css
|-- |-- borrowers-da3abd96a2852e1cfa758c2d41b82a5e.js
|-- |-- fonts
|-- |-- |-- fontello.eot
|-- |-- |-- fontello.svg
|-- |-- |-- fontello.ttf
|-- |-- |-- fontello.woff
|-- |-- images
|-- |-- vendor-9877b53c34630081b26b7b9fd19d4bb8.css
|-- |-- vendor-b29ae2f2e402c33a5d9c683aac4e0f8e.js
|-- crossdomain.xml
|-- index.html
|-- robots.txt
~~~~~~~~

Next we need to replace `vendor/fontello/fontello.css` to reference
the fonts relative to `fonts/` instead of `../font`:

{title="vendor/fontello/fontello.css", lang="CSS"}
~~~~~~~~
@font-face {
  font-family: 'fontello';
  src: url('fonts/fontello.eot?59907090');
  src: url('fonts/fontello.eot?59907090#iefix') format('embedded-opentype'),
       url('fonts/fontello.woff?59907090') format('woff'),
       url('fonts/fontello.ttf?59907090') format('truetype'),
       url('fonts/fontello.svg?59907090#fontello') format('svg');
  font-weight: normal;
  font-style: normal;
}
~~~~~~~~

Now we are ready to deploy to an S3 bucket. We need to create
the bucket and enable static website hosting. Let's set up an
index document, `index.html`.

The following guide explains how to set up your S3 bucket:
[Hosting a Static Website on Amazon S3](http://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html)

Once the bucket is set up, we can run `ember build --environment
production` and then manually upload all the files under `dist`. The
following is an example of the site working on S3:
[http://ember-cli-101.s3-website-us-east-1.amazonaws.com/](http://ember-cli-101.s3-website-us-east-1.amazonaws.com/)

It is very important that we set our bucket as public. To do this, we can
use the following bucket policy:

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

A better approach to uploading our files to S3 is to create a task to do
this for us. At the moment there is no built-in support for this in
ember-cli, but we can use [Grunt](http://gruntjs.com/) with the
following plugin:
[grunt-aws-s3](https://github.com/MathieuLoutre/grunt-aws-s3).

## Deploying to Divshot

[Divshot](https://divshot.com/) is a PaaS for deploying static
websites. This is probably the easiest way to deploy such applications,
and [Robert Jackson](https://twitter.com/rwjblue) wrote an **ember-cli addon** to make it even easier to
deploy our ember-cli applications.

Before installing the addon, we have to first create an account with
them and then install their command line interface:

~~~~~~~~
npm install -g divshot-cli
~~~~~~~~

After installing **divshot-cli**, we need to login typing
`divshot login`.

Once we are logged in, we are ready to deploy our application. First
install the addon [ember-cli-divshot](https://github.com/rwjblue/ember-cli-divshot):

~~~~~~~~
npm install ember-cli-divshot --save-dev
~~~~~~~~

With the addon installed, we need to set up DivShot with `ember generate
divshot`, and after that we can deploy just running `ember divshot
push`.

{title="Deploying to DivShot", lang="bash"}
~~~~~~~~
$ ember divshot push
version: 0.1.5
Built project successfully. Stored in "dist".
Creating build ...
Hashing Directory Contents ...
Synced!
Finalizing build ...
Releasing build to development ...
Success: Application deployed to development
Success: You can view your app at: http://development.borrowers.divshot.io
~~~~~~~~

That's it! Our application has been deployed to
[http://development.borrowers.divshot.io](http://development.borrowers.divshot.io).

## Deploying to Heroku with the heroku-buildpack-ember-cli

Deploying to [Heroku](http://heroku.com/) is a simple process
thanks to [Tony Coconate](https://twitter.com/tonycoco)'s
[heroku-buildpack-ember-cli](https://github.com/tonycoco/heroku-buildpack-ember-cli).

Assuming we have already created an account on Heroku and installed
[heroku toolbelt](https://toolbelt.heroku.com/), we can now deploy
with the following steps.

First we need to create an application based on the buildpack:

~~~~~~~~
$ heroku create --buildpack https://github.com/tonycoco/heroku-buildpack-ember-cli.git
Creating polar-cove-8298... done, stack is cedar
BUILDPACK_URL=https://github.com/tonycoco/heroku-buildpack-ember-cli.git
https://polar-cove-8298.herokuapp.com/ | git@heroku.com:polar-cove-8298.git
Git remote heroku added
~~~~~~~~

Now we can deploy doing `git push heroku master`. We can see our
application running on Heroku: [http://polar-cove-8298.herokuapp.com/](http://polar-cove-8298.herokuapp.com/)

### Using the Proxy Feature.

Supposing we don't want to enable CORS in our API, the build-pack has
a **Proxy** feature that acts similarly to the one included with
**ember-cli**.

Using the following command, we can set up the URL to which we want to proxy our request:

~~~~~~~~
heroku config:set API_URL=http://api.ember-cli-101.com/
~~~~~~~~

We can find more info about this in the [Github repository](https://github.com/tonycoco/heroku-buildpack-ember-cli#api-proxy).

## ember-cli-deploy

During RailsConf 2014, [Luke Melia](https://twitter.com/lukemelia)
gave a talk called
[Lightning Fast Deployment of Your Rails-backed JavaScript app](https://www.youtube.com/watch?v=QZVYP3cPcWQ).

Luke presented a solution to keep the deployment of JavaScript
applications separate from the backend. The basic idea is to deploy our assets to a CDN and then pass the generated
`index.html` via Redis to the application serving it.

[Aaron Chambers](https://github.com/achambers) created an addon called
[ember-cli-deploy](https://github.com/achambers/ember-cli-deploy) that
makes it super easy to implement Luke's ideas.
