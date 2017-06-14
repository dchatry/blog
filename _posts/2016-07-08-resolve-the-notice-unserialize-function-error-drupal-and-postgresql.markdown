---
layout: post
title: Resolve the "Unserialize" error on Drupal and PostgreSQL
date: 20:52 05-08-2016
headline: "Notice: unserialize() [function.unserialize]: Error at offset..."
taxonomy:
    category: blog
    tag: [database]
---

If you are working with **Drupal** and **PostgreSQL**, this is probably not the first time you encountered some weird compatibility issues, but this one is pretty twisted.

It happened to me after having restored a database dump and trying to access the homepage of my website, I kept getting hit by "**Notice: unserialize() [function.unserialize]: Error at offset[...]**" messages that occured multiple times during Drupal's boostraping process and ultimatly shutting down with fatal errors on common core functions like `user_access()` that couldn't be found. I couldn't even run the `update.php` script!

After scouring the Google for answers and following [guides after guides](https://www.drupal.org/node/529866) adressing this very issue, it turned out that the culprit was the new version of PostgreSQL. [Starting with 9.0](https://www.postgresql.org/docs/9.1/static/datatype-binary.html), it introduced a new "hex" format for `bytea` type output, which doesn't do too well with Drupal who's apparently expecting the good ol' "escape" format.

So the solution for this is just to set the database's `bytea_output` format to escape:
```
ALTER DATABASE yourdb SET bytea_output='escape'; 
```
After that just clear your cache with:
```
drush cc all
```
And you should be good to go!