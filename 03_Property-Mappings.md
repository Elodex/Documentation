# Property Mappings
One of the most important things for your index to work as expected is to create the desired property mappings for your Eloquent attributes.
That's where you tell your index how your indexed documents should be analyzed. E.g. what type your attributes have, which analyzer should be used and whether your relationships should be treated as nested objects.

Elodex provides a method to create default mappings based on the attribute settings you've (hopefully) already defined on your model class.
The following class properties will be used to create the mapping in the specified order:

1. **`dates`**: all attributes specified here will be mapped to an Elasticsearch date format accordingly. The `indexMappingDateFormat` property defines which date formatting will be used for the mapping, the default format is `yyyy-MM-dd HH:mm:ss`.  
*This does not work if your Eloquent model class uses a custom date through the `dateFormat` property !*
2. **`casts`**: the types specified in this property will be mapped to [Elasticsearch field dataypes][Elasticsearch field datatypes]. The attribute will not be added to the mapping if the type cannot be mapped.
3. **`indexMappingProperties`**: this property can be used if you want to specify any additional custom mappings.

Custom property mappings defined by the `indexMappingProperties` array overwrite any automatically determined mappings and have the following structure:
```php
class User extends Model implements IndexedModelContract
{
    use IndexedModelTrait;

    protected $indexMappingProperties = [
        'first_name' => [            // The name of the attribute
            'type' => 'string',      // Elasticsearch field datatype (NOT the PHP data type!)
            'analyzer' => 'simple',  // Elasticsearch analyzer to use for this field
        ],
    ];
```
For a detailed description of the needed structure for custom index mappings visit the [Elasticsearch documentation][Elasticsearch mapping] site.

Note that the mapping will respect the visibility setting of attributes specified by the `visible` and `hidden` properties on your class.
I.e. hidden properties will be excluded from the generated mapping.

To get all mappings for your model class in order to send them to the index manager you can call the `getIndexMappingProperties` method on a model instance.
```php
$mappings = (new Model)->getIndexMappingProperties();
```

The result can be used during the index creation as described in the _Create indices_ section.
```php
$mappings = [];

$foo = new Foo;
$mappings[$foo->getIndexTypeName()] = [
  '_source' => ['enabled' => true],
  'properties' => $foo->getIndexMappingProperties()
];

$indexManager->createIndex($indexName, $settings, $mappings);
```

If you want to update a class mapping after an index has been created or if you don't want to define the mappings during index creation you can do so by using the `putIndexMappings` class method.
```php
Model::putIndexMappings();
```
Alternatively you could use the `IndexManager` class and its `putMappings` method
```php
$defaultIndex = $indexManager->getDefaultIndex();
$mappings = $model->getIndexMappingProperties();

$indexManager->putMappings($defaultIndex, $model->getIndexTypeName(), $mappings);
```
The general rules about changing the index mappings apply. I.e. you might need to close your index or even reindex your data to change the mappings.

Note that Elasticsearch actually doesn't require you to explicitly put property mappings in order to add documents because it supports a [dynamic mapping][Elasticsearch dynamic mapping] mechanism.
But there's hardly any use case in which you would not want to use an explicit mapping for your Eloquent model classes.


## Property Mappings for Relationships
All property mappings for the relationships of a model are defined and created by the parent.
Using the `IndexedModel` trait on a related model class is not needed and will have no effect on the mappings created by the parent for this relationship.
I.e. a parent never calls `getIndexMappingProperties` on related models even if they implement this method.

You can however define a `indexMappingProperties` on your related model class which (if existent) will be used by the parent.
Even though this is not really necessary since you could as well directly define any custom property mappings of your relationships in the `indexMappingProperties` of your parent.


### Indexing Model Relationships
Your Eloquent models are usually not flat and may include several different relationships which you might want to include in your indexed document as nested objects.
You should be careful though and be aware of the implications that come with nested objects as described [here][Elasticsearch modeling your data].

To automatically add a relationship to the indexed document of a model you can simply add it to the `indexRelations` property.
```php
class User extends Model implements IndexedModelContract
{
    use IndexedModelTrait;

    protected $indexRelations = [ 'comments' ];
```
Indexed relations will be automatically loaded before the document representation for a model is created.
Any other already loaded relation will be ignored and not be included into the document!

Note that the Eloquent visibility properties `visible` and `hidden` don't have any effect on the relations defined in the `indexRelations` property.
A specifically hidden index relation will still load during the index document creation and will be integrated into the parents document.


[Elasticsearch field datatypes]: https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html#_field_datatypes "Elasticsearch field datatypes"
[Elasticsearch mapping]: https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html "Elasticsearch mapping"
[Elasticsearch dynamic mapping]: https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html#_dynamic_mapping "Elastic search dynamic mapping"
[Elasticsearch modeling your data]: https://www.elastic.co/guide/en/elasticsearch/guide/current/modeling-your-data.html "Elasticsearch modeling your data"
