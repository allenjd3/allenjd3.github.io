---
layout: post
title:  "whereIn Invalid Parameter Number Goodness"
date:   2022-09-22 15:40:00 -0400
categories: general
---
I'm posting this so that I remember in a few years when this happens again...

We had a very interesting error that simply said- SQLSTATE[HY093]: Invalid parameter number. The query that it showed looked fine and so I spent a significant amount of time scratching my head wondering if I was losing my marbles.

Here was the query- 
```sql
select * from posts where uuid in (random-uuid-here)
```

Looks fine right? Well turns out we were passing an array of an array into this whereIn- something that looked similar to this-

```php
[
  [
    'uuid_1',
    'uuid_2',
    'uuid_3',
  ]
]
```

And this generated a query that looked like this-

```sql
select * from posts where uuid in (?)
```

It was attempting to bind each uuid to that one parameter so it gave the invalid parameter number error. What we needed was something closer to this-

```sql
select * from posts where uuid in (?, ?, ?)
```

The solution was to flatten the array down to a single depth so whereIn would know to bind the correct number of parameters.

Future me- if you ever read this, make sure you pass flat arrays into whereIn. Pretty please?
