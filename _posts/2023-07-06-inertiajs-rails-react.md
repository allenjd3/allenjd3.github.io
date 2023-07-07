---
layout: post
title:  "How to install inertiajs with Rails 7 and react"
date:   2023-07-06 15:00:00 -0400
categories: general
---
Rails 7 introduced some new ways of interacting with javascript. It introduced importmaps which are a new way of pinning
your js files to your project. It also make turbo the default for frontend development which mostly got rid of the need
to have frontend javascript. One thing that I miss about rails 6 however was the quick and easy integration with tools
like inertiajs. This article goes into the new way to install it so I don't forget in the future.

Small disclaimer- most of the content of this article has been pieced together from the docs and other blogs. I didn't
find a current blog that had all of the pieces in the correct order however to end up with a working version of rails
and react. Thats what I hope to accomplish here.

1. Install Rails and skip dependencies

```
rails new exampleapp --skip-javascript --skip-asset-pipeline
```

2. Install inertia rails and vite rails

```
bundle add inertia_rails
bundle add vite_rails
```

3. Execute the vite installer

```
bundle exec vite install
```

4. Go to `/app/views/layouts/application.html.erb` and change the following-
1. Remove `<%#= stylesheet_link_tag "application" %>` (We will add the styles in our javascript)
2. Change `<%= vite_javascript_tag 'application' %>` to `<%= vite_javascript_tag 'application.jsx' %>`

5. Install vite-plugin-rails and @vitejs/plugin-react

```
yarn add -D vite-plugin-rails @vitejs/plugin-react react react-dom @inertiajs/react
```

Make sure your vite.config.ts looks like the following-

```ts
import {defineConfig} from 'vite'
import react from '@vitejs/plugin-react'
import ViteRails from 'vite-plugin-rails'

export default defineConfig({
  plugins: [
    react(),
    ViteRails(),
  ],
})
```

6. In `app/frontend` add two new folders- css and pages

In pages we can add our first test file- home.jsx with the following contents

```jsx
export default function home() {
  return (
    <div className="mt-16 mx-auto max-w-5xl">
      <h1 className="text-3xl">Hello home</h1>
    </div>
  );
}
```

7. In `app/frontend/entrypoints` you should already have a file called application.js change the name to application.jsx
   and add the following

```jsx
import {createInertiaApp} from '@inertiajs/react'
import {createRoot} from 'react-dom/client'
import '../css/styles.css' // We are going to add this in step 10

createInertiaApp({
  resolve: name => {
    const pages = import.meta.glob('../pages/**/*.jsx', {eager: true})
    return pages[`../pages/${name}.jsx`]
  },
  setup({el, App, props}) {
    createRoot(el).render(<App {...props} />)
  },
})
```

8. Generate a controller using rails's default generator

```
rails g controller home index
```

You can delete the `app/views/home/index.html.erb` file that is created

In the home_controller, change it as follows-

```ruby
class HomeController < ApplicationController
  def index
    render inertia: 'home'
  end
end
```
Then add your route to the routes file-

```ruby
Rails.application.routes.draw do
  root "home#index"
end
```

9. Now we can install tailwindcss-

```
yarn add -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

Configure your template paths in the generated `tailwind.config.js` as follows

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./app/views/**/*.html.erb",
    "./app/frontend/**/*.{js,ts,jsx,tsx,css}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

10. Add the styles.css file to `app/frontend/css` and paste the following

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

11. Start up your server and see if it works!

```
foreman start -f Procfile.dev
```

You should now have a working rails inertiajs react setup. My recommendation is to install rspec and follow along in the
docs to use the built in testing support. Enjoy!
