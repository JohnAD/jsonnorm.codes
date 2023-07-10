---
title: "JSON Normalized"
TableOfContents: true
---
# `JSONNORM 1.0.0`

JSON is a universally recognized method of serializing data that can be
passed between systems and people. Or, to put it more simply, it is a way
for information to be "written out" as a readable series of characters and spaces.

The spec for generic JSON can be found at https://www.json.org/

## The Problem

For computer-to-computer transactions, the official spec is often enough to
generaly convey data between systems. However, the specification is "loose"
in the sense that there are an almost infinite ways to express the same data.

For example, this JSON document:

{{< codecaption codeLang="json" copy="false">}}{"b":40,"c":[],"a":"hello"}{{< /codecaption >}}

and this JSON document:

{{< codecaption codeLang="json" copy="false">}}{
    "a":   "hello",
    "b":     4.0e1,
    "c":[]
}{{< /codecaption >}}

contain exactly the same data. So, the actual content of a JSON document
cannot be

- completely predicted, or
- easily compared with other documents using line-oriented tools like git.

### Problem 1: Lack of Prediction

If an application reads an "input" JSON document; determines that nothing should be done
to the data; and writes a "output" JSON document containing that same data, the input
and output files will be identical. Correct?

In today's environment, they probably won't be. They "effectively" contain the
same information, but the ordering of object fields, the spacing used, numeric
exponent handling, and other characteristics could very easily change things.

For many applications, this is might not be a problem. But for applications where
this is a problem, we want a way to *normalize* the JSON so that the same data
creates the exact same JSON document.

If saved to files, the file should have the same length and line count. If a hash
is calculated of the files, they should result in the exact same hash.

So, that is the first goal:

**Normalize JSON documents to an exhaustive degree that ensures that the exact 
same data creates precisely the same document.**

### Problem 2: Changes Not Trackable

An application reads an "input" JSON document; but this time, it makes some 
changes to the data. It writes a "output" JSON document containing the changed 
data.

It would be useful, especially when using tracking tools like git and diff,
to be able to easily identify what changes have been made.

To do this, not only would the JSON need to be normalized, it would need to be
normalized in such a manner that the changes are easily tracked by line-oriented
tools.

That is the second goal:

**Normalize JSON documents such that changes are easily tracked on a 
line-by-line basis.**

## The Solution

This website presents a formal specification called `JSONNORM 1.0.0` that creates
a very exacting normalization. Normalized JSON is fully compliant
with the JSON spec, so any application consuming a normalized document need not
know that a document is normalized.

See the [JSONNORM specification here](/specification).

The website also hosts the documentation for a command line tool called
`jsonnorm` that transforms JSON documents into the normalized form. If the
document cannot be normalized, the reason is spelled out in detail.

This tool could be used by hand, or in shell scripts, or in CI/CD pipelines.

See the [`jsonnorm` tool documentation here](/tooldoc).

There is also a set of testing files that can be used by libraries for verifying
the normalization. These files are part of the [source code repo](tbd)
of `jsonnorm`.
