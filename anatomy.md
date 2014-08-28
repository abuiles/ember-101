# Anatomy
In this chapter we will learn about ember-cli main components.

`ember-cli` is a `Node.js` command line application sitting on top of other libraries.

It's main  component is `Broccoli`, which allows us to have fast builds, `Broccoli` is a builder designed with the goal of keeping builds as fast as possible [^broslow].

When we run `ember server`, `Broccoli` compiles our project and put it in a directory when it can be served using `Express.js`[^express] which is a `Node.js` library. `Express` is not only used to served files but also to extend `ember-cli` functionality using its `middlewares`, an example of this is the `http-proxy` which supports the `--proxy` option, allowing us to develop against our development backend.

Testing  is powered by `QUnit` and `Testem`, we can always navigate to `http:/localhost:4200/tests` and our test will be run automatically. We can also run Testem in `CI` or `--development` mode with the `ember test` command. Currently there is just support for `QUnit` through an `ember-cli add-on`, we will probably see support for other testing frameworks and runners as people get familiar with the add-on system.

`ember-cli` uses it's own resolver and has a different naming convention to `Ember.js's` defaults.

`ember-cli` makes us write our application using `ES6 Modules`, then the code gets transpiled (compiled)[^transpiled] to `AMD`[^amd] and finally it is loaded with `loader.js` which is a minimalist `AMD` loader.

If you want to use coffee script you can use it but it is encouraged to use plain JS and ES6 where possible, we'll explore on next chapters its syntax and features.

Finally we need to cover `Broccoli` plugins because without them, `Broccoli` wouldn't be as helpful. Every transformation that your files are going through, are done with a `Broccoli` plugin, e.g. transpiling, minifying, finger-printing, uglifying. You can have your own `Broccoli` plugins and plug it wherever you want it the build process.

[^broslow]: Currently there are some issues on Mac which slows down the build time but there is work in progress to fix it.
[^express]: [http://expressjs.com/](http://expressjs.com/)
[^transpiled]: The transpiring process is done with [es6-module-transpiler](https://github.com/esnext/es6-module-transpiler).
[^amd]: To know more about `AMD` checkout [their wiki](https://github.com/amdjs/amdjs-api/wiki/AMD)
