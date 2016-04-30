# Index Management

General index management is available through the `IndexManager` class. You can either use dependency injection or access the `elodex.index` application singleton.
```php
app('elodex.index')->createIndex();
```

Another method is to call the `getDefaultIndexManager` method on your indexed model class
```php
User::getDefaultIndexManager()->createIndex();
```

The index manager is responsible for all administrative tasks related to your index. This includes creation and deletion of an index, putting index settings and mappings and using the [analyze][Elasticsearch indices - analyze] and [upgrade][Elasticsearch indices - upgrade] methods.

Index manager operations with an optional index name will use the the default index name specified in your configuration.


## Creating Indices
Before you can start synchronizing your indexed model classes with Elasticsearch you first need to create an index.
You usually create your index on your server while your app is in maintenance mode or during deployments.
Elodex includes a basic [Artisan command][Elodex Artisan commands] for that purpose.

Since index mappings should usually be set during index creation to prevent having to [reindex your data][Elasticsearch reindexing your data] you usually want to create a custom command as described [here][Elodex Artisan commands].

The structure of property mappings is defined by [Elasticsearch][Elasticsearch create indices - mappings]:
```php
$mappings = [
    'type_1' => [
        'properties' => [ ... ],
    ],
    'type_2' => [
        'properties' => [ ... ],
    ],
],
```

The type names `type_1` and `type_2` you see in this example should usually be determined using the `getIndexTypeName` method of your models rather than explicitily specifying them.
For an in-depth description about index creations in Elasticsearch take a look into the [Elasticsearch documentation][Elasticsearch create indices].

The [Property Mappings][Elodex Property Mappings] section describes how to automatically create mappings for your model classes which can then be used in the described mappings array.

If you decide not to use an Artisan command to create your indices you can make use of the `IndexManager` class as described above.


## Deleting Indices
Elodex provides an [Artisan command][Elodex Artisan commands] to delete existing indices via command line.

Inside your application the `deleteIndex` method of the `IndexManager` class will basically do the same.

```php
app('elodex.index')->deleteIndex('my_index');
```


[Elasticsearch indices - analyze]: https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-analyze.html "Elasticsearch indices - analyze"
[Elasticsearch indices - upgrade]: https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-upgrade.html "Elasticsearch indices - upgrade"
[Elasticsearch create indices]: https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html "Elasticsearch create indices"
[Elasticsearch create indices - mappings]: https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html#mappings "Elasticsearch create indices - mappings"
[Elasticsearch reindexing your data]: https://www.elastic.co/guide/en/elasticsearch/guide/current/reindex.html "Elasticsearch reindexing your data"
[Laravel Artisan]: https://laravel.com/docs/5.2/artisan "Laravel Artisan"
[Elodex Property Mappings]: 03_Property-Mappings.md "Elodex Property Mappings"
[Elodex Artisan commands]: 10_Artisan-Commands.md "Elodex Artisan Commands"
