# Highlighting
Elodex allows you to the [Elasticsearch Highlighting][Elasticsearch Highlighting] feature in order to highlight search results on fields.

You can simply call the `highlight` method on search queries to add highlighting for fields.
Elasticsearch will highlight fields with `<em>` tags by default, if you want to use custom tags you can do so by using the `withHighlightTags` method.


```php
$result = User::indexSearch()
    ->multiMatch(['first_name^4', 'last_name^2'], 'a', BoolQuery::MUST, ['type'=>'phrase_prefix'])
    ->highlight('first_name')
    ->highlight('last_name')
    ->withHighlightTags(['<em>'], ['</em>'])
    ->get();
```

Highlights are returned as a part of the [search result metadata][Elodex Search Results Metadata].
You can access the highlights either through the metadata dictionary returned by the `getMetadata` method or by using the `getHighlight` method which will return the highlighted fields for a specific document.
```php
$highlights = $result->getHighlight($modelId);
```


If you want a documents dictionary with merged highlighted fields you can use the `getHighlightedDocuments` method.
```php
$result->getHighlightedDocuments();
```

The resulting dictionary will look like this:
```php
[
  66 => [
    "first_name" => "Otha"
    "last_name" => "<em>Adams</em>"
  ]
  25 => [
    "first_name" => "<em>Aimee</em> <em>Adriann</em>"
    "last_name" => "Predovic"
  ]
]
```


[Elasticsearch Highlighting]: https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-highlighting.html "Elasticsearch Highlighting"
[Elodex Search Results Metadata]: https://github.com/Elodex/Documentation/blob/develop/06_Search.md#metadata "Elodex Search Results Metadata"
