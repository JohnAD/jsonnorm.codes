---
title: "Tool Documentation"
date: 2023-07-10T16:57:42-05:00
---

The `jsonnorm` command-line tool reads one JSON document and outputs a normalized
version of that document.

The tool expects the document to come from `stdin` if a filename is not passed
as the first parameters. The tool outputs the normalized document to `stdout`
unless a `-o <filename>` parameter is passed.

If the document is normalized successfully per the JSONNORM 1.0.0 specifications,
then the application exits with an exit code of 0. On the other hand, if it fails,
the reason why is output to `stderr` as an error message and the tool emits an
exit code of 1.

Although the tool works on operationally recursive documents (JSON), the algorithm
used is not recursive and should be fast and lean.

This document assumes you are operating under Linux, BSD, or a unix-like OS.
However, the instructions for Windows and MacOS are virtually identical.

## How to Install

tbd

## Examples of Use

{{< codecaption codeLang="bash" caption="Usage via optional parameters.">}}jsonnorm original.json -o normalized.json{{< /codecaption >}}

{{< codecaption codeLang="bash" caption="Usage with piping.">}}cat original.json | jsonnorm > normalized.json{{< /codecaption >}}

## Examples of Expected Behavior

{{< codecaption codeLang="bash" caption="Already normalized files should be identical. `sha1sum` tool should confirm this." copy="false">}}$ jsonnorm original.json -o normalized_a.json
$ jsonnorm normalized_a.json -o normalized_b.json
$ sha1sum normalized_a.json
f91bc626a87649250b6da1f8801efc854b210ec8  normalized_a.json
$ sha1sum normalized_b.json
f91bc626a87649250b6da1f8801efc854b210ec8  normalized_b.json
{{< /codecaption >}}

{{< codecaption codeLang="bash" caption="Here, we are going to see if the foobar app changed anything. It looks like a number was added to the `c` array." copy="false">}}
$ echo xyz.json
{"b":4.0e1,"c":[],"a":"hello"}

$ foobar xyz.json
writing output to `bling.json`

$ jsonnorm xyz.json -o normalized_xyz.json
$ jsonnorm bling.json -o normalized_bling.json

$ echo normalized_xyz.json
{
  "a": "hello",
  "b": 40,
  "c": [
  ]
}

$ echo normalized_bling.json
{
  "a": "hello",
  "b": 40,
  "c": [
     4
  ]
}

$ diff normalized_xyz.json normalized_bling.json
4a5
>     4
{{< /codecaption >}}

That same diff as seen by the GUI app `meld`:

![diff](/meld.png "difference as seen by meld")

## License and Source Code

This tool is open source and distributed under the MIT license. Visit
https://github.com/JohnAD/jsonnorm-tool for further details or to support
the project or authors.
