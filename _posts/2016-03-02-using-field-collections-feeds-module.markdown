---
layout: post
title: Using "Field collection feeds" in Drupal 7
date: 19:18 11-03-2015
headline: Dealing with feeds and field collections
taxonomy:
    category: blog
    tag: [drupal]
---

Importing content with [Feeds][1] can be a little tricky, especially when dealing with field collections. By chance, the contrib module [Field collection Feeds][2] can help us achieve that without too much hassle. With a simple use case, customers and orders, we will see how to configure the feed processor to link both fields and entity. 
## Data model

Consider the following data model: 

<img src="using-field-collections-feeds-module/images/datam_1103151.png" alt="datam_110315" class="aligncenter size-full wp-image-157" />

Let's say we have already imported the customers into our Drupal database, the customer entity is just 4 fields: a *unique* **id** (*field_customer_id*), a **name**, an **address** and an **orders** field collection (*field_customer_orders*). From a Drupal standpoint, the **orders** field collection consists of 2 fields: the **item** ordered and the **order date**. 
## Feed configuration

Create a new "Orders" feed and configure the basic settings. The interesting part lays in the "Processor" section, click on "Change" and pick the "Field collection processor". 

Now head over to the "Field collection processor Settings" section. In the **Bundle** field, choose the field collection you want to import: *field_customer_orders*.

Again in the **Field name** field, select your field collection (here *field_customer_orders*). The **Host entity type** field wants to know the type of the entity where the field collection is used, in our case the "Orders" field collection is used in the entity "Customer" which is of type **node**.

Finally, we need to tell the processor which field to use when linking the order and the customer, here *field_customer_id* is the **unique** identifier for the customer entity so it's the obvious choice. Also put "guid" in the *identifier field name* field.

Don't forget to enable the **Is field** checkbox, that way the processor knows that our GUID is a field and not a property (a property could be *title*, *nid* etc). 

<img data-action="zoom" src="using-field-collections-feeds-module/images/3.png" alt="3" class="aligncenter size-full wp-image-162" /> The processor is almost configured, we just need to map the fields from the CSV file to the field collection fields. 
<img data-action="zoom" src="using-field-collections-feeds-module/images/4.png" alt="4" class="aligncenter size-full wp-image-163" />

The link with the host entity (customer) is done via the target "Host Entity GUID". 

## Import

We're done! Now we can import the CSV file containing the orders, for example: 

    item;order_date;customer_id
    "Keyboard";"03-01-2015";1
    "Mouse";"03-01-2015";1
    "TV";"03-01-2015";1
    "PS4";"02-10-2015";2
    "PS4 Controller";"01-11-2015";2
    "Speakers";"03-12-2015";2
    "Headphones";"11-10-2014";2

That's it! The orders have been imported and linked to their respective customers: 

<img data-action="zoom" src="using-field-collections-feeds-module/images/5.png" alt="5" class="aligncenter size-full wp-image-159" /> If you want to try this configuration by yourself, here is the [exported feed configuration][3].

 [1]: https://www.drupal.org/documentation/modules/feeds
 [2]: https://www.drupal.org/project/field_collection_feeds
 [3]: /home/using-field-collections-feeds-module/feed.txt