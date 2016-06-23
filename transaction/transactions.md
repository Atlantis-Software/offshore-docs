# Transactions

Transactions are an important feature of databases, as they allow correct recovery from failures and keep a database consistent even in cases of system failure. An Offshore transaction can concern multiple databases (as long as they have an adapter implementing the transacting interface) and all queries within it are executed as a single unit of work. Any failure will mean the databases will rollback any queries executed on that transaction to the pre-transaction state.

## How to initialize a transaction

Transactions are initialized through Offshore.Transaction() :

### .Transaction( `collections`, `callback` )

| Description | Types | Description | Required ? |
|    :---:    | :---: |    :---:    |    :---:   |
| collections | Collection Object or array of Collections | Given collections must be already loaded and initialized by Offshore | Yes |
| callback | function | First argument is a transaction object, second argument is callback | Yes |


```javascript
Offshore.Transaction([User, Pet], function(trx, cb) {
	// trx is your transaction
})
```

## How to use transactions

Once created, the transaction object contains all the collections it was initialized with.
They can be used like any other Collection object : 

```javascript
Offshore.Transaction([User, Pet], function(trx, cb) {
	trx.user.create({name: 'Bob'}).exec(function(err, myUser) {
		trx.pet.create([{type: 'cat', owner: myUser.id}, {type: 'dog', owner: myUser.id}],
		function(err, userPets) {

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
		trx.pet.createEach([{type: 'cat', owner: myUser.id}, {type: 'dog', owner: myUser.id}])
		.exec(function(err, userPets) {
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
