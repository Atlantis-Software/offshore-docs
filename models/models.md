# Waterline Models

Models represent a structure of data which requires persistent storage. The data may live in any
data-store but is interfaced in the same way. This allows your users to live in PostgreSQL and your
user preferences to live in MongoDB and you will interact with the data models in the exact same way.

If you're using MySQL, a model might correspond to a table. If you're using MongoDB, it might
correspond to a collection. In either case, our goal is to provide a simple, modular way of managing
data without relying on any one type of database.

### How Do I Define A Model

Model definitions contain `attributes`, `validations`, `instance methods`, `lifecycle callbacks`
and `class methods`. To define a model you will extend the `Waterline.Collection` object and add
in your own `attributes` and methods.

By default an attribute named `id` will be automatically added to your model which will contain
an auto-incrementing number unique to each record. This will be your model's `primary key` and
will be indexed when available. You can override this if you would like to define your own primary
key factory or attribute.

Each model will also get two timestamp attributes added by default: `createdAt` and `updatedAt` which
will track when a record went into the datastore and when it was last updated.

```javascript
var Person = Waterline.Collection.extend({

  // Identity is a unique name for this model and must be in lower case
  identity: 'person',

  // Connection
  // A named connection which will be used to read/write to the datastore
  connection: 'local-postgresql',

  // Attributes are basic pieces of information about a model
  attributes: {
    firstName: 'string',
    lastName: 'string',
    age: 'integer',
    birthDate: 'date',
    emailAddress: 'email'
  }
});

module.exports = Person;
```

You can also set options for each attribute. These include `validations` and any indexing or unique
properties.

```javascript
var Person = Waterline.Collection.extend({

  identity: 'person',
  connection: 'local-postgresql',

  attributes: {

    // Don't allow two objects with the same value
    lastName: {
      type: 'string',
      unique: true
    },

    // Ensure a value is set
    age: {
      type: 'integer',
      required: true
    },

    // Set a default value if no value is set
    phoneNumber: {
      type: 'string',
      defaultsTo: '111-222-3333'
    },

    // Create an auto-incrementing value (not supported by all data-stores)
    incrementMe: {
      type: 'integer',
      autoIncrement: true
    },

    // Index a value for faster queries
    emailAddress: {
      type: 'email', // Email type will get validated by the ORM
      index: true
    }
  }
});
```

### Data Types and Attribute Properties

The following data types are currently available:

* string
* text
* integer
* float
* date
* time
* datetime
* boolean
* binary
* array
* json

These will map to the underlying database type if available. If a database doesn't support a type
a polyfill will be used. For example when using an array or json type in MySQL the values will be
stringified before being saved.

#### Attribute Properties

These properties are also available on an attribute and can be used to enforce various constraints
on the data.

###### defaultsTo

Will set a default value on an attribute if one is not supplied when the record is created. The supplied value can also be a 
function that waterline will run while creating the record.

```javascript
attributes: {
  phoneNumber: {
    type: 'string',
    defaultsTo: '111-222-3333'
  },
  id: {
    type: 'text',
    primaryKey: true,
    unique: true,
    defaultsTo: function() { return uuid.v4(); }
  }
}
```

###### autoIncrement

Will create a new auto-incrementing attribute. These should always be of type `integer` and will
not be supported in all datastores. For example MySQL will not allow more than one auto-incrementing
column per table.

```javascript
attributes: {
  placeInLine: {
    type: 'integer',
    autoIncrement: true
  }
}
```

###### unique

Ensures no two records will be allowed with the same value. This is a database level constraint so
in most cases a unique index will be created in the underlying data-store.

```javascript
attributes: {
  username: {
    type: 'string',
    unique: true
  }
}
```

###### index

Will create a simple index in the underlying datastore for faster queries if available. This is only
for simple indexes and currently doens't support compound indexes. For these you will need to create
them yourself or use a migration.

There is currently an issue with adding indexes to string fields. Because Waterline performs it's
queries in a case insensitive manner we are unable to use the index on a string attribute. There are
some workarounds being discussed but nothing is implemented so far. This will be updated in the
near future to fully support indexes on strings.

```javascript
attributes: {
  email: {
    type: 'string',
    index: true
  }
}
```

###### primaryKey

Will set the primary key of the record. This should be used when `autoPK` is set to false.

```javascript
attributes: {
  uuid: {
    type: 'string',
    primaryKey: true,
    required: true
  }
}
```

###### enum

A special validation property which will only allow values which match a whitelisted set of values.

```javascript
attributes: {
  state: {
    type: 'string',
    enum: ['pending', 'approved', 'denied']
  }
}
```

###### size

If supported in the datastore, can be used to define the size of the attribute. For example in MySQL
size can be used with a string to create a column with data type: `varchar(n)`.

```javascript
attributes: {
  name: {
    type: 'string',
    size: 24
  }
}
```

###### columnName

Override the attribute name before sending to a datastore. This allows you to have a different
interface for interacting with your data at the application layer and the data layer. It comes in
handy when integrating with legacy databases. You can have a nice API for your data and still allow
the data to be saved in legacy columns.

```javascript
attributes: {
  name: {
    type: 'string',
    columnName: 'legacy_data_user_name'
  }
}
```

Be warned, that Waterline may implement more keywords in the future which would conflict with any custom keywords in your application.

