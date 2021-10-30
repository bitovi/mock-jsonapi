# mock-jsonapi

## Goals 

1. Easily generate a mock JSONAPI that handles relationships, filtering, sorting, etc
2. Make it easy to work with backend teams. (Hand over the same data model) .. how should this work with the docs?
3. Allow customization

## Inspiration


- [JSONServer](https://github.com/typicode/json-server) - Nice mock JSON server
- [json api spec](https://jsonapi.org/format/)
- [objection](https://vincit.github.io/objection.js/guide/models.html#examples) for describing the model
- [can-query-logic](https://canjs.com/doc/can-query-logic.html) for filtering, sorting, etc
- [can-local-store](https://canjs.com/doc/can-local-store.html) for a data store.
- [JSONSchema](https://json-schema.org/) - how objection does validation ... can be used for describing properties
- [json-schema-faker](https://github.com/json-schema-faker/json-schema-faker/blob/master/docs/USAGE.md) - Fakes data for a JSON schema
- [ChanceJS](https://chancejs.com/) - Random generator helper.


## Use


