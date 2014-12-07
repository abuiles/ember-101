# Anatomy
In this chapter we will learn about the main components of ember-cli.

**ember-cli** is a **Node.js** command line application that sits on top of
other libraries.

Its main component is **Broccoli**, a builder designed to keep builds as
fast as possible.

When we run `ember server`, **Broccoli** compiles our project and puts it
in a directory where it can be served using **Express.js**[^express], a **Node.js** library. **Express** not only serves
files but also extends **ember-cli**'s functionality using its
**middlewares**. An example of this is **http-proxy**, which supports
the `--proxy` option that allows us to develop against our development
backend.

Testing is powered by **QUnit** and **Testem**. By navigating to
**http:/localhost:4200/tests**, our tests run automatically.
We can also run Testem in **CI** or `--development` mode with the **ember
test** command. Currently, only **QUnit** is supported and it's done via an
**ember-cli add-on**. We will probably see support for other testing frameworks
and runners as more people become familiar with the add-on system.

**ember-cli** uses its own resolver and has a different naming
convention from **Ember.js's** defaults.

**ember-cli** makes us write our application using **ES6 Modules**. The
code is then transpiled (compiled)[^transpiled] to **AMD**[^amd] and
finally loaded with the minimalist **AMD**[^amd] loader, **loader.js**.

You can use **CoffeeScript** if you want, but you are encouraged to use plain JS
and ES6 modules where possible. In subsequent chapters, we'll explore its syntax and
features.

Finally, we need to cover plugins that enhance the functionality of **Broccoli**. Each transformation your
files go through is done with a **Broccoli** plugin, e.g.
transpiling, minifying, finger-printing, uglifying. You can have your
own **Broccoli** plugins and plug them wherever you like throughout the build
process.

[^express]: [http://expressjs.com/](http://expressjs.com/)
[^transpiled]: The transpiling process is done with [es6-module-transpiler](https://github.com/esnext/es6-module-transpiler).
[^amd]: To know more about **AMD** checkout [their wiki](https://github.com/amdjs/amdjs-api/wiki/AMD)
