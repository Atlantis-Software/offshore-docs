# Offshore Installation

Offshore is available via NPM.

```sh
$ npm install --save offshore
```
Offshore ships without any adapters, so you will need to install these separately. For example:

```sh
$ npm install --save offshore-sql
$ npm install --save-dev offshore-memory
```

You can install any number of adapters into your application.

The `offshore-disk` and `offshore-memory` adapters are common choices for development and testing.

If you are new to Node, we have a [guide](new-to-node.md) to help you get started on your preferred platform.
