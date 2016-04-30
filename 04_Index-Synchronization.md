# Index Synchronization

## Adding Models to the Index
To add the document of a single model to your default index repository you can call `addToIndex` on the model instance.
```php
$user = new User();
$user->save();
$user->addToIndex();
```
Note that `addToIndex` will fail and throw an exception if the document already exists.

As described in _Document Creation for the Index_ the index document representation of a model and all its related objects is created by the `toIndexDocument` method.

If you want to add or replace a document you can use the `saveToIndex` method, this method will not fail if a document doesn't exist already.
```php
$user->saveToIndex();
```

You may also add a collection result of a model query to the index using the same methods.
```php
User::all()->addToIndex();

User::all()->saveToIndex();
```

Index operations on collections are always [bulk operations][Elasticsearch bulk API].
A `BulkOperationException` exception will be thrown if the operation fails for any of the specified models in the collection.  
Bulk deletions however don't follow this rule since Elasticsearch doesn't set the error flag in this case.

**Careful:** Your index will be in an inconsistent state after a bulk operation fails partially, Elasticsearch doesn't support transactions!
The bulk operation exception instance will contain a list of failed items which can be accessed through the `getFailedItems` method.
It's the programmers responsibility to catch and handle the `BulkOperationException` and to bring the index back to a consistent state.


### Removing Models from the Index
To remove a model's document from its index repository just call `removeFromIndex` on the instance.

```php
$user->removeFromIndex();
```

If you want to remove multiple documents as a result of a query you can do so by calling the same method on the resulting collection
```php
User::all()->removeFromIndex();
```


## Partial Updates
Partially updating your model is supported through the `updateIndex` method.

```php
$user->updateIndex();
```

It uses the `getChangedIndexDocument` on your model to create a document of changed attributes which will be used for the update operation.
The standard implementation of this method temporarily hides all unchanged attributes and then serializes attributes just like the `toArray` method does.

`getChangedIndexDocument` does not include any relationships or changed data from relationships.
This is because Eloquent doesn't have a tight binding between parents and their related models besides the ability to touch the timestamp of parents.

In other words: ***Partial updates are not supported for models with indexed relationships!***

If you partially update a model with indexed relationships it might become inconsistent because their related models didn't get updated in the nested documents.

Generally speaking: there're very few reasons to use partial updates at all due to the fact [how partial updates work in Elasticsearch][Elasticsearch partial updates].
Elasticsearch will always internally perform a full document update anyways.


## Synchronizing Eloquent Changes
Once you've filled your index repositories with your existing Eloquent models you usually want to keep your index in sync with your database.

This can be easily achieved by adding an [event subscriber][Laravel Event Subscribers] for all relevant Eloquent events.
Elodex provides a method to [generate a default implementation](10_Artisan-Commands.md#creating-eloquent-Synchronization-handlers) for your model classes.
```bash
$ php artisan make:es:sync-handler Model
```
You can then add the generated synchronization handler to the list of subscribers in your `EventServiceProvider` class.

You might notice that `ShouldQueue` is used for the generated event subscriber which will cause all event methods to be called on a queue, most likely asynchronously unless you use the `Sync` queue.
Using queued methods is highly recommended!

Index operations may fail or throw exceptions and you don't want that to happen as part of a user's request cycle.
Queues will automatically retry failed jobs depending on your queue configuration and they make it possible for you to handle job failures outside the user's request cycle.


## Synchronizing Index Relationships
If your indexed document contains nested objects you need to make sure that all parents of a changed related model get updated as well.

It's pretty obvious if you look at the structure of nested documents. Let's assume you've got an array of comments and in this case a comment is the parent of a user:
```php
  [
    'text' => 'Foo',
    'user' => [
      'id' => 1,
      'first_name' => 'John',
      'last_name' => 'Doe',
    ]
  ],
  [
    'text' => 'Bar',
    'user' => [
      'id' => 1,
      'first_name' => 'John',
      'last_name' => 'Doe',
    ]
  ],
```

If the user _John Doe_ changes, all comment entries containing this user have to be updated as well.
```php
  public function onSaved($user)
  {
    $user->saveToIndex();

    $user->comments->saveToIndex();
  }
```

So let's say you've got 1000 comments which belong to 1 user, that would mean 1001 documents have to be updated if this user changes.
That's something you should keep in mind if you decide to use relationships in indices.

[Laravel Event Subscribers]: https://laravel.com/docs/5.2/events#event-subscribers "Laravel Event Subscribers"
[Elasticsearch bulk API]: https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html "Elasticsearch bulk API"
[Elasticsearch partial updates]: https://www.elastic.co/guide/en/elasticsearch/guide/current/partial-updates.html "Elasticsearch partial updates"
