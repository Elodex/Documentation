# Document Creation for the Index
The standard implementation of the `toIndexDocument` method provided by Elodex to create a document for a model works very similar to the `toArray` method.
With the exception of the way relationships are integrated into the parent document. See the next section for more details about documents for relationships.

If you want to modify the way indexed documents are being created you can do so by implementing your own `toIndexDocument` in your model class.

```php
<?php

namespace App;

use Elodex\Model as ElodexModel;

class Model extends ElodexModel
{
  public function toIndexDocument()
  {
    $doc = parent::toIndexDocument();

    $doc['added_index_field'] = 'foo';

    return $doc;
  }

```

Partial updates use the `getChangedIndexDocument` to create a document for the changed data.


## Documents of related Models
As described in *Indexing Model Relationships* any relationship specified in `indexRelations` will cause the relationship to be added to your parent's document.
Related models implementing the `Contracts\IndexedDocument` interface will be added with their document representation to the parent document.
If a related model doesn't implement the interface a fallback to the standard serialization method with `toArray` will be used.
