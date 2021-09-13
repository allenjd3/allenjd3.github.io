---
layout: post
title:  "Thinking in React... in Livewire"
date:   2021-09-13 16:00:00 -0400
categories: general
---
So I've talked a little about Hotwire in the past but for Laravel there is a much more ubiquitous "wire" around- [Livewire](https://laravel-livewire.com/). Hotwire is very much a "build the way you always build and then sprinkle in the magic" tool. Livewire is more of a thought switch so I struggled with it at first. I think I've found the correct balance for it though and it comes from an article I read in the React Docs of all places: [Thinking in React](https://reactjs.org/docs/thinking-in-react.html)

Livewire is a bit of a mental switch. Its more react-like in the way it handles data. The craziest thing to me is that you don't need controllers anymore. All the business logic can be organized into components. You can even route to individual components. 

```php
Route::get('/post', ShowPosts::class);
```

The key to thinking livewire is to break your component into smaller components like in the 'thinking in react' article linked above. This helps to answer the questions-
1. What do I test?
2. Where do I query the data? 
3. Do I pass it as props or do I handle it in the component itself? 
4. Where do I change or act on the data?

Without answering these questions, you can get some really odd spaghetti code. You can be querying and modifying data in a sub component that shouldn't have that ability. It makes it harder to tell which components are mutating state and which are getting their own state. Start thinking in react in livewire. Seriously- helps a lot.
