# Graphqlizejs

[![Build Status](https://travis-ci.com/stvkoch/graphqlizejs.svg?branch=master)](https://travis-ci.com/stvkoch/graphqlizejs)
[![NPM](https://img.shields.io/npm/v/graphqlizejs.svg)](https://www.npmjs.com/package/graphqlizejs) [![JavaScript Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](https://standardjs.com)

Graphqlizejs automatically generate data types and resolvers for graphql servers from your sequelizejs models!

> _it's awesome... really awesome!_

You define your models and everything it's available!

- Inputs
  - inputs wheres
  - inputs operators
  - inputs mutations
- Types
  - types models
  - types associations
- Queries
  - queries models
  - queries counters
- Mutations
  - create mutation
  - update mutation
  - delete mutation
- Subscriptions

  - create
  - update
  - delete

## Install

```
git clone https://github.com/stvkoch/graphqlize.git
cd graphqlize
npm install# or npm
npm run example # or npm
# open url http://localhost:4000/graphql
# can access complete generate schema in http://localhost:4000
```

### It's awesome because SequelizeJs it's very powerful!

Do you know about sequelizejs?

- No? Then check out the site http://docs.sequelizejs.com/

You can do a lot of things with sequelizejs and Graphqlizejs automagic generate graphql datatype and resolvers from your models

### No patience?

OK, let's check the demo?

#### Graphql

graphql> https://graphqlize.herokuapp.com/graphql

schema> https://graphqlize.herokuapp.com/

### Examples of queries that you can play

```
{
  # IN operator
  queryAsInOp: services(where: {id: { in: ["3", "7", "12"] }}) {
    id
    name
    price
  }
  # operator combination AND
  countQueryAsAndOp: _servicesCount(where: {price: { gt: 150, lt: 200 }})
  queryAsAndOp: services(where: {price: { gt: 150, lt: 200 }}) {
    id
    name
    price
  }
  # you can also use conditions inside of yours associations
  country(where: {id: { eq: "PT" }}) {
    id
    name
    _servicesCount(where: {price: { gt: 150, lt: 200 }})
    services(where: {price: { gt: 150, lt: 200 }}) {
      id
      name
      price
    }
  }
  # we don't support directly OR, but in graphql you request more that one list
  expensiveServices: services(where: {price: { gt: 980 }}) {
    id
    name
    price
  }
  #OR
  cheapServices: services(where: {price: { lt: 20 }}) {
    id
    name
    price
  }
}
```

## Go to by examples

Let's imagine that we have follow models (example folder):

```
// models/category.js file with all graphqlizejs options available
export default (sequelize, DataTypes) => {
  const Category = sequelize.define(
    "category",
    {
      id: {
        type: DataTypes.INTEGER,
        primaryKey: true,
        autoIncrement: true,

         // option enable/disable graphql prop [default: false]
        gqIgnore: false
      },
      name: {
        type: DataTypes.STRING,

        // option enable/disable graphql prop [default: false]
        gqIgnore: false
      }
    },
    {
      freezeTableName: true,

      // optional different name for graphql
      gqName: 'category',

      // option enable/disable generate all grapqhl queries/mutations for this model [default: false]
      gqIgnore: false,

      // option enable/disable generate grapqhl query for this model [default: true]
      gqQuery: true,

      // option enable/disable generate grapqhl query count  for this model [default: true]
      gqQueryCount: true,

      // option enable/disable generate grapqhl mutation create for this model [default: true]
      gqCreate: true,

      // option enable/disable generate grapqhl mutation update for this model [default: true]
      gqUpdate: true,

      // option enable/disable generate grapqhl mutation delete for this model [default: true]
      gqDelete: true,

      // enable/disable generate grapqhl subscriptions of follow operations [default: false]
      gqSubscriptionCreate: false,
      gqSubscriptionUpdate: false,
      gqSubscriptionDelete: false,

      // set middlewares for each operations to restrict or changes requests or results [default: defaultMiddleware]
      gqMiddleware: {
        query: defaultMiddleware,
        queryCount: defaultMiddleware,
        create: defaultMiddleware,
        update: defaultMiddleware,
        delete: defaultMiddleware,
        subscribe: defaultMiddleware
      }
    }
  );
  Category.associate = models => {
    Category.hasOne(models.category, { as: "parent", foreignKey: "parentId" });
    Category.hasMany(models.service);
  };
  return Category;
};

// models/service.js file
export default (sequelize, DataTypes) => {
  const Service = sequelize.define(
    "service",
    {
      id: {
        type: DataTypes.INTEGER,
        primaryKey: true,
        autoIncrement: true
      },
      name: DataTypes.STRING,
      price: DataTypes.DECIMAL(10, 2)
    },
    {
      freezeTableName: true
    }
  );
  Service.associate = models => {
    Service.belongsTo(models.category);
  };
  return Service;
};

```

### Generating schema and resolvers

```
//...
import { schema, resolvers } from "graphqlizejs";
import db from './models';

const schemaGenerated = schema(db.sequelize);
const resolversGenerated = resolvers(db.sequelize);


const server = new ApolloServer({
  typeDefs: gql(schemaGenerated),
  resolvers: resolversGenerated,
  context: { db },
  introspection: true,
  playground: true
});
....

```

### Simple Queries

```
query GetCategory {
  categories {
    id
    name
  }
}
```

### Simple Queries With Conditions

To add conditions in your queries you should use \_inputWhere input generated for each model. This kind inputs will hold field model and input operators for the type defined.

```
query GetCategory($where: _inputWhereCategory) {
  categories(where: $where) {
    id
    name
  }
}

variable:
{
  "where": {"name": {"like": "%u%"}}
}
```

> To avoid collisions names, graphqlizejs generate input names and counter association fields starting with an underscore character. Example: \_associationsCount, \_inputs.

### Simple Count Queries

Each associate field defined in your model has your counter field called by _underscore_ + _association name_ + _Count_ word.

In the example, below look for \_servicesCount.

```
query GetCategory {
  categories {
    id
    name
    totalServices: _servicesCount
    serviceCountStartU: _servicesCount(where: {name: {like:"u%"}})
    services(where: {name:{like:"u%"}}) {
      id
      name
    }
  }
}
```

### Association Queries

Retrieve all services by category:

```
query GetCategory {
  categories {
    id
    name
    services {
      id
      name
    }
  }
}
```

You also can filter your association's data as you did "Simple Queries With Conditions" example.

### Association Count Queries

Each query also has your counter association field follow the same name definition: _underscore_ + model name* + \_Count* word.

```
query GetCategoryCount {
  renameCategoryCount: _categoriesCount
  _categoriesCount(where: {name:{like:"%u%"}})
}
```

## Inputs Where

For each model, graphqlize will create a graphql input with all available model fields to be used in your conditions.
You will see that, the fields defined in your input not use the model type, instead, it's used the type \_input _Type_ Operator\_ that will hold all operators supported by the model type specified.

For instance _country_ model, graphqlize will generate the \__inputWhereCountry_.

## Inputs Operators

Sequelize ORM supports several query operators to allow filtered your data using findAll method. For this reason was create \_inputTypeOperator.

Most of the types support the following operators:

```
eq
ne
gte
gt
lte
lt
not
is
in
notIn
between
notBetween
```

String type support the follow additional operators:

```
like
notLike
iLike
notILike
startsWith
endsWith
substring
regexp
notRegexp
iRegexp
notIRegexp
overlap
contains
contained
adjacent
strictLeft
strictRight
```

## Inputs Create and Update

To able to mutate your data you will need to hold your data inside of the input mutation type. Graphqlizejs will generate the \_inputCreate and \_inputUpdate for each model and _through_ models.

### Input Create

Example of country model:

```
type _inputCreateCountry {
  name: String!
  callCode: String
  currencyCode: String
  createdAt: String
  updatedAt: String
}
```

> Note that graphqlizejs didn't create the input with the primary keys. If you want to create or update your primary keys, enable graphqlizejs to create the input with the primary keys setting the model options with:

```
gqInputCreateWithPrimaryKeys: true
```

### Input Update

```
type _inputUpdateCountry {
  name: String!
  callCode: String
  currencyCode: String
  createdAt: String
  updatedAt: String
}
```

Same way, if you want to enable update primary keys set the model option:

```
gqInputUpdateWithPrimaryKeys: true
```

### Pagination handlers inside of input where type:

- \_limit: Int
- \_offset: Int
- \_orderBy: Array[Array]
- \_group: Array

### orderBy

_\_orderBy_ argument accept a array with fieldName and direction. Ex: ['username', 'DESC']

### Subscriptions

You can subscribe changes using graphqlizejs subscriptions generated for each mutation by setting:

```
gqSubscriptionCreate: true,
gqSubscriptionUpdate: true,
gqSubscriptionDelete: true
```

#### Example Subscription

```
subscription {
  updateCountry(where: { id: { eq: "PT" } }) {
    id
    name
    serviceCount: _servicesCount
  }
}
```

```
mutation {
  updateCountry(
    where: { id: { eq: "PT" } },
    input: { name: "Purtugaal" }
  ) {
    id
  }
}
```

## Middlewares Resolvers

Middleware is the way to add some control over the model resolvers. You can add middlewares receiving the next resolver and returning the function with the same signature found in apollo server resolvers (root/parent, args, context, info) arguments.

Defining middleware:

```
function requireUser(nextResolver) {
  return (parent, args, context, info) => {
    if (!context.user)
      throw new AuthenticationError('Required valid authentication.');

    // you always should call the next resolver if you want to keep flow the resolvers.
    return nextResolver(parent, args, context, info);
  }
}
```

Using middleware in your models:

```
import flow from 'lodash.flow';
import {requireUser, onlyOwnData} from '../models/middlewares';

// models/service.js file
export default (sequelize, DataTypes) => {
  const Service = sequelize.define(
    "service",
    {
      id: {
        type: DataTypes.INTEGER,
        primaryKey: true,
        autoIncrement: true
      },
      name: DataTypes.STRING,
      price: DataTypes.DECIMAL(10, 2)
    },
    {
      freezeTableName: true,
      gqMiddleware: {
        query: requireUser,
        queryCount: requireUser,
        create: flow([requireUser, onlyOwnData]),
        update: flow([requireUser, onlyOwnData]),
        destroy: flow([requireUser, onlyOwnData])
      }
    }
  );
  Service.associate = models => {
    Service.belongsTo(models.category);
  };
  return Service;
};

```

## Complete example

https://github.com/stvkoch/example-graphqlizejs
