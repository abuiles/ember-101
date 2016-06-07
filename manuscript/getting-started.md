# Getting started

With this book, we'll create an app to keep track of items
we lend to our friends. It's a very simple app, but it will allow us to
learn Ember. At the same time, we'll learn how to use Ember CLI
generators, work with third party libraries, and write
**Ember CLI add-ons**.

## Requirements

1. Install `Node.js`. The easiest way is to download the installer from [http://nodejs.org/](http://nodejs.org/).
2. Install the ember-inspector. Click [here for Chrome](https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi?hl=en) or [here for Firefox](https://addons.mozilla.org/en-US/firefox/addon/ember-inspector/).
3. Install [watchman](https://github.com/facebook/watchman) for fast watching. We can start it with `watchman watch ~/path-to-dir`.
4. Make sure you are not required to run `npm` (Node's package manager) with sudo. To test this, run the following command

{title="", lang="bash"}
~~~~~~~~
npm -g install ember-cli
~~~~~~~~

If you were prompted to install as sudo, make sure you can run
`npm` without it. Tyler Wendlandt wrote an excellent tutorial for
installing `npm` without sudo:
[http://www.wenincode.com/installing-node-jsnpm-without-sudo](http://www.wenincode.com/installing-node-jsnpm-without-sudo).
It's very important that you are not required to run `npm` as sudo,
otherwise you will have problems when running Ember CLI.

All set? Now let's create our first Ember app.

## ember new

Like other command line tools, `Ember CLI` comes with a bunch of useful
commands. The first one we will explore is `new`, which creates a
new project.

{title="Creating a new project", lang="bash"}
~~~~~~~~
ember new borrowers
~~~~~~~~

The **new** command will create a directory with the following structure:


{title="Project Structure", lang="bash"}
~~~~~~~~
|-- README.md
|-- app
|-- bower.json
|-- bower_components
|-- config
|-- ember-cli-build.js
|-- node_modules
|-- package.json
|-- public
|-- testem.json
|-- tests
+-- vendor
~~~~~~~~

Change directory into your project.

T> We can add `--help` to any `ember` command to see available
T> options (e.g., `ember new --help`).

T> By default, Ember CLI assumes we are using git. If we are not, we
T> can opt out by passing **--skip-git**: `ember new borrowers --skip-git`.

We will cover all the components as we move through this text, but the
following are the most important.

- `app` is where the app code is located: models, routes, templates, components and styles.
- `tests` is where test code is located.
- `bower.json` helps us manage `JavaScript` plugins via `Bower`.
- `package.json` helps us with `JavaScript` dependencies via `npm`.

A> A question that pops up often is, "What's the difference between `npm` and `bower`?" From [this](http://stackoverflow.com/questions/21198977/difference-between-grunt-npm-and-bower-package-json-vs-bower-json/21199026#21199026) `Stack Overflow`: *Npm and Bower are both dependency management tools. The main difference between them is that npm is used to install Node js modules while bower js is used to manage front end components like html, css, js, etc.

If everything is fine, we can do `ember server` and navigate to `http://localhost:4200` where we should see a `Welcome to Ember` message.
