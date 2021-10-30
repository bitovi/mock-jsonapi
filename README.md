# mock-jsonapi

## Goals 

1. Easily generate a mock JSONAPI that handles relationships, filtering, sorting, etc
2. Make it easy to work with backend teams.
3. Allow customization
4. Work with typescript

## Inspiration


- [JSONServer](https://github.com/typicode/json-server) - Nice mock JSON server
- [json api spec](https://jsonapi.org/format/)
- [objection](https://vincit.github.io/objection.js/guide/models.html#examples) for describing the model
- [can-query-logic](https://canjs.com/doc/can-query-logic.html) for filtering, sorting, etc
- [can-local-store](https://canjs.com/doc/can-local-store.html) for a data store.
- [JSONSchema](https://json-schema.org/) - how objection does validation ... can be used for describing properties
- [json-schema-faker](https://github.com/json-schema-faker/json-schema-faker/blob/master/docs/USAGE.md) - Fakes data for a JSON schema
- [ChanceJS](https://chancejs.com/) - Random generator helper.


## Questions

- Could this auto-generate OpenAPI?
- Migrations are the source of truth for server-side Objection field data.  Can this be exported and provided back to the client Model?
- How do you define rich types like UUID?

## Use

The __tldr__ use case is as follows:

```js
// 1. Define your models:
const definitions = {
  Employee: {
    name: "Employee", idColumn: 'id',
    schema: {
        type: "object",
        required: ['fullName'],
        properties: {
          id: {type: "string", format: "uuid"},
          startDate: {type: "string", format: "date-time"}
        }
      }
    },
    relationMappings: {
      skills: { relation: "hasManyRelation", modelClass: {$ref: "#/definitions/Skill"}, through: "#/definitions/EmployeeSkill" },
      employeeRoles: { /* ... */ },
    }
  },
  Skill: {/* ... */},
  EmployeeRoles: {/* ... */}
}

// 2. Generate your raw data
const data = generate( definitions, {
  Employee: {from: 0, to: 100, properties: {name: faker.name.findName} },
  Skill: [{ name: "JS" }, {name: "UX"}, {name: "Backend"}],
  EmployeeSkill: {for: "Employee", from: 0, to: 10}
});

// 3. Conditionally load data into your data Store
const dataStore = new LocalDataStore(definitions, "some-key");
if(dataStore.isEmpty() ){
  dataStore.import( data );
}

// 4. Connect the dataStore to mock service worker
const handlers = mswJSONApi(dataStore, {
  Employee: "/employees",
  Skill: "/skills",
});
setupServer( handlers );

// 5. Make beautiful JSONAPI requests
fetch("/employees?include=skills")
```



### Step 1: Define your Models

```js
// in /models/employees.js

import { Model } from "mock-jsonapi";

class Employee extends Model {
  static get tableName() {
    return 'employees';
  }
  static get idColumn(){
    return 'id';
  }
  static get jsonSchema(){
    return {
      type: "object",
      required: ['fullName'],
      properties: {
        id: {type: "string", format: "uuid"},
        startDate: {type: "string", format: "date-time"}
      }
    }
  }
  static get relationMappings () {
    return {
      employeeRoles: { relation: Model.HasManyRelation, modelClass: EmployeeRoles },
      skills: { ... }
    }
  }
}
```

Notes:

- `Model` could be from something else that we could make directly produce an Objection model.

#### Alternate

```js
static Employee = {
  name: "Employee"
  idColumn: 'id',
  schema: {
      type: "object",
      required: ['fullName'],
      properties: { // too bad we can't call this attributes :-(
        id: {type: "string", format: "uuid"},
        startDate: {type: "string", format: "date-time"}
      }
    }
  },
  relationMappings: {
    employeeRoles: { relation: "hasManyRelation", modelClass: {$ref: "#/definitions/Role"} },
    skills: { ... }
  }
}
```

Notes:
- What about hooks?
- How would docs on a property get generated? Where would this data go?


### Step 2: Generate Data

```js
import { generate } from "mock-jsonapi";
import faker from "faker";

const definitions = {
  Employee, Skill, EmployeeSkill
}

const data = generate( definitions, {
  Employee: {from: 0, to: 100, properties: {name: faker.name.findName},
  Skill: [{ name: "JS" }, {name: "UX"}, {name: "Backend"}],
  EmployeeSkill: {for: "Employee", from: 0, to: 10}
});

data === {
  Employee: [{},{},{}],
  Skill: [{ name: "JS", id: 001 }, {name: "UX", id: 002}, {name: "Backend", id: 003}],
  EmployeeSkill: [ ... ]
}
```

### Step 3: Put data in storage

```
import { LocalDataStore } from "mock-jsonapi";

const dataStore = new LocalDataStore(definitions, "some-key");
if(dataStore.isEmpty() ){
  dataStore.import( data );
}
```


### Step 4: Hookup JSONAPI to the dataStore


```
import { setupServer } from 'msw/node'; // is node here right?!?!
import { mswJSONApi } from "mock-jsonapi"


const handlers = mswJSONApi(dataStore, {
  Employee: "/employee",
  Skill: "/skills",
});
setupServer( handlers );
```
