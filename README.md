# typescript-to-gdscript

Convert TypeScript type definitions into concrete models in GDScript for godot

This utility enables a pipeline where TypeScript files containing interfaces can be transformed into gdscript classes.

The classes can then ingest data sent across http or WebSocket requests with some minimal validation against the TypeScript interface.

This enables strongly typed communication between TypeScript and the Godot game engine.

## Usage

Usage:

```
typescript-to-gdscript [--debug-print] templatefile.gd.tmpl outputdir input1.ts [input2.ts...]
```

Reads all interfaces from input.ts files exports them to `outputdir/[InterfaceName].gd` files.
Uses templatefile.gd.tmpl as the template.

## Directives

Comments can contain directives to help out with conversion

* `@typescript-to-gdscript-type: int|float|String`: Forces the type of a property.
* `@typescript-to-gdscript-skip`: This type will not be imported, and will be completely ignored by this program.
* `@typescript-to-gdscript-gd-impl`: This type will be imported, but the gdscript file will not be generated.
    * This is useful for interface union types that would generate one type or another based on the kind property for example.
    * See [any-kind.ts](./test-fixtures/any-kind.ts) for an example.

## Templates

Templates use the [tinytemplate syntax](https://docs.rs/tinytemplate/latest/tinytemplate/syntax/index.html). See the [example template](./gdscript-model.gd.tmpl).

## Output

The example template outputs a gdscript 'model' class which has a constructor that accepts an optional `Array` or `Dictionary` object, as well as an `update` method which allows the model to be updated in place.

Properties of the incoming interface are mapped to additional 'model' class files and imported automatically with `preload`.

### Example Output

```Python
extends Reference
# Generated by typescript-to-gdscript. Do not edit by hand!
# You can extend this in another class to override behaviors
# Model for TestInterface typescript interface in "test-fixtures/test-interface.ts"
const AnyKind = preload("./AnyKind.gd")
const ISO8601Date = preload("./String.gd")
const ImportedInterface = preload("./ImportedInterface.gd")

var id: int
var str_key: String
var float_key: float
var bool_key: bool
var optional_date: String
# String | null
var nullable_optional_date: String
var date: String
# Literally "abcd"
var str_lit: String
# Literally 1
var int_lit: int
# Literally 1.0
var float_lit: float
# Literally "training" | Literally "full"
var str_union: String
var intf_union: AnyKind
# Literally true
var true_lit: bool
var imported: ImportedInterface
var record_object: TestInterface
var array: Array

func _init(src: Dictionary = {}):
    update(src)

func update(src: Dictionary):
    # custom import logic can be added by overriding this function

    id = src.id
    str_key = src.strKey
    float_key = src.floatKey
    bool_key = src.boolKey
    if 'optionalDate' in src: optional_date = ISO8601Date.new(src.optionalDate)
    if 'nullableOptionalDate' in src: nullable_optional_date = ISO8601Date.new(src.nullableOptionalDate) if __item__ != null else null
    date = ISO8601Date.new(src.date)
    str_lit = src.strLit
    int_lit = src.intLit
    float_lit = src.floatLit
    str_union = src.strUnion
    intf_union = AnyKind.new(src.intfUnion)
    true_lit = src.trueLit
    imported = ImportedInterface.new(src.imported)
    record_object = {}
    for __key__ in src.recordObject:
        var __value__ = src.recordObject[__key__]
        record_object[__key__] = TestInterface.new(__value__)

    array = []
    for __item__ in src.array:
        var __value__ = ImportedInterface.new(__item__))
        array.append(__value)
```

## Limitations

Generic type parameters aren't supported well. We'd have to get into extensible sub-classes to model them properly. Use `@typescript-to-gdscript-gd-impl` to opt out here...

TypeScript Enums with assignment expressions don't work (yet)

# Getting Started
## Set up Rust

- Install rustup
  `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh` or `rustup`
- Install `rust-analyzer` and `CodeLLDB` extensions in vscode.
- Restart VsCode

## Building and coding with Rust

`cargo run -- [arguments]` to run the program.

Add `#![allow(warnings)]` to the top of the file to ignore warnings while writing code
