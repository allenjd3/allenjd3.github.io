---
layout: post
title:  "Old is New Again"
date:   2020-01-12 20:02:00 -0400
categories: general
---
You may have heard about the latest and greatest new piece of front-end tech : Hotwire. Hotwire literally stands for HTML over the wire. If you don't have a clue about what I'm talking about, check out this site and then make your way back here. [hotwire.dev](https://hotwire.dev)

So now that you understand the basics I want to provide you with a couple of the small things that weren't obvious to me from the docks and they all relate to turbo-streams.

1. Hotwire is not restricted to rails but rails has a couple of neat adaptors for it.
You can install hotwire-rails, turbo-rails and or stimulus-rails. Hotwire-rails is just turbo-rails and stimulus rails bundled together. 

2. You need the redis adaptor installed in order to install turbo-rails. 
This is used if you use turbo-streams and connect it to a websocket. If you need the redis gem enabled that means one other thing- you need redis installed as well. I had a lot of luck using just plain docker for this. If you aren't super familiar with using redis in rails (which I wasn't) the config file that has your redis connection information is in **cable.yml**

3. Turbo Streams need some Model magic
Make sure you put the following for an example model called Post if you want to broadcast your changes.
```ruby
class Post < ApplicationRecord
    after_create_commit { broadcast_append_to "posts"}
    after_update_commit { broadcast_replace_to "posts"}
    after_destroy_commit { broadcast_remove_to "posts"}
end
```

4. You need to wrap your data that will be appended.
Its important to wrap it in something like this-
```ruby
<%= turbo_stream_from "posts" %>
  <%= turbo_frame_tag "posts" %>
    #important stuff in here
  <% end %>
<% end %>
```
Then within that frame tag each individual posts need to be wrapped in another frame tag with the dom_id helper function like below-
```ruby
<%= turbo_frame_tag dom_id(post) %>
  #individual post
<% end %>
```
Doing all of the above will allow you to add stuff to your list, remove things and updated things.

5. Streams need even more help if you want to broadcast errors.
The above will have you creating updating and destroying as long as you don't have any validation. Of course, you will always need some sort of validation so that really isn't feasible. In order to get your errors sent back and your turbo frame updated, you need something like the following in your controller-
```ruby
    respond_to do |format|
      if @post.save
        format.html { redirect_to posts_path, notice: 'Post was successfully created.' }
        format.json { render :show, status: :created, location: @post }
      else
        format.turbo_stream {render turbo_stream: turbo_stream.replace(@post, partial: "posts/form", locals: {post: @post})}
        format.html { render :new }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
```
The important bit here is the new format- turbo_stream. This allows you to pass back something from yuor controller that will replace your original frame.

Anyway, I hope that is all helpful information. Hotwire seems like its really going to help speed up development!
