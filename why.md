# Why
Before getting into the specifics I'd like to explain why ember-cli
was created and how it is different to other tools.

The main objective of ember-cli is to reduce what we call glue code
and allowing developers to focus in what is most important for them:
building their app.

Glue code are all those things which are not related to your
application but that you require to have in every project, for
example, you need to test your code, you have to compile your assets,
you need to somehow serve your files in the browser, you need to
interact with a back-end API, you might have to use third party
libraries, and so on. All those things can be automated and as it is
done in other frameworks, some conventions can be used to give you a
common ground to start building your applications.

Having a tool that does that for you is not just easing the process to
start writing your app but is also saving you time and money (you
don't have to think about problems which are already solved).

`ember-cli` aims to be exactly that tool, it tries to reduce the time
you have to wait while your assets compile using
`Broccoli`[^broccoli], it has built-in support to start writing your
tests with `QUnit`[^qunit] and then run them with `Testem`[^testem], if
you need to deploy you build with production environment then you get
fingerprint, compression, and some other features for free.

It also encourages you to write ES6(ECMAScript 6)[^ES6], having
built-in support for modules, integrating easily with other plugins
allowing you to write your applications using other ES6 features.

Next time you are considering to waste your time wiring up all those
things that I just mentioned, consider `ember-cli`, it will make your
life easier and you will get support from a lot of smart people who
is also using this tool.

[^ES6]: [https://people.mozilla.org/~jorendorff/es6-draft.html](https://people.mozilla.org/~jorendorff/es6-draft.html)
[^broccoli]: [https://github.com/broccolijs/broccoli](https://github.com/broccolijs/broccoli)
[^testem]: [https://github.com/airportyh/testem](https://github.com/airportyh/testem)
[^qunit]: [http://qunitjs.com/](http://qunitjs.com/)