### Instance Methods

You can attach instance methods to a model which will be available on any record returned from a
query. These are defined as functions in your model attributes.

```javascript
var User = Waterline.Collection.extend({

  identity: 'user',
  connection: 'local-postgresql',

  attributes: {
    firstName: 'string',
    lastName: 'string',
    fullName: function() {
      return this.firstName + ' ' + this.lastName;
    }
  }
});
```

#### toObject/toJSON Instance Methods

The `toObject()` method will return the currently set model values only, without any of the instance
methods attached. Useful if you want to change or remove values before sending to the client.

However we provide an even easier way to filter values before returning to the client by allowing
you to override the toJSON() method in your model.

Example of filtering a password in your model definition:

```javascript
var User = Waterline.Collection.extend({

  identity: 'user',
  connection: 'local-postgresql',

  attributes: {
    name: 'string',
    password: 'string',

    // Override toJSON instance method to remove password value
    toJSON: function() {
      var obj = this.toObject();
      delete obj.password;
      return obj;
    }
  }
});
```

### Associations

Associations are available in Waterline starting in version `0.10`. See [Associations](associations.md) for
information on how to define and query relations between your models.

### Configuration

You can define certain top level properties on a per model basis. These will define how your schema
is synced with the datastore and allows you to turn off default behaviour.

###### identity

A required property on each model which describes the name of the model. This must be unique per
instance of Waterline and it must be in lower case.

```javascript
var Foo = Waterline.Collection.extend({

  identity: 'foo'

});
```

###### connection

A required property on each model that describes which connection queries will be run on. You can use
either a string or an array for the value of this property. If an array is used your model will have
access to methods defined on both adapters in the connections. They will inherit from right to left
giving the adapter from the first connection priority in adapter methods.

So for example if you defined connections using both `sails-postgresql` and `sails-mandrill` and the
`sails-mandrill` adapter exposes a `send` method your model will contain all the CRUD methods exposed
from `sails-postgresql` as well as a `send` method which will be run on the mandrill adapter.

```javascript
// String Format
var Foo = Waterline.Collection.extend({

  identity: 'foo',
  connection: 'my-local-postgresql'

});

// Array Format
var Bar = Waterline.Collection.extend({

  identity: 'bar',
  connection: ['my-local-postgresql', 'sails-mandrill']

});
```

###### autoPK

A flag to toggle the automatic primary key generation. If turned off no primary key will be created
by default and one will need to be defined.

```javascript
var Foo = Waterline.Collection.extend({

  identity: 'foo',
  connection: 'my-local-postgresql',

  autoPK: false

});
```

###### autoCreatedAt

A flag to toggle the automatic timestamp for createdAt.

```javascript
var Foo = Waterline.Collection.extend({

  identity: 'foo',
  connection: 'my-local-postgresql',

  autoCreatedAt: false

});
```

###### autoUpdatedAt

A flag to toggle the automatic timestamp for updatedAt.

```javascript
var Foo = Waterline.Collection.extend({

  identity: 'foo',
  connection: 'my-local-postgresql',

  autoUpdatedAt: false

});
```

###### schema

A flag to toggle schemaless or schema mode in databases that support schemaless data structures. If
turned off this will allow you to store arbitrary data in a record. If turned on, only attributes
defined in the model's attributes object will be allowed to be stored.

For adapters that don't require a schema such as Mongo or Redis the default setting is to be
schemaless.

```javascript
var Foo = Waterline.Collection.extend({

  identity: 'foo',
  connection: 'my-local-postgresql',

  schema: true

});
```

###### tableName

You can define a custom table name on your adapter by adding a `tableName` attribute. If no table
name is supplied it will use the identity as the table name when passing it to an adapter.

```javascript
var Foo = Waterline.Collection.extend({

  identity: 'foo',
  connection: 'my-local-postgresql',

  tableName: 'my-legacy-table-name'

});
```

#### Using an Existing Database
  
There might be times when you want to use an existing database in your models.

In this example, the WB Company has prefixed all of their fields with `wb_`. You'll notice that you can use the `tableName` attribute, but also `columnName` in the `attributes` object. Additionally, we will set `migrate: 'safe'` so that Waterline doesn't attempt to add/remove fields or otherwise automatically restructure the existing database.

```javascript
var Widget = Waterline.Collection.extend({
  identity: 'wbwidget',
  connection: 'wb-widget-database',
  tableName: 'wb_widgets',
  attributes: {
    id: {
      type: 'integer',
      columnName: 'wb_id',
      primaryKey: true
    },
    name: {
      type: 'string',
      columnName: 'wb_name'
    },
    description: {
      type: 'text',
      columnName: 'wb_description'
    }
    autoPK: false,
    autoCreatedAt: false,
    autoUpdatedAt: false,
  }
});
```
`autoPK`, `autoCreatedAt`, and `autoUpdatedAt` are set to false in this example, because we want the model to be read-only from existing fields.

### Class Methods

"Class" methods are functions available at the top level of a model. They can be called anytime after
a Waterline instance has been initialized.

These are useful if you would like to keep model logic in the model and have reusuable functions
available.

```javascript
var Foo = Waterline.Collection.extend({

  identity: 'foo',
  connection: 'my-local-postgresql',

  attributes: {},

  // A "class" method
  method1: function() {}

});

// Example
Foo.method1()
```
