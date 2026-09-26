
# valk-toml

TOML for [Valk](https://valk-lang.dev): reading and writing, straight into and out of your own
classes. Purely written in Valk, with no os-package dependencies.

Requires Valk 0.7.0 or newer.

## Install

```
vman install github.com/ctxcode/valk-toml
```

## Example

```rust
use toml
use valk.fs

class Server {
    host: String ("127.0.0.1")
    port: uint (8080)
}

class Settings {
    name: String
    debug: bool (false)
    server: Server
}

fn main() {
    let text = fs.read("config.toml") ! panic("Cannot read config.toml")
    let settings = toml.decode_to[Settings](text) ! panic(E.message)
    println(settings.server.port)

    fs.write("config.toml", toml.encode_of(settings)) ! panic("Cannot write config.toml")
}
```

Every field of the class must be in the file, unless it is nullable or has a default. The
error names the key: `'server.host' is missing`, `'server.port' should be an integer that fits the
field`. A syntax error also sets `E.line` and `E.column`.

## When the shape is not fixed

`decode` returns a `json.Value`, which is the same document type the standard library's `json`
package uses, so everything that works with JSON works here:

```rust
let document = toml.decode(text) ! panic("%{E.message}")
println(document["server"]["port"].int)
println(document["tags"][0].string)
each document["servers"].array.values as server : println(server["name"].string)
```

`toml.encode(document)` writes one back out.

## What it reads

Everything a configuration file uses: comments, bare and quoted keys, dotted keys, basic and
literal strings with their multi-line forms and escapes, integers in decimal, hex, octal and
binary with `_` separators, floats with exponents and `inf`/`nan`, booleans, arrays over as many
lines as they like, inline tables, `[table]`, `[a.nested.table]` and `[[arrays of tables]]`.

In your own classes, a `time.DateTime` field (or `?time.DateTime`, or `Array[time.DateTime]`)
takes a TOML date or date and time. One with an offset is converted to UTC, one without is taken
as UTC, and a date alone is its midnight; a time of day on its own does not fit a `DateTime`.
`encode_of` writes such a field as a date and time in UTC, and leaves out fields that are null.
Dates inside an array of tables need Valk 0.7.8 or newer, and keep their offset rather than being
converted to UTC.

`decode` passes dates through as the text they were written as: `1979-05-27T07:32:00Z` stays
exactly that.

## Errors

`syntax`, `duplicate` (a key or table given twice) and `type` (the document does not fit the
class), each with the `line` and `column` where the reader stopped:

```rust
toml.decode(text) ! {
    println(E.message + " at line " + E.line + ", column " + E.column)
    return
}
// A string was not closed before the end of the line (line 2, column 1)
```

## Development

`make test` runs the suite, `make example` runs the example, `make lint` checks the sources and
`make docs` regenerates the API documentation.
