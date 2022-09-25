---
layout: post
title:  "Initial setup for Vue3 and Spatie Data"
date:   2022-09-25 15:00:00 -0400
categories: general
---
So I decided to try something new just to keep myself sharp. I decided to manually install inertia and configure it for use with vue3 and vite. I hit a few small snags but I'm going to document them all here to hopefully not run into the same issues again.

[Jump to Solution Already...](#tldr-use-glob-imports)

### Install Inertia

The initial installation was exactly the way the docs presented it. I started to run into some issues when I got to the client side installation.

This is what the docs show you for the client side installation.
```js
import { createApp, h } from 'vue'
import { createInertiaApp } from '@inertiajs/inertia-vue3'

createInertiaApp({
  resolve: name => require(`./Pages/${name}`),
  setup({ el, App, props, plugin }) {
    createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
})
```

The first thing you may notice is that vite does not allow you to use require directly. If you switch to import it would look something like this-

```js
resolve: name => import(`./Pages/${name}`),
```

Easy enough- but if you run this you will get a warning in the vite compiler

```
  createInertiaApp({
      resolve: name => import(`./Pages/${name}`),
                              ^
      setup({ el, App, props, plugin }) {
          createApp({ render: () => h(App, props) })
The above dynamic import cannot be analyzed by Vite.
```

First thing I tried was to add .vue to the end of the import. Something like this-
```js
resolve: name => import(`./Pages/${name}.vue`),
```
But you will find that vue files placed in subdirectories will not resolve. For instance, if I wanted to have a file in `Pages/books/Index.vue` then it would not be able to find this file.

So what is the solution? Globs...
### TLDR use Glob imports

First you need a function to parse out the name into the directory and file name. Something like this-

```js
/**
 * Imports the given page component from the page record.
 */
function resolvePageComponent(name, pages) {
  for (const path in pages) {
    if (path.endsWith(`${name.replace('.', '/')}.vue`)) {
      return typeof pages[path] === 'function'
        ? pages[path]()
        : pages[path]
    }
  }

  throw new Error(`Page not found: ${name}`)
}
```

Then you need to use this function to resolve your component names.
```js
resolve: (name) => resolvePageComponent(name, import.meta.glob('./Pages/**/*.vue')),
```

All together it will look like this-

```js
function resolvePageComponent(name, pages) {
  for (const path in pages) {
    if (path.endsWith(`${name.replace('.', '/')}.vue`)) {
      return typeof pages[path] === 'function'
        ? pages[path]()
        : pages[path]
    }
  }

  throw new Error(`Page not found: ${name}`)
}

createInertiaApp({
  resolve: (name) => resolvePageComponent(name, import.meta.glob('./Pages/**/*.vue')),
  setup({ el, App, props, plugin }) {
    createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
})
```

PS- Credit goes to [laravel-vite](https://laravel-vite.dev/guide/extra-topics/inertia.html#server-side-rendering ) for the answer to this problem!

### But wait there's more!

I also was attempting to get Laravel Data up and running so that I could use the Typescript transformer and import my types. (What can I say, I pay close attention to the Laracons...)

First of all there are two packages you will need in order to use the Typescript functionality which is something I wasn't aware of coming into it.
You will need
```
composer require spatie/laravel-data
&&
composer require spatie/typescript-transformer
```

After publishing your config for the typescript-transformer you will add the DataTypeScriptTransformer to the transformers in typescript-transformer.php like so-

```php
    'transformers' => [
      ...
      \Spatie\LaravelData\Support\TypeScriptTransformer\DataTypeScriptTransformer::class,
    ],
```

Now when you add the `#[TypeScript]` attribute to your Data object you will be able to generate your typescript files by running `php artisan typescript:transform`.

Finally and crucially, you will need to add this new type file to your vite.config.js file like so

```js
    compileroptions: {
        types: [
            ...
            "./resources/types/generated",
        ]
    }
```

Thats it! Now you should be able to use your type in your Vue3 files like so- `App.Data.Books.BooksData`
