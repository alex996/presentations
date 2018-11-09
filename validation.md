# Validation

- enforce business logic
- preserve data integrity
- avert malicious inputs

## [Directives](https://graphql.org/learn/queries/#directives)

Ex: [`graphql-constraint-directive`](https://github.com/confuser/graphql-constraint-directive) (featured in [Apollo Blog](https://blog.apollographql.com/graphql-validation-using-directives-4908fd5c1055))

**Pros**

- GraphQL centric and versatile

**Cons**

- Does _not_ allow for `@constraint` on query args
- No compat with `apollo-server` v2, quirks in v1

## Resolvers

Ex: validating `args.id` for an `ObjectID`

**Pros**

- Simple and intuitive

**Cons**

- Too repetitive, not DRY
- Clutters up, doesn't scale

## Utils

Ex: pure funcs using [`validator.js`](https://github.com/chriso/validator.js/)

**Pros**

- Simple enough, composable, can scale

**Cons**

- Reinventing the wheel
- Code to maintain

## ODM/ORM

Ex: [built-in validators](https://mongoosejs.com/docs/validation.html#built-in-validators) in Mongoose

**Pros**

- Common checks
- Extensible
  - define custom validation logic
  - customize error messages

**Cons**

- Too many gotchas, e.g.
  - `unique` is not a validator, but a unique index
  - `update*` validators are off by default
  - many `pre`/`post` hooks don't fire for `update*`
- Hard to validate a subset of fields
  - e.g. `findById` then `save` will validate all fields
- Models grow out of size

## Object Schema

Ex:
- [ajv](https://www.npmjs.com/package/ajv)
- [schm](https://www.npmjs.com/package/schm)
- [joi](https://www.npmjs.com/package/joi)
  - [yup](https://www.npmjs.com/package/yup) for browsers (used in [formik](https://www.npmjs.com/package/formik))

**Pros**

- Very expressive & readable
- DRY, doesn't clutter up
- Extensible & customizable
  - custom validators, plugins, messages

**Cons**

- Cryptic error messages (though customizable)
