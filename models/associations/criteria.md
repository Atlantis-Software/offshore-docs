# Association Criteria

like models themselves, theirs associations could have a criteria key.

## Example Ontology

```javascript
// Person.js
module.exports = {
  connection: 'ourMySQL',
  attributes: {
    id: {
      type: "integer",
      primaryKey: true
    },
    name: "string",
    sheeps: {
      collection: "Animal",
      via: "person",
      criteria: {
        legsCount: 4,
        color: "White",
        sound: "baa"
      }
    }
  }
};
```

```javascript
// Animal.js
module.exports = {
  connection: 'ourMySQL',
  attributes: {
    id: {
      type: "integer",
      primaryKey: true
    },
    name: "string",
    color: "string",
    legsCount: "integer",
    sound: "string",
    person: {
      "model": "person"
    },
    age: "integer"
  }
};
```

so Animals could have differents colors, legsCounts and sounds and belongs to a person but person's sheep could only be four legged, White and sound "baa".

if you try:
```javascript
person.create({name: 'john', sheeps: [{name: 'boby', color: 'Black', legsCount: 4, sound: 'bark'}]})
```
will error because a sheep can't be Black or bark ...

although :
```javascript
animal.create({name: 'boby', color: 'Black', legsCount: 4, sound: 'bark', person: 1})
```
will create the animal but
```javascript
person.find(1).populate('sheeps')
```
will not populate previously created animal because a sheep could only be four legged, White and sound "baa".

if you don't specify color, legsCount or sound in nested create like this:
```javascript
person.create({name: 'john', sheeps: [{name: 'shaun'}]})
```
as shaun is a sheep, it's created with defaults legsCount: 4, color: "White" and sound: "baa".
