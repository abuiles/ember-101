# Updating your project to the latest version of ember-cli

**ember-cli** is a project that is still moving quickly, so from time to
time we'll need to update our applications to use the latest version.

By the time this chapter was written the application was using version
**0.0.46**, which was one of the last releases before moving to
**0.1.X**. Now we want to move to the newest version in npm.

The following steps are the same that come listed with every release of
ember-cli:

1. We get rid of the current installed version: **npm uninstall -g ember-cli**
2. A lot of libraries that ember-cli relied on were updated as well.
We want to make sure we are getting the latest versions from npm
and not using the ones we had previously installed. To do that, we
clean the npm cache with: **npm cache clean**
3. Now we'll do the same with bower: **bower cache clean**
4. Finally, we'll install ember-cli again: **npm install -g ember-cli**

Once we have installed the latest version of **ember-cli**, we need to
update our project. Let's run the following commands in the borrowers
app directory:

1. Remove installed libraries, dist files, and temporary files: rm -rf node_modules bower_components dist tmp
2. Next we need to update **ember-cli**'s version in **package.json** running: npm install --save-dev ember-cli
3. Install dependencies: **npm install && bower install**

We are almost done. We have upgraded **ember-cli** and
**dependencies** successfully, but we still need to upgrade some files
in our projects. The good news is that we don't have to do it all
manually. We can use the command **ember init**. When we run this it will
try to make some changes in some of our existing files, and we
can answer these requests with any of the following options:

{title="Updating ember-cli", lang=bash}
~~~~~~~~
  y) Yes, overwrite
  n) No, skip
  d) Diff
  h) Help, list all options
~~~~~~~~

A good approach is to first inspect what changed with the option **d**
and then decide if we want to accept the change or not.

Let's run **ember init** and see the output. We'll also include some
comments that are not part of the original output just to clarify:


{title="", lang=bash}
~~~~~~~~
$ ember init .
version: 0.1.1
installing

#
# ember-cli will try to replace some files with their blueprint version
# we'll respond with d to see the diff
#

[?] Overwrite /borrowers/Brocfile.js? (Yndh) d

--- /borrowers/Brocfile.js
+++ /borrowers/Brocfile.js
@@ -3,28 +3,18 @@
 var EmberApp = require('ember-cli/lib/broccoli/ember-app');

 var app = new EmberApp();

+// Use `app.import` to add additional libraries to the generated
+// output files.
+//
+// If you need to use different assets in different
+// environments, specify an object as the first parameter. That
+// object's keys should be the environment name and the values
+// should be the asset to use in that environment.
+//
+// If the library that you are including contains AMD or ES6
+// modules that you would like to import into your application,
+// please specify an object with the list of modules as keys
+// along with the exports of each module as its value.
-app.import('vendor/fontello/fontello.css');
-app.import('vendor/fontello/font/fontello.ttf', {
-  destDir: 'font'
-});
-app.import('vendor/fontello/font/fontello.eot', {
-  destDir: 'font'
-});
-app.import('vendor/fontello/font/fontello.svg', {
-  destDir: 'font'
-});
-app.import('vendor/fontello/font/fontello.woff', {
-  destDir: 'font'
-});
-app.import('bower_components/picnic/releases/v2.min.css');
-app.import('bower_components/moment/moment.js');
-app.import('bower_components/borrowers-dates/index.js', {
-  exports: {
-    'borrowers-dates': [
-      'format'
-    ]
-  }
-});

 module.exports = app.toTree();

[?] Overwrite /borrowers/Brocfile.js? n
~~~~~~~~

In the diff above, the line with a plus (+) sign is what will
get added and the lines with the minus (-) will be removed.

In this specific scenario, nothing important got added to the
Brocfile and it is trying to remove our imports. We can simply ignore
this file with the option **n**.


Next it asks us if we want to overwrite the README. This file is
only a blueprint, so we can ignore it safely by hitting the key **n**.


{title="", lang=bash}
~~~~~~~~

[?] Overwrite /borrowers/README.md? (Yndh)
~~~~~~~~


The next file is **index.html**. This file can change from time to
time, so we should keep an eye on it. Let's see the diff with **d**:


{title="", lang="bash"}
~~~~~~~~
[?] Overwrite /borrowers/app/index.html? d

--- /borrowers/app/index.html
+++ /borrowers/app/index.html
@@ -6,20 +6,14 @@
     <title>Borrowers</title>
     <meta name="description" content="">
     <meta name="viewport" content="width=device-width, initial-scale=1">

+    {{content-for 'head'}}
-    {{BASE_TAG}}

     <link rel="stylesheet" href="assets/vendor.css">
     <link rel="stylesheet" href="assets/borrowers.css">
   </head>
   <body>
-    <script type="text/javascript">
-      window.EmberENV = {{EMBER_ENV}};
-    </script>
     <script src="assets/vendor.js"></script>
     <script src="assets/borrowers.js"></script>
-    <script type="text/javascript">
-      window.Borrowers = require('borrowers/app')['default'].create({{APP_CONFIG}});
-    </script>
   </body>
 </html>


[?] Overwrite /borrowers/app/index.html? (Yndh) Y
~~~~~~~~

At the beginning, ember-cli used to include inline scripts for starting
the app and defining a global ENV variable. It has changed to
encourage users to write CSP-compliant applications.

