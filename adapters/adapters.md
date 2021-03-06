# Creating Adapters for Offshore

Adapters are the *glue* connecting offshore core to the different data stores. Adapters expose 
[interfaces](interfaces.md), which imply a contract to implement certain functionality. This allows us to guarantee conventional usage patterns across multiple models, 
developers, apps, and even companies, making app code more maintainable, efficient, and reliable.

## Before writing code

Offshore github page doesn't list an adapter for a particular datastore and you think it would be a good idea to create one, 
good stuff! But before you start we recommend you [search github](https://github.com/search) for `offshore-datastorename`
and check if a project doesn't exist already. If it does it's generally a good idea to approach the author of an existing adapter and offer
your contribute instead of starting a new project. Most developers will welcome your help and the combined efforts will likely
result in a better quality adapter. If one doesn't exist we recommend you create a new project and name it following the 
convention: `offshore-datastorename`.

## How to start

The best way to start is to use [sails-generate-adapter](https://github.com/balderdashy/sails-generate-adapter) to generate all 
the boilerplate code. Alternatively you can fork [sails-adapter-boilerplate](https://github.com/balderdashy/sails-adapter-boilerplate)
but this project is no longer maintained.

Within the generated template file [adapter.js](https://github.com/balderdashy/sails-generate-adapter/blob/master/templates/boilerplate/adapter.js)
are inline comments explaining what each method does and what is expected to return.

## Going beyond `find`, `update`, `create` and `destroy`

These are the main methods to support the `Semantic` interface, yet there are a few more interfaces you can implement for
additional functionality and performance. For a comprehensive list check 
[Adapter Interface Reference](interfaces.md).

## Testing

Offshore provides an integration test suite named 
[offshore-adapter-tests](https://github.com/Atlantis-Software/offshore-adapter-tests) that can be used to test any adapter. 
The adapter boilerplate code includes a 
[runner.js](https://github.com/balderdashy/sails-generate-adapter/blob/master/templates/boilerplate/test/integration/runner.js) 
which will run these tests for you requiring you only to setup the interfaces you plan to support 
(and targeted version of Offshore) in the adapter's `package.json` file:

```javascript
{
  //...
  "offshoreAdapter": {
    "offshoreVersion": "~0.0.9",
    "implements": [
      "semantic",
      "queryable",
      "associations"
    ]
  }
}
```

In your adapter's directory, run:

```sh
$ npm test
```


## Other References
* [sails-docs - Intro to custom adapters](https://github.com/balderdashy/sails-docs/blob/master/contributing/intro-to-custom-adapters.md)
* [sails-docs - Adapters.md](https://github.com/balderdashy/sails-docs/blob/master/concepts/extending-sails/Adapters/Adapters.md): general description of what an adapter is
