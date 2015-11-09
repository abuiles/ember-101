# Conventions
We will explore some of the basic conventions and best practices both in `Ember` and `Ember CLI`.

- Use `camelCase` even if you are writing `CoffeeScript`.
- Avoid globals as much as possible: `Ember CLI` supports `ES6 Modules` out of the box so you can write your app in a modular way.
- Create custom `shims` for apps that are not distributed in `AMD` format: we will cover this in a subsequent chapter, but the basic idea is to treat libraries that are not written with `ES6 Modules` as if they were.
- Create reusable code: if there is a functionality you are using in several different places, remember that `Ember` offers an [Ember.Mixin class](http://emberjs.com/api/classes/Ember.Mixin.html) that you can then reuse in different parts. If you think other people can benefit from this, create an add-on.
- Name your files using kebab-case: Use hyphens instead of underscores to separate words in a file name. For example, if you have a model called `InvoiceItem`, `Ember CLI` expects this model to be under `app/models/invoice-item.js`.
