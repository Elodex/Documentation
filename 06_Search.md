# Index Search

Indexed model classes provide a simple method to search for indexed documents of their type without having to directly access the corresponding index repository.
```php
$results = User::indexSearch()->get();
```

All search operations use a `Search` query builder utilizing the [Elasticsearch DSL library][Elasticsearch DSL library].
You can create a new search query builder with the static `indexSearch` method or with the `newIndexSearch` method on the model instance.  
The Elodex `Search` class provides some convenient methods to add common search queries and parameters just like the Eloquent query builder does.

```php
$results = User::indexSearch()
  ->prefix('first_name', 'John')
  ->sort('first_name')
  ->get();
```
Multiple queries are combined with a `must` [bool query][Elasticsearch bool query] by default.

You can specify other occurrence types as well:
```php
$results = User::indexSearch()
  ->prefix('first_name', 'John', BoolQuery::SHOULD)    // or
  ->match('last_name', 'Miller', BoolQuery::MUST_NOT)  // and not
  ->sort('first_name')
  ->get();
```

To add more complex queries use the `addQuery` method.
Consult the [Elasticsearch DSL documentation][Elasticsearch DSL] on how to build search queries of any type.

```php
$search = User::indexSearch();

$companyMatch = new MatchQuery('company.name', $term, ['type'=>'phrase_prefix']);
$nestedCompanyMatch = new NestedQuery('company', $companyMatch);

$search->addQuery(nestedCompanyMatch);

$results = $search->get();
```


## Search Results
Search operations return a `SearchResult` object containing information about the search and the results returned by the query.

By default the following information is available:
- `totalHits`, total number of hits found for the query.
- `maxScore`, the maximum score value of the document with the highest score.
- `took`, the execution time of the query in ms.
- `timedOut`, indicating if a query timed out.

You should be aware that searches might fail partially due to timeouts.
Timeouts are usually not critical, temporary and only cause some of the search results to not be returned.
The `getShards` method returns detailed information about how many shards failed.


You can use the `getDocuments` method to get the dictionary of all found documents. It is keyed by the identifier of the model, the desired order of the search result as specified by any search ordering remains intact.

Note that the Eloquent model objects for the found documents are not automatically loaded.  
This gives you the opportunity to work with the documents returned by the index instead of Eloquent model instances. It has the advantage of not having to perform an extra database query if not absolutely necessary.


### Getting Eloquent Models for a Search Result
If you need the Eloquent model instances instead of their documents you may use the `getModels` method on the search result instance.
This implies a certain overhead to fetch and create the model instances.
```php
$models = $results->getModels();
```

`getModels` by default automatically uses eager loading for the index relationships specified in the `indexRelations` property.
The first parameter of the method accepts an array of eagerly loaded relationships if you do not want to use eager loading or if you want to specify your own eagerly loaded relationships.
```php
$models = $results->getModels(['company'])
```

Note that the models created will not contain any metadata returned by the search with the exception for the score value and the document version.
Those values can be accessed via the `getIndexScore` and `getIndexVersion` methods.


### Metadata
Some search queries may return metadata like a score value or the document version along with the documents.
You can get the metadata dictionary with the `getMetadata` method on the search result object.
```php
$metadata = $results->getMetadata();
```

The metadata dictionary is keyed by the document ID.

```php
$score = $metadata[1]['_score'];
$version = $metadata[1]['_version'];
```


### Highlights
Searches containing [highlight queries][Elodex highlighting] return the highlight results in the metadata, i.e. the source documents will remain untouched.
If you want to get the documents combined with the highlights use the `getHighlightedDocuments` method.
This method will merge highlighted fields into their corresponding source documents.

```php
$documents = $results->getHighlightedDocuments();
```

If you just want to get a single highlight for a specific document use the `getDocumentHighlight` method. This method requires the document ID in order to return the wanted highlight.

```php
$highlight = $results->getDocumentHighlight(1);
```

For more details about highlighting see the [Highlighting section][Elodex highlighting].


