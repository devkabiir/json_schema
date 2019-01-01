# JSON Schema (Forked)

  A *platform agnostic* (dart:html or dart:io) Dart library for validating JSON instances against JSON Schemas (multi-version support with latest of Draft 6).

## Forked changes

- Does not support `dart < 2.0.0`
- Supports dart 2.0.0
- Uses uri instead of uri_template
- Package name changed to json_schema_dart2 (to allow publishing on pub)

![Build Status](https://travis-ci.com/devkabiir/json_schema.svg)

## How To Create and Validate Against a Schema
  
### Synchronous Creation - Self Contained
  
The simplest way to create a schema is to pass JSON data directly to `JsonSchema.createSchema` with a JSON `String`, or decoded JSON via Dart `Map` or `bool`. 

After creating any schema, JSON instances can be validated by calling `.validate(instance)` on that schema. By default, instances are expected to be pre-parsed JSON as native dart primitives (`Map`, `List`, `String`, `bool`, `num`, `int`). You can also optionally parse at validation time by passing in a string and setting `parseJson`: `schema.validate('{ "name": "any JSON object"}', parseJson: true)`.
    
  > Note: Creating JsonSchemas synchronously implies access to all $refs within the root schema. If you don't have access to all this data at the time of the construction, see "Asynchronous Creation" examples below.


#### Example

A schema can be created with a Map that is either hand-crafted, referenced from a JSON file, or *previously* fetched from the network or file system.

```dart
import 'package:json_schema_dart2/json_schema.dart';

main() {
  /// Define schema in a Dart [Map] or use a JSON [String].
  final mustBeIntegerSchemaMap = {"type": "integer"};

  // Create some examples to validate against the schema.
  final n = 3;
  final decimals = 3.14;
  final str = 'hi';

  // Construct the schema from the schema map or JSON string.
  final schema = JsonSchema.createSchema(mustBeIntegerSchemaMap);

  print('$n => ${schema.validate(n)}'); // true
  print('$decimals => ${schema.validate(decimals)}'); // false
  print('$str => ${schema.validate(str)}'); // false
}
```

### Synchronous Creation, Local Ref Cache

If you want to create `JsonSchema`s synchronously, and you have $refs that cannot be resolved within the root schema, but you have a cache of those $ref'd schemas locally, you can write a `RefProvider` to get them during schema evaluation.

#### Example

```dart 
import 'package:json_schema_dart2/json_schema.dart';
import 'package:dart2_constant/convert.dart';

main() {
  final referencedSchema = {
    r"$id": "https://example.com/geographical-location.schema.json",
    r"$schema": "http://json-schema.org/draft-06/schema#",
    "title": "Longitude and Latitude",
    "description": "A geographical coordinate on a planet (most commonly Earth).",
    "required": ["latitude", "longitude"],
    "type": "object",
    "properties": {
      "name": {"type": "string"},
      "latitude": {"type": "number", "minimum": -90, "maximum": 90},
      "longitude": {"type": "number", "minimum": -180, "maximum": 180}
    }
  };

  final RefProvider refProvider = (String ref) {
    final Map references = {
      'https://example.com/geographical-location.schema.json': JsonSchema.createSchema(referencedSchema),
    };

    if (references.containsKey(ref)) {
      return references[ref];
    }

    return null;
  };

  final schema = JsonSchema.createSchema({
    'type': 'array',
    'items': {r'$ref': 'https://example.com/geographical-location.schema.json'}
  }, refProvider: refProvider);

  final workivaLocations = [
    {
      'name': 'Ames',
      'latitude': 41.9956731,
      'longitude': -93.6403663,
    },
    {
      'name': 'Scottsdale',
      'latitude': 33.4634707,
      'longitude': -111.9266617,
    }
  ];

  final badLocations = [
    {
      'name': 'Bad Badlands',
      'latitude': 181,
      'longitude': 92,
    },
    {
      'name': 'Nowhereville',
      'latitude': -2000,
      'longitude': 7836,
    }
  ];

  print('${json.encode(workivaLocations)} => ${schema.validate(workivaLocations)}');
  print('${json.encode(badLocations)} => ${schema.validate(badLocations)}');
}
```

### Asynchronous Creation, Remote HTTP Refs

If you have schemas that have nested $refs that are HTTP URIs that are publicly accessible, you can use `Future<JsonSchema> JsonSchema.createSchemaAsync` and the references will be fetched as needed during evaluation. You can also use `JsonSchema.createSchemaFromUrl` if you want to fetch the root schema remotely as well (see next example).

#### Example

```dart
import 'dart:io';

import 'package:json_schema_dart2/json_schema.dart';

// For VM:
import 'package:json_schema_dart2/vm.dart';

// For Browser:
// import 'package:json_schema_dart2/browser.dart';

main() async {
  // For VM:
  configureJsonSchemaForVm();

  // For Browser:
  // configureJsonSchemaForBrowser();

  // Schema Defined as a JSON String
  final schema = await JsonSchema.createSchemaAsync(r'''
  {
    "type": "array",
    "items": {
      "$ref": "https://raw.githubusercontent.com/json-schema-org/JSON-Schema-Test-Suite/master/remotes/integer.json"
    }
  }
  ''');

  // Create some examples to validate against the schema.
  final numbersArray = [1, 2, 3];
  final decimalsArray = [3.14, 1.2, 5.8];
  final strArray = ['hello', 'world'];

  print('$numbersArray => ${schema.validate(numbersArray)}'); // true
  print('$decimalsArray => ${schema.validate(decimalsArray)}'); // false
  print('$strArray => ${schema.validate(strArray)}'); // false

  // Exit the process cleanly (VM Only).
  exit(0);
}
```

### Asynchronous Creation, From URL or File

You can also create a schema directly from a publicly accessible URL, like so:

#### Example 1 - URL

```dart
import 'dart:io';

import 'package:json_schema_dart2/json_schema.dart';

// For VM:
import 'package:json_schema_dart2/vm.dart';

// For Browser:
// import 'package:json_schema_dart2/browser.dart';

main() async {
  // For VM:
  configureJsonSchemaForVm();

  // For Browser:
  // configureJsonSchemaForBrowser();

  final url = "https://raw.githubusercontent.com/json-schema-org/JSON-Schema-Test-Suite/master/remotes/integer.json";

  final schema = await JsonSchema.createSchemaFromUrl(url);

  // Create some examples to validate against the schema.
  final n = 3;
  final decimals = 3.14;
  final str = 'hi';

  print('$n => ${schema.validate(n)}'); // true
  print('$decimals => ${schema.validate(decimals)}'); // false
  print('$str => ${schema.validate(str)}'); // false

  // Exit the process cleanly (VM Only).
  exit(0);
}
```

#### Example 2 - File

```dart
import 'dart:io';

import 'package:json_schema_dart2/json_schema.dart';

// For VM:
import 'package:json_schema_dart2/vm.dart';

// For Browser:
// import 'package:json_schema_dart2/browser.dart';

main() async {
  // For VM:
  configureJsonSchemaForVm();

  // For Browser:
  // configureJsonSchemaForBrowser();

  final file = "example/readme/asynchronous_creation/geo.schema.json";

  final schema = await JsonSchema.createSchemaFromUrl(file);

  // Create some examples to validate against the schema.
  final workivaAmes = {
    'latitude': 41.9956731,
    'longitude': -93.6403663,
  };

  final nowhereville = {
    'latitude': -2000,
    'longitude': 7836,
  };

  print('$workivaAmes => ${schema.validate(workivaAmes)}'); // true
  print('$nowhereville => ${schema.validate(nowhereville)}'); // false

  // Exit the process cleanly (VM Only).
  exit(0);
}
```

### Asynchronous Creation, with custom remote $refs:

If you have nested $refs that are either non-HTTP URIs or non-publicly-accessible HTTP $refs, you can supply an `RefProviderAsync` to `createSchemaAsync`, and perform any custom logic you need.

#### Example

```dart
import 'dart:io';
import 'dart:async';
import 'package:dart2_constant/convert.dart';

import 'package:json_schema_dart2/json_schema.dart';

// For VM:
import 'package:json_schema_dart2/vm.dart';

// For Browser:
// import 'package:json_schema_dart2/browser.dart';

main() async {
  // For VM:
  configureJsonSchemaForVm();

  // For Browser:
  // configureJsonSchemaForBrowser();

  final referencedSchema = {
    r"$id": "https://example.com/geographical-location.schema.json",
    r"$schema": "http://json-schema.org/draft-06/schema#",
    "title": "Longitude and Latitude",
    "description": "A geographical coordinate on a planet (most commonly Earth).",
    "required": ["latitude", "longitude"],
    "type": "object",
    "properties": {
      "name": {"type": "string"},
      "latitude": {"type": "number", "minimum": -90, "maximum": 90},
      "longitude": {"type": "number", "minimum": -180, "maximum": 180}
    }
  };

  final RefProviderAsync refProvider = (String ref) async {
    final Map references = {
      'https://example.com/geographical-location.schema.json': JsonSchema.createSchema(referencedSchema),
    };

    if (references.containsKey(ref)) {
      // Silly example that adds a 1 second delay.
      // In practice, you could make any service call here,
      // parse the results into a schema, and return.
      await new Future.delayed(new Duration(seconds: 1));
      return references[ref];
    }

    // Fall back to default URL $ref behavior
    return await JsonSchema.createSchemaFromUrl(ref);
  };

  final schema = await JsonSchema.createSchemaAsync({
    'type': 'array',
    'items': {r'$ref': 'https://example.com/geographical-location.schema.json'}
  }, refProvider: refProvider);

  final workivaLocations = [
    {
      'name': 'Ames',
      'latitude': 41.9956731,
      'longitude': -93.6403663,
    },
    {
      'name': 'Scottsdale',
      'latitude': 33.4634707,
      'longitude': -111.9266617,
    }
  ];

  final badLocations = [
    {
      'name': 'Bad Badlands',
      'latitude': 181,
      'longitude': 92,
    },
    {
      'name': 'Nowhereville',
      'latitude': -2000,
      'longitude': 7836,
    }
  ];

  print('${json.encode(workivaLocations)} => ${schema.validate(workivaLocations)}');
  print('${json.encode(badLocations)} => ${schema.validate(badLocations)}');

  exit(0);
}
```

## How To Use Schema Information

  Schema information can be used for validation; but it can also be a valuable source of information about the structure of data. The `JsonSchema` class fully parses the schema first, which itself must be valid on all paths within the schema. Accessors are provided for all specified keywords of the JSON Schema specification associated with a schema, so tools can use it to create rich views of the data, like forms or diagrams.

  One example use is the *deprecated* _schemadot_ program included in the _bin_
  folder which takes schema as input and outputs a _Graphviz_ _dot_
  file, providing a picture of the schema. This does not provide all
  information of the schema, and is a work in progress - but it can be
  useful to *see* what a schema is.

  For example, the grades_schema.json is:

    {
        "$schema": "http://json-schema.org/draft-04/schema#",
        "title" : "Grade Tracker",
        "type" : "object",
        "additionalProperties" : false,
        "properties" : {
    	"semesters" : {
    	    "type" : "array",
    	    "items" : {
                    "type" : "object",
                    "additionalProperties" : false,
                    "properties" : {
                        "semester": { "type" : "integer" },
                        "grades" : {
                            "type" : "array",
                            "items" : {
                                "type" : "object",
                                "additionalProperties" : false,
                                "required" : [ "date", "type", "grade", "std" ],
                                "properties" : {
                                    "date" : { "type" : "string"},
                                    "type" : { "enum" : [ "homework", "quiz", "test", "final_exam" ] },
                                    "grade" : { "type" : "number"},
                                    "std" : { 
                                        "oneOf" : [ 
                                            {"type" : "number"}, 
                                            {"type" : "null"}
                                        ] 
                                    },
                                    "avg" : { 
                                        "oneOf" : [ 
                                            {"type" : "number"}, 
                                            {"type" : "null"}
                                        ] 
                                    }
                                }
                            }
                        }
                    }
                }
      	    }
        }
    }

  And the generated image is:

  ![Grades!](https://raw.github.com/patefacio/json_schema/master/example/from_url/grades_schema.png)  

  For more detailed image open link:
  <a href="https://raw.github.com/patefacio/json_schema/master/example/from_url/grades_schema.png"
  target="_blank">Grade example schema diagram</a>
