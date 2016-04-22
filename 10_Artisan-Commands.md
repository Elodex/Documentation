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

Note that this command does not create any property mappings for you.
You can however derive from the `Elodex\Console\CreateIndex` class and override the `createIndex` method to create your custom application specific index creation command.

```php
<?php

namespace App\Console\Commands;

use Elodex\Console\CreateIndex as BaseCreateIndex;

class CreateIndex extends BaseCreateIndex
{
    ...

    protected function createIndex($indexName, $settings)
    {
        $settings = array_merge($settings, [
            // put our settings here
        ]);
        $mappings = [
            // put your mappings here
        ];

        $this->indexManager->createIndex($indexName, $settings, $mappings);
    }
```

The parent class automatically makes an index manager instance available which can be used for all index management operations.
Consult the [Laravel documentation][Laravel Artisan] on how to make the Artisan command available to your application.


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


[Laravel Artisan]: https://laravel.com/docs/5.2/artisan "Laravel Artisan"
