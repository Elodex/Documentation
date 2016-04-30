# Scrolling
If you ever need to [reindex all of your documents][Elastcisearch reindex] or need to retrieve a large number of documents efficiently you should consider using a scroll query.

Elodex makes it very easy for you to use scrolling without having to worry about the scroll ID or releasing a scrolling context.

```php
User::indexSearch()
  ->prefix('first_name', 'John')
  ->scroll('2m', function ($results) {
      foreach ($results as $document) {
        // ...
      }
});
```

As you can see you can build up your query just like you would for a normal search query. But instead of calling `get` to get the search results you call the `scroll` function.  
The first parameter is a timeout value as described [here][Elastcisearch scroll] and defines how long a scrolling window will be kept open, the second is a callback function to use for each scrolling window.

You usually don't want to use scrolling as part of a request cycle but rather for asynchronous processing on a queue or for Artisan commands.


[Elastcisearch reindex]: https://www.elastic.co/guide/en/elasticsearch/guide/current/reindex.html "Elasticsearch reindex"
[Elastcisearch scroll]: https://www.elastic.co/guide/en/elasticsearch/guide/current/scroll.html "Elasticsearch scroll"