CSP (Content Security Policy) is basically a mechanism to help us
write more secure applications. The following is a great write-up by
the HTML5 Rocks folks:
[An Introduction to Content Security Policy](http://www.html5rocks.com/en/tutorials/security/content-security-policy/).


Next we have **router.js** and **application.hbs**. We won't include the
output for the diffs for the sake of brevity, but the first one doesn't change
very often and the latter one can be ignored since we don't want
any changes in our application template.

Unless we are in a version lower than **0.0.46**, we can safely ignore both.

{title="" lang="bash"}
~~~~~~~~
[?] Overwrite /borrowers/app/router.js? (Yndh) n
[?] Overwrite /borrowers/app/templates/application.hbs? (Yndh) n
~~~~~~~~

Next it will ask us if we want to overwrite bower.json. This and
package.json will probably change often since dependencies get
updated frequently. Here is where our revision control system plus a
bit of strategy comes in really handy. Let's inspect the diff with **d**:


{title="" lang="bash"}
~~~~~~~~
[?] Overwrite /borrowers/bower.json? (Yndh) d

--- /borrowers/bower.json
+++ /borrowers/bower.json
@@ -2,19 +2,16 @@
   "name": "borrowers",
   "dependencies": {
     "handlebars": "~1.3.0",
     "query": "^1.11.1",
     "ember": "1.7.0",
     "ember-data": "1.0.0-beta.10",
     "ember-resolver": "~0.1.7",
+    "loader.js": "stefanpenner/loader.js#1.0.1",
-    "loader": "stefanpenner/loader.js#1.0.1",
     "ember-cli-shims": "stefanpenner/ember-cli-shims#0.0.3",
     "ember-cli-test-loader": "rwjblue/ember-cli-test-loader#0.0.4",
     "ember-load-initializers": "stefanpenner/ember-load-initializers#0.0.2",
     "ember-qunit": "0.1.8",
     "ember-qunit-notifications": "0.0.4",
+    "qunit": "~1.15.0"
-    "qunit": "~1.15.0",
-    "picnic": "https://github.com/picnicss/picnic.git",
-    "moment": "~2.8.3",
-    "borrowers-dates": "~0.0.1"
   }
 }

[?] Overwrite /borrowers/bower.json? (Yndh) Y
~~~~~~~~

We responded with yes to the previous command. In this particular case,
not many dependencies changed but we need the update anyways. We also
notice that the dependencies we added were deleted.

How do we deal with this in a scenario where there are a lot of
dependencies changed and the ones introduced by us get deleted?
Version control systems to the rescue!

A good strategy is to put all of our dependencies at the end of the
default libraries (after QUnit) and then simply overwrite the whole file
when updating.

If we are using Git, we can bring back that last hunk
that was deleted from our file; it can easily be done with any GUI
based tool. If you are an Emacs user you can use diff-hl-mode,
Sublime emacs-git-gutter, or Vim's vim-gitgutter.

Next is **environment.js**. We should check the changes here since it
could have breaking changes:

{title="", lang="bash"}
~~~~~~~~
[?] Overwrite /borrowers/config/environment.js? d

--- /borrowers/config/environment.js
+++ /borrowers/config/environment.js
@@ -20,9 +20,9 @@
   };

   if (environment === 'development') {
     // ENV.APP.LOG_RESOLVER = true;
+    ENV.APP.LOG_ACTIVE_GENERATION = true;
-    // ENV.APP.LOG_ACTIVE_GENERATION = true;
     // ENV.APP.LOG_TRANSITIONS = true;
     // ENV.APP.LOG_TRANSITIONS_INTERNAL = true;
     ENV.APP.LOG_VIEW_LOOKUPS = true;
     }
~~~~~~~~

As in previous scenarios, there are not many significant changes, so we can
simply ignore by responding with **n**.

Next is **package.json**. Most of the changes are packages being updated.
We'll say yes to this change. Again, use the same strategy mentioned
with **bower.json** by putting our own libraries at the end.

{title="", lang="bash"}
~~~~~~~~
[?] Overwrite /borrowers/package.json? (Yndh) d

--- /borrowers/package.json
+++ /borrowers/package.json
@@ -18,13 +18,14 @@
   "author": "",
   "license": "MIT",
   "devDependencies": {
     "body-parser": "^1.2.0",
+    "broccoli-asset-rev": "0.3.0",
-    "broccoli-asset-rev": "0.1.1",
     "broccoli-ember-hbs-template-compiler": "^1.6.1",
+    "ember-cli": "0.1.1",
+    "ember-cli-content-security-policy": "0.2.0",
-    "ember-cli": "^0.1.1",
     "ember-cli-ic-ajax": "0.1.1",
+    "ember-cli-inject-live-reload": "^1.2.2",
-    "ember-cli-inject-live-reload": "^1.0.2",
     "ember-cli-qunit": "0.1.0",
     "ember-data": "1.0.0-beta.10",
     "express": "^4.8.5",
     "glob": "^4.0.5"

~~~~~~~~

Next is **.jshintrc**. Ideally we shouldn't have a lot of things in
there, but we should be especially careful if we are whitelisting some
vars. In this case we'll accept the change because we don't have
anything custom.

{title="", lang="bash"}
~~~~~~~~
[?] Overwrite /borrowers/tests/.jshintrc? (Yndh)  Y

~~~~~~~~

Next we have test helper files, which we are going to accept because we
haven't edited any of those files.

{title="", lang="bash"}
~~~~~~~~
[?] Overwrite /borrowers/tests/helpers/start-app.js? y
[?] Overwrite /borrowers/tests/index.html? y
[?] Overwrite /borrowers/tests/test-helper.js? y
~~~~~~~~

We are almost done. Using the strategy we mentioned to bring
back different hunks in a file, we'll make sure **bower.json** has
the following packages after **QUnit**:

~~~~~~~~
    "picnic": "https://github.com/picnicss/picnic.git",
    "moment": "~2.8.3",
    "borrowers-dates": "~0.0.1"
~~~~~~~~

Now we are done. If we run **ember server --proxy
http://api.ember-cli-101.com**, the application should start without any
problems.
