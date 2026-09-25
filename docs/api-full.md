
# Documentation

Namespaces: [main](#main)

---

# main

## Errors for 'main'

```js
// Thrown when text is not TOML.
+ error Error (syntax, duplicate, type) payload { message: String, line: uint (0), column: uint (0) }
```

### Error

Thrown when text is not TOML.

- `syntax`: the text does not read as TOML.
- `duplicate`: a key or a table is given twice, which TOML does not allow.
- `type`: a value is not what the type it is read into needs.

`line` and `column` say where the reader stopped, counting from one, so the message can be
shown against the file.

## Functions for 'main'

```js
// Reads TOML and returns it as a `json.Value`, so everything that works with JSON works with it.
+ fn decode(text: String) Value !Error
// Reads TOML into a class or struct of your own.
+ fn decode_to[T](text: String) T !Error
// Writes a `json.Value` as TOML.
+ fn encode(value: Value) String
// Writes a class or struct of your own as TOML, the way `json.from` would read it; a `time.DateTime` field is written as a TOML date and time in UTC.
+ fn encode_of(value: $T) String
```

### decode

Reads TOML and returns it as a `json.Value`, so everything that works with JSON works with it.

Dates and times come through as the text they were written as; TOML has types for them and
JSON does not. `decode_to` reads them into `time.DateTime` fields.

```valk
let config = toml.decode(fs.read("config.toml") !!) ! panic("%{E.message}")
println(config["server"]["port"].int)
```

### decode_to

Reads TOML into a class or struct of your own.

Every field must be in the text, unless it is nullable or has a default. A value that does
not fit throws `.type`, and its message names the key, such as `'server.port' is missing`;
`line` and `column` are only set for syntax errors. A `time.DateTime` field takes a date, or a
date and time, converted to UTC.

```valk
class Settings {
    name: String
    port: uint (8080)
}

let settings = toml.decode_to[Settings](text) ! panic("%{E.message}")
```

### encode

Writes a `json.Value` as TOML.

Plain values come first, then the tables, then the arrays of tables, which is the order TOML
is read in. Text that is not a bare key is quoted, and a string is escaped.

```valk
fs.write("config.toml", toml.encode(settings)) ! panic("%{E.message}")
```

### encode_of

Writes a class or struct of your own as TOML, the way `json.from` would read it; a
`time.DateTime` field is written as a TOML date and time in UTC.
