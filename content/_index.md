---
title: "JSON Normalized Expression"
---
JSON is a universally recognized method of serializing data that can be
passed between systems and people. Or, to put it more simply, it is a way
for information to be "written out" as a readable series of characters and spaces.

The spec for this can be found at https://www.json.org/

For computer-to-computer transactions, the official spec is enough to
properly convey data between systems. However, the specification is "loose"
in the sense that there are an almost infinite ways to express the same data.

For example:

```json
{"b":4,"c":[],"a":"hello"}
```

and

```json
{
    "a":   "hello",
    "b":         4,
    "c":[]
}
```

contain exactly the same data. So, the actual content of a JSON document
cannot be

- predicted at byte-for-byte level, or
- easily compared with other documents using line-oriented tools.

So, this website contains:

1. a [specification](/specification) for "normalizing" JSON documents to make them both predictable and easily compared.
2. a command-line tool for transforming any valid JSON document into it's normalized form.
3. a set of examples for testing libraries that wish to conform to this
standard.

