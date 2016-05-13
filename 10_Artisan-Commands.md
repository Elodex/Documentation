# Artisan Commands
Elodex ships with some standard commands for basic index management purposes.
All commands use the prefix "es". None of the commands require you to specify an index name.
If no index name is explicitly specified the default index name as defined in your configuration is used.

You can either use the short parameter `-I <IndexName>` or the long version `--index=<IndexName>` to provide an index name.

The return value of all commands is `0` in case of success and non-zero on failures.


## Index Creation
```bash
$ php artisan es:create-index
```

The command will not overwrite an existing index by default but rather fail.
If you want to force the deletion of existing indices the parameter `--reset` can be used.
```bash
$ php artisan es:create-index --reset
```

Note that this command does not create any property mappings unless you use the `--models` parameter.
```bash
$ php artisan es:create-index --models=User,Company
```

### Custom Index Creation Command
You can derive from the `Elodex\Console\CreateIndex` class and override the `createIndex` method to create your custom application specific index creation command.
One use case could be if you do not want to specify all model classes via command line parameter or if you want to add custom settings.

```php
<?php

namespace App\Console\Commands;

use Elodex\Console\CreateIndex as BaseCreateIndex;

class CreateIndex extends BaseCreateIndex
{
    ...

    protected function createIndex($indexName, $settings, array $models = [])
    {
        $settings = array_merge($settings, [
            // put our settings here
        ]);
        $models = [\App\User::class, \App\Company::class];

        parent::createIndex($indexName, $settings, $models);
    }
```
The parent class automatically makes an index manager instance available via the `indexManager` property which can be used for all index management operations.
Consult the [Laravel documentation][Laravel Artisan] on how to make an Artisan command available to your application.

The method to get the property mappings `getPropertyMappings` can be overriden as well.
Even though it is usually unnecessary to do so since you can directly define custom [property mappings](03_Property-Mappings.md) on your model classes.
```php
    protected function getPropertyMappings(array $models = [])
    {
        $mappings = array_merge(parent::getPropertyMappings($models), [
            // put your additional custom mappings here
        ]);

        return $mappings
    }
```

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

The type names `type_1` and `type_2` in this example should usually be determined using the `getIndexTypeName` method of your models rather than explicitily specifying them.
For an in-depth description about index creations in Elasticsearch take a look into the [Elasticsearch documentation][Elasticsearch create indices].


## Index Deletion
```bash
$ php artisan es:delete-index
```

The deletion command will prompt you for deletions by default, if you want to quietly force the deletion use `--force` as a parameter.

Note that you can delete all indices with one command by using a wildcard value `*` for the index name.

```bash
$ php artisan es:delete-index --index=*
```


## Opening / Closing Indices
Some changes to the index require you to previously lock your index before you can perform them.
One example would be if you want to change an analyzer as part of the index mapping properties.

You can close an index with the following command
```bash
$ php artisan es:close-index
```

To reopen a closed index use
```bash
$ php artisan es:open-index
```


## Seeding
To seed **all** indexed models of one ore more model class use the `es:seed` command.
```bash
$ php artisan es:seed Model
```
The first parameter accepts a comma separated list of indexed model classes to seed multiple classes.
```bash
$ php artisan es:seed Model1,Model2
```

The command will fail if the index is not empty and any of the models to seed already exists.
You can use the `--save` parameter to add or replace any existing indexed models, this will however not clean the index before the saving takes place.


## Show Index Mappings
To display an overview of all index mappings which have been set for your types use the `es:get-mappings` command.

```bash
$ php artisan es:get-mappings
Index property mappings for type 'users':
+---------------------+---------+---------------------+----------+------------------+
| Property            | Type    | Format              | Analyzer | Child properties |
+---------------------+---------+---------------------+----------+------------------+
| birthday            | date    | yyyy-MM-dd HH:mm:ss |          |                  |
| company             | nested  |                     |          | name             |
| email               | string  |                     |          |                  |
| updated_at          | date    | yyyy-MM-dd HH:mm:ss |          |                  |
+---------------------+---------+---------------------+----------+------------------+
```


## Index Statistics
If you want to check some basic statistics about your index you can use the `es:get-stats` command.

```bash
$ php artisan es:get-stats
Shards
  10 total, 5 successful, 0 failed
All Indices
  Documents:   9209 total, 0 deleted
  Store:       1526657 Bytes
  Search:      5 queries, 0 ms
  Query Cache: 0 hits, 5 misses, 0 Bytes
```

This command also supports a `--dump` parameter which will return the raw data returned by the Elasticsearch client.


## Index Settings
To show the settings for an index use the following command:
```bash
$ php artisan es:get-settings
```

Global custom analyzers will be printed as well.


## Analyze Command
The `es:analyze` command helps you to quickly check the functionality of your analyzers.

```bash
$ php artisan es:analyze simple "Hello World"
+-------+--------------+------------+------+----------+
| Token | Start offset | End offset | Type | Position |
+-------+--------------+------------+------+----------+
| hello | 0            | 5          | word | 0        |
| world | 6            | 11         | word | 1        |
+-------+--------------+------------+------+----------+
```


## Creating Eloquent Synchronization Handlers
The following command creates a standard index synchronization event subscriber implementation for a specified indexed model class.
```bash
$ php artisan make:es:sync-handler Model
```
The resulting generated class can be found in the `app/Listeners` folder with the name pattern `<Model>IndexSyncHandler`.
You can [activate the subscriber][Laravel Event Subscribers] in your app via the `EventServiceProvider` just like any other subscriber.

If no namespace is included in the `name` parameter for the model class, the `App` namespace will be used.

The default implementation of the generated subscriber class already provides almost everything you need for a proper synchronization of your Eloquent model class with the index.
There're only two custom implementations that usually need to be added:

1. The synchronization of indexed relationships as [described here](04_Index-Synchronization.md#synchronizing-index-relationships).
2. A proper job failure handling in the `failed` method.



[Laravel Artisan]: https://laravel.com/docs/5.2/artisan "Laravel Artisan"
[Laravel Event Subscribers]: https://laravel.com/docs/5.2/events#event-subscribers "Laravel Event Subscribers"
[Elasticsearch create indices]: https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html "Elasticsearch create indices"
[Elasticsearch create indices - mappings]: https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html#mappings "Elasticsearch create indices - mappings"
