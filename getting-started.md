# Getting started

In this book will be creating a simple app to keep track of the things we lend to our friends, it's a very simple app but it will allow us to learn how to use ember-cli generators, how to work with third party libraries, write `ember-cli add-ons` and if you are getting started with `Ember.js`, then introduce you to some of `Ember.js` concepts.

## Requirements

1. Install `Node.js`, the easies way is to download the installer from [http://nodejs.org/](http://nodejs.org/)
2. Install the ember-inspector, click [here for Chrome](https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi?hl=en) or [here for Firefox](https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi?hl=en)
3. Make sure you are not required to run `npm` (Node's package manager) with sudo, to test this run the following command

```shell
npm -g install ember-cli
```
If you were prompt to install as sudo, then make sure you can run `npm` without it, Tyler Wendlandt wrote an excellent tutorial for installing `npm` without sudo [http://www.wenincode.com/installing-node-jsnpm-without-sudo](http://www.wenincode.com/installing-node-jsnpm-without-sudo). It's very important that you are not required to run `npm` as sudo otherwise you will have problems when running ember-cli.

All set? Now let's create our first ember-cli app.

## ember new
Like other command line tools `ember-cli` comes with a bunch of useful commands, the first one we will be exploring is `new` which creates a new project for us with the basic structure

```shell
ember new borrowers
```

The **new** command will create a directory with the following structure:


```shell
$ tree -L 1
.
├── Brocfile.js
├── README.md
├── app
├── bower.json
├── bower_components
├── config
├── dist
├── package.json
├── public
├── server
├── testem.json
├── tests
└── vendor
```

T> Add `--help` to any `ember` command if you want to see available
options; i.e. `ember new --help`

T> By default ember-cli assumes you are using git, if you are not, you
can opt-out passing **--skip-git**: **ember new borrowers --skip-git**.

I'm going to mention the most important parts here and the rest we will learn as we move forward.

- `app` is where the app code is located, controllers, routes, views, templates, and styles.
- `tests` is where tests code is located.
- `bower.json` helps us manage `JavaScript` plugins via `Bower`.
- `package.json` help us with `JavaScript` dependencies via `npm`.

A> What's the difference between `npm` and `bower` you might ask? From [this](http://stackoverflow.com/questions/21198977/difference-between-grunt-npm-and-bower-package-json-vs-bower-json/21199026#21199026) `Stack Overflow`: *Npm and Bower are both dependency management tools. But the main difference between both is npm is used for installing Node js modules but bower js is used for managing front end components like html,css,js etc.

If everything is fine you can do `ember server` and then navigate to `http://localhost:4200` you should see a `Welcome to Ember.js` message.
