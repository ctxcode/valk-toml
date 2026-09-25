
# Documentation

Namespaces: [main](#main)

---

# main

## Errors for 'main'

```js
// Thrown when text is not TOML.
+ error Error (syntax, duplicate, type) payload { message: String, line: uint (0), column: uint (0) }
```

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
