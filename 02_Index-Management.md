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
Elodex includes a basic [Artisan command](10_Artisan-Commands.md#index-creation) for that purpose.


### Property Mappings
Index property mappings should usually be set during index creation to prevent having to [reindex your data][Elasticsearch reindexing your data]. The [index creation command](10_Artisan-Commands.md#index-creation) supports the creation of default property mappings for model classes through the `--models` parameter.

The [Property Mappings](03_Property-Mappings.md) section describes in detail how to set mappings for indexed model classes which will be used by the index creation command with the described option.

If you need further settings or want to perform custom actions during index creation you can create a custom command as described [here](10_Artisan-Commands.md#index-creation).

If you decide not to use an Artisan command to create your indices you can make use of the `IndexManager` class as described above.


### Analyzers
Custom analyzers can be defined in the Elodex configuration and will be automatically set during index creations.
```php
    /*
      |--------------------------------------------------------------------------
      | Index Analyzers
      |--------------------------------------------------------------------------
      |
      | Analyzers added to the ElasticSearch index during index creation.
      |
     */
    'analyzer' => [
        'my_text_analyzer' => [
            'type' => 'custom',
            'char_filter' => ['html_strip'],
            'tokenizer' => 'standard',
            'filter' => ['standard', 'lowercase', 'stop'],
        ],
    ],
```


## Deleting Indices
Elodex provides an [Artisan command](10_Artisan-Commands.md#index-deletion) to delete existing indices via command line.

Inside your application the `deleteIndex` method of the `IndexManager` class will basically do the same.

```php
app('elodex.index')->deleteIndex('my_index');
```


[Elasticsearch indices - analyze]: https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-analyze.html "Elasticsearch indices - analyze"
[Elasticsearch indices - upgrade]: https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-upgrade.html "Elasticsearch indices - upgrade"
[Elasticsearch reindexing your data]: https://www.elastic.co/guide/en/elasticsearch/guide/current/reindex.html "Elasticsearch reindexing your data"
[Laravel Artisan]: https://laravel.com/docs/5.2/artisan "Laravel Artisan"
