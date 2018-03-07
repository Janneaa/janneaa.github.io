Date: 2012-01-14
Title: Drupal tip: Add your own fields to Solr search index
Slug: drupal-tip-add-your-own-fields-to-solr-search-index
Category: Drupal
Tags: Drupal, Solr, PHP

<div class="intro-image border">
  <img src="/images/drupal-solr.png" alt="Drupal and Solr" class="no-border" />
</div>

_This is a repost from my old blog._

Drupal 7 is the PHP content management system that needs no introduction. But if you also want an excellent search engine to use with it then [Apache Solr](http://lucene.apache.org/solr/) is the best choice. Solr builds on Lucene and adds some nice features on top of it.

There is a very good [Drupal module](http://drupal.org/project/apachesolr) to integrate the search engine with. It simplifies the Solr installation and adds most of the basic node fields into search index by default. But what about when you need to **define your own fields**?

Like any of the Solr tutorials will tell you, Solr's index (and fields) is defined in its **schema.xml** file, but you don't even need to touch that to add some basic fields. Luckily the Solr module provides some useful hooks that it uses during indexation.

For Solr indexing
-----------------

You need to use hook_apachesolr_index_document\_build. This hook is fairly new in the Solr module (since version 7.x-1.0-beta14) and it replaces the deprecated hook_apachesolr_update_index hook. This change actually confused me because I hadn't noticed it and my custom fields weren't being indexed anymore. Here's how to use it in your custom module:

    <?php
    function MY_MODULE_apachesolr_index_document_build(ApacheSolrDocument
        $document, $entity, $entity_type, $env_id) { 
        $price = $entity->field_price;
        $document->addField('ss_product_price', $price / 100);
    }

Here I've fetched the price field of a node and added it into the Solr document. Of course you have to replace the beginning of the method name with your own module's name.

One trick you have to notice here is that I've prefixed the field with "ss\_". Solr's schema supports **dynamic field definitions**. For example all fields starting with "ss_" are defined as string types. You need to **pay attention to the type** you want. If you have a text field that you want to be searchable you should probably use text field type.

For Solr searching
------------------

By default this new field doesn't get returned when you search from Solr and therefore you need to use the **hook_apachesolr_query_alter** hook.

    <?php
    function MY_MODULE_form_apachesolr_query_alter($query) { 
        $query->addParam('fl', 'ss_product_price');
    }

This tells Solr to add the new field into the set of returned fields. Simple as that!

End result
----------

Now you've got the new field in Solr's search index. If you also want to be able to search its contents, pay attention to the type you define it as and make sure that Solr's QueryParser uses the field.

