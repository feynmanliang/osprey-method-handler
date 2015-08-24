# Osprey Method Handler

[![NPM version][npm-image]][npm-url]
[![NPM Downloads][downloads-image]][downloads-url]
[![Build status][travis-image]][travis-url]
[![Test coverage][coveralls-image]][coveralls-url]

Middleware for validating requests and responses based on a [RAML method](https://github.com/raml-org/raml-spec/blob/master/raml-0.8.md#methods) object.

## Installation

```
npm install osprey-method-handler --save
```

## Features

* Header validation (ignores undocumented headers)
* Query validation (ignores undocumented parameters)
* Request body validation
  * JSON schemas
  * XML schemas
  * URL-encoded `formParameters` (ignores undocumented parameters)
  * Multipart form data `formParameters` (ignores undocumented parameters)
  * Discards unknown bodies
* Accept content type negotiation (based on defined success response bodies)
* Automatically parsed request bodies
  * JSON (`req.body`)
  * URL-encoded (`req.body`)
  * XML ([`req.xml`](https://github.com/polotek/libxmljs))
  * Form Data (`req.form` using [Busboy](https://github.com/mscdex/busboy), but you need to pipe the request into it - `req.pipe(req.form)`)

**Please note:** Due to the build time of `libxmljs`, it does not come bundled. If you need XML validation, please install `libxmljs` as a dependency of your own project.

## Usage

```js
var express = require('express')
var handler = require('osprey-method-handler')
var app = express()

app.post('/users', handler({
  headers: {},
  responses: {
    '200': {
      body: {
        'application/json': {
          schema: '...',
          example: '...'
        }
      }
    }
  },
  body: {
    'application/json': {
      schema: '...'
    }
  }
}, '/users', 'POST', { /* ... */ }), function (req, res) {
  res.send('success')
})
```

Accepts the RAML schema as the first argument, method and path in subsequent arguments (mostly for debugging) and options as the final argument.

**Options**

* `discardUnknownBodies` Discard undefined request streams (default: `true`)

### Adding JSON schemas

If you are using external JSON schemas with `$ref`, you can add them to the module before you compile the middleware. Use `handler.addJsonSchema(schema, key)` to compile automatically when used.

### Validation Errors

The library intercepts incoming requests and does validation. It will respond with `400`, `406` or `415` error instances from [http-errors](https://github.com/jshttp/http-errors). Validation errors are attached to `400` instances and noted using `ramlValidation = true` and `validationErrors = []` (an array of errors that were found).

The errors object format is:

```ts
interface Error {
  type: 'json' | 'form' | 'headers' | 'query' | 'xml'
  message: string
  keyword: string
  dataPath: string
  data: any
  schema: any
  meta?: Object
}
```

**Please note:** XML validation does not have a way to get the `keyword`, `dataPath`, `data` or `schema`. Instead, it has a `meta` object that contains information from `libxmljs` (`domain`, `code`, `level`, `column`, `line`).

To create custom error messages for your application, you can handle the errors using Express, Connect or any other error callback handler.

## License

MIT license

[npm-image]: https://img.shields.io/npm/v/osprey-method-handler.svg?style=flat
[npm-url]: https://npmjs.org/package/osprey-method-handler
[downloads-image]: https://img.shields.io/npm/dm/osprey-method-handler.svg?style=flat
[downloads-url]: https://npmjs.org/package/osprey-method-handler
[travis-image]: https://img.shields.io/travis/mulesoft-labs/osprey-method-handler.svg?style=flat
[travis-url]: https://travis-ci.org/mulesoft-labs/osprey-method-handler
[coveralls-image]: https://img.shields.io/coveralls/mulesoft-labs/osprey-method-handler.svg?style=flat
[coveralls-url]: https://coveralls.io/r/mulesoft-labs/osprey-method-handler?branch=master
