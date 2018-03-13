# validate

Validate object properties in javascript.

[![npm version](http://img.shields.io/npm/v/validate.svg?style=flat)](https://npmjs.org/package/validate)
[![Build Status](http://img.shields.io/travis/eivindfjeldstad/validate.svg?style=flat)](https://travis-ci.org/eivindfjeldstad/validate)

## Usage

The `.validate()` function returns an array of validation errors.

```js
const Schema = require('validate')

const user = new Schema({
  username: {
    type: 'string',
    required: true,
    length: { min: 3, max: 32 }
  },
  name: {
    type: 'string',
    required: true
  },
  pets: [{
    name: { type: 'string' },
    type: { enum: ['cat', 'dog']}
  }],
  address: {
    street: {
      type: 'string',
      required: true
    },
    city: {
      type: 'string',
      required: true
    }
    zip: {
      type: 'string',
      match: /[0-9]+/,
      required: true
    }
  }
})

const errors = user.validate(obj)
```

Each error has a `.path`, describing the full path of the property that failed validation, and a `.message` describing the error.

```js
errors[0].path //=> 'address.street'
errors[0].message //=> 'address.street is required.'
```

### Custom error messages

You can override the default error messages by passing an object to `Schema#message()`.

```js
const post = new Schema({
  title: { required: true }
})

post.messsage({
  required: (path) => `${path} can not be empty.`
})

const [error] = post.validate({})
assert(error.message = 'title can not be empty.')
```

### Nesting

Objects and arrays can be nested as deep as you want:

```js
const event = new Schema({
  title: { type: 'string' },
  participants: [{
    name: {
      type: 'string'
    },
    email: {
      type: 'string',
      required: true
    },
    things: [{
      name: { type: 'string' },
      amount: { type: 'number' }
    }]
  }]
})
```

Arrays can be defined implicitly, like in the above example, or explicitly:

```js
const post = new Schema({
  title: { type: 'string' },
  content: { type: 'string' },
  keywords: {
    type: 'array',
    each: { type: 'string' }
  }
})
```

Nesting also works with schemas:

```js
const user = new Schema({
  name: { type: 'string' },
  email: { type: 'string' }
})

const post = new Schema({
  title: { type: 'string' },
  content: { type: 'string' },
  author: user
})
```

If you think it should work, it probably works.

### Custom validators

Custom validators can be defined by passing an object with named validators to `.use`:

```js
const hexColor = val => /^#[0-9a-fA-F]$/.test(val)

const car = new Schema({
  color: {
    type: 'string',
    use: { hexColor }
  }
})
```

Define a custom error message for the validator:

```js
car.message({
  hexColor: path => `${path} must be a valid color.`
})
```

### Chainable API

If you want to avoid constructing large objects, you can add paths to a schema by using the chainable API:

```js
const user = new Schema()

user
  .path('username')
    .type('string')
    .required()
  .path('address.zip')
    .type('string')
    .required()
```

Array elements can be defined by using `$` as a placeholder for indices:

```js
const user = new Schema()

user
  .path('pets')
    .type('array')
  .path('pets.$')
    .type('string')
```

This is equivalent to writing

```js
const user = new Schema({ pets: [{ type: 'string' }]})
```

### Typecasting

Values can be automatically typecast before validation.
To enable typecasting, pass an options object to the `Schema` constructor with `typecast` set to `true`.

```js
const user = new Schema(definition, { typecast: true })
```

You can override this setting by passing an option to `.validate()`.

```js
user.validate(obj, { typecast: false })
```

### Property stripping

By default, all values not defined in the schema will be stripped from the object.
Set `.strip = false` on the options object to disable this behavior.

## API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

#### Table of Contents

-   [Property](#property)
    -   [schema](#schema)
    -   [use](#use)
    -   [required](#required)
    -   [type](#type)
    -   [length](#length)
    -   [enum](#enum)
    -   [match](#match)
    -   [each](#each)
    -   [path](#path)
    -   [typecast](#typecast)
    -   [validate](#validate)
-   [Schema](#schema-1)
    -   [path](#path-1)
    -   [validate](#validate-1)
    -   [assert](#assert)
    -   [message](#message)

### Property

A property instance gets returned whenever you call `schema.path()`.
Properties are also created internally when an object is passed to the Schema constructor.

**Parameters**

-   `name` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** the name of the property
-   `schema` **[Schema](#schema)** parent schema

#### schema

Mount given `schema` on current path.

**Parameters**

-   `schema` **[Schema](#schema)** the schema to mount

**Examples**

```javascript
const user = new Schema({ email: 'string' });
prop.schema(user);
```

Returns **[Property](#property)** 

#### use

Validate using named functions from the given object.
Error messages can be defined by providing an object with
named error messages/generators to `schema.message()`

The message generator receives the value being validated,
the object it belongs to and any additional arguments.

**Parameters**

-   `fns` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** object with named validation functions to call

**Examples**

```javascript
const schema = new Schema()
const prop = schema.path('some.path')

schema.message({
  binary: (path, ctx) => `${path} must be binary.`,
  bits: (path, ctx, bits) => `${path} must be ${bits}-bit`
})

prop.use({
  binary: (val, ctx) => /^[01]+$/i.test(val),
  bits: [(val, ctx, bits) => val.length == bits, 32]
})
```

Returns **[Property](#property)** 

#### required

Registers a validator that checks for presence.

**Parameters**

-   `bool` **[Boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** `true` if required, `false` otherwise (optional, default `true`)

**Examples**

```javascript
prop.required()
```

Returns **[Property](#property)** 

#### type

Registers a validator that checks if a value is of a given `type`

**Parameters**

-   `type` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** type to check for

**Examples**

```javascript
prop.type('string')
```

Returns **[Property](#property)** 

#### length

Registers a validator that checks length.

**Parameters**

-   `rules` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** object with `.min` and `.max` properties
    -   `rules.min` **[Number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** minimum length
    -   `rules.max` **[Number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** maximum length

**Examples**

```javascript
prop.length({ min: 8, max: 255 })
```

Returns **[Property](#property)** 

#### enum

Registers a validator for enums.

**Parameters**

-   `enums`  
-   `rules` **[Array](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array)** allowed values

**Examples**

```javascript
prop.enum(['cat', 'dog'])
```

Returns **[Property](#property)** 

#### match

Registers a validator that checks if a value matches given `regexp`.

**Parameters**

-   `regexp` **[RegExp](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/RegExp)** regular expression to match

**Examples**

```javascript
prop.match(/some\sregular\sexpression/)
```

Returns **[Property](#property)** 

#### each

Registers a validator that checks each value in an array against given `rules`.

**Parameters**

-   `rules` **([Array](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array) \| [Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object) \| [Schema](#schema) \| [Property](#property))** rules to use

**Examples**

```javascript
prop.each({ type: 'string' })
prop.each([{ type: 'number' }])
prop.each({ things: [{ type: 'string' }]})
prop.each(schema)
```

Returns **[Property](#property)** 

#### path

Proxy method for schema path. Makes chaining properties together easier.

**Parameters**

-   `args` **...any** 

**Examples**

```javascript
schema
  .path('name')
    .type('string')
    .required()
  .path('email')
    .type('string')
    .required()
```

#### typecast

Typecast given `value`

**Parameters**

-   `value` **Mixed** value to typecast

**Examples**

```javascript
prop.type('string')
prop.typecast(123) // => '123'
```

Returns **Mixed** 

#### validate

Validate given `value`

**Parameters**

-   `value` **Mixed** value to validate
-   `ctx` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** the object containing the value
-   `path` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)?** path of the value being validated (optional, default `this.name`)

**Examples**

```javascript
prop.type('number')
assert(prop.validate(2) == false)
assert(prop.validate('hello world') instanceof Error)
```

Returns **([Error](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Error) \| [Boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean))** 

### Schema

A Schema defines the structure that objects should be validated against.

**Parameters**

-   `obj` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** schema definition (optional, default `{}`)
-   `opts` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** options (optional, default `{}`)
    -   `opts.typecast` **[Boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** typecast values before validation (optional, default `false`)
    -   `opts.strip` **[Boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** strip properties not defined in the schema (optional, default `true`)

**Examples**

```javascript
const post = new Schema({
  title: {
    type: 'string',
    required: true,
    length: { min: 1, max: 255 }
  },
  content: {
    type: 'string',
    required: true
  },
  published: {
    type: 'date',
    required: true
  },
  keywords: [{ type: 'string' }]
})
```

```javascript
const author = new Schema({
  name: {
    type: 'string',
    required: true
  },
  email: {
    type: 'string',
    required: true
  },
  posts: [post]
})
```

#### path

Create or update `path` with given `rules`.

**Parameters**

-   `path` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** full path using dot-notation
-   `rules` **([Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object) \| [Array](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array) \| [String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Schema](#schema) \| [Property](#property))?** rules to apply

**Examples**

```javascript
const schema = new Schema()
schema.path('name.first', { type: 'string' })
schema.path('name.last').type('string').required()
```

Returns **[Property](#property)** 

#### validate

Validate given `obj`.

**Parameters**

-   `obj` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** the object to validate
-   `opts` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** options, see [Schema](#schema-1) (optional, default `{}`)

**Examples**

```javascript
const schema = new Schema({ name: { required: true }})
const errors = schema.validate({})
assert(errors.length == 1)
assert(errors[0].message == 'name is required')
assert(errors[0].path == 'name')
```

Returns **[Array](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array)** 

#### assert

Assert that given `obj` is valid.

**Parameters**

-   `obj` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** 
-   `opts` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 

**Examples**

```javascript
const schema = new Schema({ name: 'string' })
schema.assert({ name: 1 }) // => Throws an error
```

#### message

Override default error messages.

**Parameters**

-   `obj` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** an object containing new messages

Returns **[Schema](#schema)** 

## Licence

MIT
