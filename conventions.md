# Conventions
We will explore some the basic conventions and best practices both in `Ember.js` and `ember-cli`.

## In your code
- **Use `camelCase` even if you are writing `CoffeeScript`.**
- **Avoid globals as much as possible:** `ember-cli` supports `ES6 Modules` out of the box so you can write your app in a modular way.
- **Create custom `shims` for apps which are not distributed in `AMD` format:** we will cover this in other chapter but the basic idea is to treat libraries which are not written with `ES6 Modules` as if they were.
- **Create reusable code:** if there is a functionality you are using in several different places remember that `Ember.js` offers an [Ember.Mixin class](http://emberjs.com/api/classes/Ember.Mixin.html) which you can the reuse in different parts. Do you think other people can benefit from this? Create an add-on.

## In your project
- **Name your files using kebab-case:** Use hyphens instead of underscores for separating your file name, so if you have a model called `InvoiceItem`, `ember-cli` expects this model to be under `app/models/invoice-item.js`.
- **Optionally include the file type at the beginning:** Some people like to include the type of the file they are writing on its name e.g. `app/routes/route-index.js`, I personally prefer not to do it but if you want to, it's fine, just remember to include it at the beginning otherwise your app will not be able to find in this case the `IndexRoute` .
- **Put child files in subdirectories:** `app/routes/invoice-item/index.js` or
`app/controllers/invoice-items/index.js`.