### Suggestions
Suggestions can be part of a search query as described [here][Elodex suggestions], they are returned as a `SuggestResult`.
The suggestions for a query can be accessed via the `getSuggestions` method on the search result.

```php
$suggestions = $results->getSuggestions();
```

For more details about suggestions see the [Suggestions section][Elodex suggestions].


## Searching Relationships
As explained in the _Indexing Model Relationships_ section relationships are by default mapped as nested objects inside indexed documents.

This means that you have to use [nested queries][Elasticsearch DSL nested query] if you want to search in fields of your nested documents.

```php
// Create a query for the 'comment' relationship of the user.
$commentQuery = new WildcardQuery('comment.description', "*hello*");

$results = User::indexSearch()
  ->nestedQuery('comment', $commentQuery)  // add the nested query for the 'comment' relation
  ->get();
```


## Pagination
You can paginate search results by calling the `paginate` method on `Search` query builder instances.
Pagination in Elodex basically works like [pagination for Eloquent queries][Laravel Pagination].
```php
$results = User::indexSearch()
  ->paginate(5);
```
The paginator will return the documents of the search result by default if you iterate through the elements.  
If you need the model instances to be loaded from the DB you can call the `getItems` method just like you can on the `SearchResult` instance itself.
```php
$users = $results->getItems();
```
Accessing the underlying search result instance is possible through the `getSearchResult` method.

You should be aware of the [pagination limitations][Elasticsearch pagination] that come with Elasticsearch.


## Search Queries and Options for Chaining
Almost all queries can be simply chained on the search class itself without having to explicitly create a query object and adding it to the search.  
The last two optional parameters are usually the the bool operator and the array of additional parameters.


### Available Search Queries
Here's a brief list of all queries combined into one search, for a more detailed list and description see the `Search` class and the [Elasticsearch DSL library][Elasticsearch DSL library]:
```php
Modell::indexSearch()
  ->term('first_name', 'term')
  ->terms('first_name', ['term1', 'term2'])
  ->commonTerms('last_name', 'this is bonsai cool')
  ->prefix('last_name', 'Mill')
  ->match('street', 'this is a test')
  ->multiMatch(['first_name', 'last_name'], 'this is a test')
  ->regexp('city', '[a-zA-Z]+')
  ->wildcard('country', 'a*')
  ->fuzzy('description', 'ki')
  ->queryString('foo', 'this AND that OR thus');
```

A [match all query][Elasticsearch DSL match all] is available as well:
```php
Model::indexSearch()
  ->match_all();
```


### Sorting and Result Limit
[Sorting][Elasticsearch search request sort] basically works like it does with Eloquent queries:
```php
Model::indexSearch()
  ->orderBy('first_name')
  ->orderBy('last_name', 'desc');
```

Offsets and limiting the number of search results are available as well:
```php
Model::indexSearch()
  ->offset(5)
  ->limit(10);
```


[Elasticsearch DSL library]: https://github.com/ongr-io/ElasticsearchDSL "Elasticsearch DSL library"
[Elasticsearch DSL]: https://github.com/ongr-io/ElasticsearchDSL/blob/master/docs/index.md "Elasticsearch DSL"
[Elasticsearch DSL nested query]: https://github.com/ongr-io/ElasticsearchDSL/blob/master/docs/Query/Nested.md "Elasticsearch DSL nested query"
[Elasticsearch DSL match all]: https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-all-query.html "Elasticsearch DSL match all"
[Elasticsearch bool query]: https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html "Elasticsearch bool query"
[Elasticsearch pagination]: https://www.elastic.co/guide/en/elasticsearch/guide/current/pagination.html "Elasticsearch pagination"
[Elasticsearch search request sort]: https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-sort.html "Elasticsearch Sort"
[Laravel Pagination]: https://laravel.com/docs/5.2/pagination#displaying-results-in-a-view "Laravel Pagination"
[Elodex highlighting]: 07_Highlighting.md "Elodex Highlighting"
[Elodex suggestions]: 09_Suggestions.md "Elodex Suggestions"
