# Suggestions
There are two ways to use [suggesters][Elasticsearch Suggesters] in Elasticsearch. You can either add a suggest request to a search alongside a standard search request or you can make a separate suggest request.

Elodex currently only supports [term suggesters][Elasticsearch Suggesters - Term].

Note that all suggestions are index specific but not type specific.
In other words a suggestion query will always return results from all your indexed models.

The response data of a suggestion query is always a `SuggestResult` instance which itself contains a dictionary of the suggestions.
See the official [Elasticsearch documentation][Elasticsearch Suggesters] on how the dictionary is structured.


## Suggest Request as Part of a Search Query
You can use `suggestTerm` on a search query to combine it with a suggestion query.
```php
$results = User::indexSearch()
  ->match('first_name', 'Adrian')
  ->suggestTerm('my-suggestion-1', 'Adrian')
  ->suggestTerm('my-suggestion-2', 'Miller')
  ->get();
```

The first parameter of `suggestTerm` is a custom identifier for the suggestion which is needed to access the wanted suggestion in the suggestion result.
The search results object itself contains the [suggestion result][Elodex Search Results - Suggestions] as part of the response. It can be retreived with the `getSuggestions` method and will returns an instance of `SuggestResult`.

```php
$results->getSuggestions();
```


## Separate Suggest Request
Standalone suggest requests are different from standard search requests and use the `Suggest` query class.
There's no convenient way to create a suggest query through an indexed model class since suggestions are never specific to a model class.

Instead you simply create a new `Suggest` instance:
```php
$results = Suggest::create()
  ->term('my-suggestion-1', 'Adrian')
  ->term('my-suggestion-2', 'Miller')
  ->get();
```

The `get` method performs the query and returns a `SuggestResult` instance.



[Elasticsearch Suggesters]: https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html "Elasticsearch Suggesters"
[Elasticsearch Suggesters - Term]: https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters-term.html "Elasticsearch Suggesters - Term"
[Elodex Search Results - Suggestions]: https://github.com/Elodex/Documentation/blob/develop/06_Search.md#suggestions "Elodex Search Results - Suggestions"