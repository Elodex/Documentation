# Quickstart

This is a short step by step guide to quickly integrate and use Elodex in your Laravel project.
If you just want to play around a little and test the library than this guide is certainly the right place to start.

Note that your Elasticsearch server needs to listen to `127.0.0.1` port `9200` for this quickstart to work since we don't create a custom configuration here.

However if you want to integrate Elodex in your production code you certainly should read the full documentation and do some additional steps.

## 1. Add Elodex to your Composer File
```bash
$ composer require "elodex/elodex=dev-develop"
```

**Note**: If you want to use a development version instead of the latest stable version add the following stability setting to your `composer.json` file:
```
"minimum-stability": "dev"
```


## 2. Add the Elodex Service Provider
Edit the application config file under `config/app.php`, search for the `providers` entry and add the following line:
```php
  'providers' => [
    ...
    \Elodex\IndexServiceProvider::class,
  ],
```


## 3. Edit your Model Class
Open the file of the model for which you want to add the indexing functionality and add the needed includes:
```php
use Elodex\Contracts\IndexedModel as IndexedModelContract;
use Elodex\IndexedModel as IndexedModelTrait;
```

Specify that your model class implements the `IndexedModel` interface and include the `IndexedModel` trait:
```php
class MyModel implements IndexedModelContract
{
    use IndexedModelTrait;
```


## 4. Create your Index
Call the Artisan command to create your index.
```bash
$ php artisan es:create-index
Index 'my_custom_index_name' successfully created.
```


## 5. Play around
That's it, you can now use all indexing functions for your model class.

You could save all your models to the index ...
```php
Model::all()->saveToIndex();
```

And make an index search  ...
```php
$result = Model::indexSearch()
  ->match('name', 'foo')
  ->get();

$foundModels = $result->getModels();
```

From here on you can explore the rest of the documentation to try other Elodex functionalities.