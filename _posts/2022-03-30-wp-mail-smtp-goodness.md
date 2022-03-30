---
layout: post
title:  "WP Mail SMTP Goodness"
date:   2022-03-30 07:40:00 -0400
categories: general
---
I recently migrated a wordpress site that I manage to trellis. Trellis is a deployment tool which makes deploying wordpress sites a breeze. Its fairly opinionated about how everything works however so I wasn't sure how it would go. Here are my initial thoughts-

## Wow it was easy to set up!

Rebuilding the site which used to take an entire afternoon took me about 30 minutes with trellis. Basically all I had was a fresh ubuntu instance on Digital Ocean, my sql dump file from my previous page and the contents of the wp contents folder. Trellis set up everything really quickly and its more secure than I could ever make it.

## Hmm- something is weird here...

I immediately started noticing odd issues with wordfence. I uninstalled the plugin and reinstalled- all started working again. Then I started to get fatal errors blamed on yoast seo. I disabled the plugin and everything was working again. 

## Yoast didn't like wp smtp

In my efforts to figure out why yoast wasn't working correctly I discovered that if I disabled WP SMTP then yoast would stop killing my post editing. Then I noticed something odd. Apparently when I migrated the site, WP SMTP didn't have all of its database tables that it needed. My simple solution: uninstall and reinstall. I was hoping for a quick solution like with wordfence but in this case, reinstalling did nothing. 

Long story short, I found a setting that allowed me to delete all data from WP SMTP from the database. It was in the miscellaneous tab of the WP SMTP settings. 

<img width="958" alt="Screen Shot 2022-03-30 at 9 44 01 AM" src="https://user-images.githubusercontent.com/8092154/160849234-cd18b464-02c3-476b-ac4c-3dbf1ecac4a4.png">

After I checked that box- I just uninstalled and reinstalled the plugin. The only downside was having to find my mailgun key again but that wasn't a huge deal. WP SMTP installed all its necessary tables and we are back in business

Overall, I love trellis. It is ten times quicker to make changes to the site and if something were to happen to my current site it wouldn't take long to completely rebuild it elsewhere. I give it two thumbs and two big toes up.
