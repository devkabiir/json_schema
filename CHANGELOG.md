## 3.0.0

* Does not support `dart < 2.0.0`
* Supports dart 2.0.0
* Uses uri instead of uri_template
* Package name changed to json_schema_dart2 (to allow publishing on pub)

## 2.0.0

* json_schema is no longer bound to dart:io and works in the browser!
* Full JSON Schema draft6 compatibility
* Much better $ref resolution, including deep nesting of $refs
* More typed keyword getters for draft6 like `examples`
* Syncronous schema evaluation by default 
* Optional async evaluation and fetching with `createSchemaAsync`
* Automatic parsing of JSON strings passed to `createSchema` and `createSchemaAsync`
* Ability to do custom resolution of $refs with `RefProvider` and `RefProviderAsync`
* Optional parsing of JSON strings passed to `validate` with `parseJson = true`
* Dart 2.0 compatibility
* Many small changes to make things more in line with modern dart.
* Please see the [migration guide](./MIGRATION.md) for additional info.

## 1.0.8

* Code cleanup
* Strong mode
* Switch build tools to dart_dev

## 1.0.7

* Update dependency constraint on the `args` package.

## 1.0.3

* Add a dependency on the `args` package.

## 1.0.2

* Add a dependency on the `logging` package.
