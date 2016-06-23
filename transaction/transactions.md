# Transactions

Transactions are an important feature of relational databases, as they allow correct recovery from failures and keep a database consistent even in cases of system failure. All queries within a transaction are executed on the same database connection, and run the entire set of queries as a single unit of work. Any failure will mean the database will rollback any queries executed on that connection to the pre-transaction state. 

## How to initialize a transaction

Transactions are initialized through Offshore.Transaction() :

### .Transaction( `listCollections`, `callback` )

|    Description     |                     Accepted Data Types                              | Required ? |
|--------------------|----------------------------------------------------------------------|------------|
|   listCollections  |   Collection Object or array of Collections                          |   Yes      |
|     callback       |   function -> first arg is a transaction object, second is callback  |   Yes      |


```javascript
Offshore.Transaction([User, Pet], function(trx, cb) {
	// trx is your transaction
})
```

## How to use a transaction object

Once created, the transaction object contains all the collections it was initialized with.
They can be used like any other Collection object, the only difference being that they can only
be used with defered (i.e. with an .exec call) : 

```javascript
Offshore.Transaction([User, Pet], function(trx, cb) {
	trx.user.create({name: 'Bob'}).exec(function(err, myUser) {
		trx.pet.create([{type: 'cat', owner: myUser.id}, {type: 'dog', owner: myUser.id}]).exec(function(err, userPets) {


		});
	});
})
```

Commit and rollback are done through the Offshore.Transaction() callback, the first
argument being for errors : if thruthy, the transaction will be rollbacked, else it will
be commited. The second argument will be passed to transaction .exec callback, after commit.
Offshore.Transaction() can be chained with an .exec().

```javascript
Offshore.Transaction([User, Pet], function(trx, cb) {
	trx.user.create({name: 'Bob'}).exec(function(err, myUser) {
		if (err) {
			// if error, rollback
      			cb(err);
    		}
		trx.pet.createEach([{type: 'cat', owner: myUser.id}, {type: 'dog', owner: myUser.id}]).exec(function(err, userPets) {
			if (err) {
				// if error, rollback
      				cb(err);
    			}
			// Commit
			cb(null, userPets);
		});
	});
}).exec(function(err, trxResult) {
    	if (err) {
      		// err will contain the error you passed in the callback
    	}
	// trxResult contains userPets
```
