# Why Ember?

Ember is an evolving JavaScript framework for creating "ambitious web
application", it tries to maximize developers' productivity using a set
of conventions in a way that they don't need to think about common
idioms when building web applications.

Using Ember in conjuntion with Ember CLI, allows you to reduce big
amounts of glue code, given you the opportunity to focus on what is
most important for you: building your application.

Glue code refers to those things that are not related to your
application but that every project requires. For
example, you need to test your code, compile your assets,
serve your files in the browser, interact with a back-end API,
perhaps use third party
libraries, and so on. All those things can be automated and, as it is
done in other frameworks, some conventions can provide a
common ground to begin building your applications.

Having a tool that does that for you not only eases the process of
writing your app but also saves you time and money (you
don't have to think about problems that are already solved).

`Ember CLI` aims to be exactly that tool. Thanks to `Broccoli`[^broccoli],
waiting time is reduced while your assets compile. `QUnit`[^qunit] allows
you to write tests, which can then be run with `Testem`[^testem].
If you need to deploy your build to production, you'll get
fingerprint, compression, and some other features for free.

`Ember CLI` also encourages the use of ES6(ECMAScript 6)[^ES6]. It
provides built-in support for modules and integrates easily with other
plugins, allowing you to write your applications using other ES6
features.

Next time you consider wasting your day wiring up all those things we
just mentioned, consider `Ember CLI`. It will make your life easier
and you will get support from a lot of smart people who are already
using this tool.

[^ES6]: [https://people.mozilla.org/~jorendorff/es6-draft.html](https://people.mozilla.org/~jorendorff/es6-draft.html)
[^broccoli]: [https://github.com/broccolijs/broccoli](https://github.com/broccolijs/broccoli)
[^testem]: [https://github.com/airportyh/testem](https://github.com/airportyh/testem)
[^qunit]: [http://qunitjs.com/](http://qunitjs.com/)
