---
title: "Specifications for JSON Normalization"
date: 2023-07-08T16:57:42-05:00
---
# `JSONNORM 1.0.0`

This is the specification for a normalized expresion of the JSON data
format. This format is strictly a superset of the JSON specification defined
at https://json.org/. That is to say, this specification is IN ADDITION TO
the default specification. Nothing is changed or removed from the default.

So, NOT all JSON documents can be stored in JSONNORM format, but

All JSONNORM documents are stored in JSON format.

This specification uses the phrase "document" to refer to a JSON
serialization IN ITS ENTIRETY.

## DOCUMENT MUST BE BASED ON AN OBJECT

The default specification allows for any elements or values in a documents. However,
JSONNORM limits the base element to a single object. This is very often
what is seen in typical practice, but JSONNORM specifies this explicitly.

#### Examples of BAD documents

```json
[
  {
    "name": "Larry",
    "score": 4
  },
  {
    "name": "Bob",
    "score": 2
  }
]
```
> A document based on an array is legit JSON, but **bad** JSONNORM


```json
4
```
> A document based on a single value is legit JSON, but **bad** JSONNORM

#### Example of a GOOD document

```json{title="Titled code block" verbatim=false}
{
  "name": "Larry",
  "score": 4
}
```
> A document must always be based on an object.

## LINES AND LINE ENDINGS

A line contains one or more UTF8 characters terminated by a newline LF
character. This is true of all lines, including the final `}` at the end
of the document. To put it another way, all documents will have a final
byte of code `10` (hex `0A`).

Except within strings (including names), the characters are further
restricted to the ASCII character set.

Carriage returns are found nowhere in a document. If a carriage return is
carried in a string, it is typically escaped with a leading slash followed
by an `r` aka `\r`. But ultimately this is a decision made by the systems
using the document.

There is no line length limit.

## INDENTATION

At the start of a document, indentation starts at zero. Since the base of
the document is always an object, the indentation immediately raises to 2
spaces until the object closes.

Every object or array causes it's subtending elements to be further indented
by 2 additional spaces.

Indentation is always represented by a single byte SPACE characters, which is
a byte code `32` (hex `20`). All indentation rises and falls by
increments and decrements of two.

## STRINGS

Strings are encapsulated by QUOTATION MARKS. The contain zero or more UTF8
characters between them. If a quotation mark is contain within a string, it
is escaped with a backslash per the JSON spec.

There is no character limit to a string.

## OBJECTS

A JSON document, when normalized, must be based on an object. 

Every object starts with a starting curly brace `{`, which ends the line.

Every object ends on a line starting with an ending curly brace `}`. If
needed, then a comma immediately follows. Otherwise, the line ends also.

The open and closing braces are at the SAME level of indentation. And empty
object has no line between that starting line and the ending line. Very
explicityly, and empty object may NOT be represented by a curly brace pair 
`{}` on a single line.

An object has zero or more elements, each of which has a name, a colon, and a 
value. (See JSON spec.) The order of these is SORTED by name (see NAME
SORTING).

Names may not be duplicated in the context of a single object. The JSON spec
allows repeating of the same name, but normalization does NOT.

The name is a string. It is immediately followed by a colon. That is then
followed by a single SPACE. That is then followed by the value (or start of
the value if a object or array).

Each name/value element is separated by a comma. The comma is placed
immediately after the value and is followed by the LF that ends the line.

The JSON spec does not allow for "trailing commas" in a list of name/values, so
inserting an name/value element to the end of an object will appear as a "change to the
last line" (to add the comma) plus the insertion of the new line.

## NAME SORTING

The fields in an object are to be sorted by their name so that the same data
is always represented in the same order.

It is tempting sort using the official Unicode collation algorithm.
That algorithm would sort the fields in strict alphabetic order that is very
familiar to human readers. For example, the collation algorithm would sort the
following characters in this order: 

```text
A a à B
```

Unfortunately, the collation algorithm has two problems:

1. (major) The algorithm changes. In fact, UTF8 itself changes as new characters
are added/removed by the Unicode Consortium. The collation algorithmm itself
changes. It is currently on version 14.
2. (minor) The algorithm is very complex and not available in all computer
languages.

Predictability is more important for normalization than human comfort. So, 
instead the name strings are to be simply sorted by character.

Unfortunately, this means the example list would be sorted 

```text
A B a à
```

which is less intuitive.

Please note that this is not the same thing as sorting by byte (like one can
do with ASCII strings.) Instead, each character, which can be 8, 16 or
32bits in size, should be placed into unsigned 32-bit integer arrays. And
those arrays are used as the source of determining whether one name comes
before another.

Duplicate names are forbidden in the context of an object's name/value
list.

## ARRAYS

A JSON array is an ordered sequence of unnamed values.

A JSON array is NOT an array in the strict sense [used in Computer
Science](https://en.wikipedia.org/wiki/Array_(data_structure) ).
Computer Science defines an array as an ordered sequence of entries of the same
type/size. But a JSON array allows mixed types.
So it is *technically* a **list**. The unfortunate naming is due to the historical connection to Javascript which 
is historically connected to C, which used arrays-of-pointers to store mixed 
data-type lists. However, to prevent confusion, we will still refer to these
lists as arrays.

The original spec leaves this nature undiscussed, so this specification
states it explicitely: arrays may mix data types.

```json
{
  "a-list": [
    99,
    false,
    null,
    [
      19,
      "bob",
      true
    ],
    "hello"
  ]
}
```
> The value of `a-list` is a perfectly valid array. The 4th item in that
> array is also a perfectly valid array.

The start of an array begins with a OPEN SQUARE BRACKET (`[`) and ends the current
line.

The entries of the array are indented by a further 2 spaces. Each entry
starts it's own line.

The end of an array is on a new line with a CLOSING SQUARE BRACKET (`]`). It
is followed by a comma if needed. Then the line is terminated with a LF.

The open and closing brackets are always on separate lines even if the array
is empty.

The JSON spec does not allow for "trailing commas" in an array, so
inserting a value to the end of an list will appear as a "change to the
last line" (to add the comma) plus the insertion of the new line.

## EMPTY VS UNKNOWN VS VOID

Philosophically, there are subtle but real distinctions between these three
concepts and the original JSON spec *implies* them, but does not address the
issue explicity.

All three ideas represent "missing" or not existent values. They are
represented in three different ways in JSON. JSONNORM is very explicit about
this to avoid confusion.

#### EMPTY

An empty value is a real value, but that value is "nothing" or
"emptiness" or "scaled of empty".

In a document, they are represented as:

| value type     | empty         |
|----------------|---------------|
| string         | `""`          |
| number         | `0`           |
| object         | `{}`          |
| array          | `[]`          |

The boolean value type (`true`/`false`) has no "empty" representation. It
is common in most computer languages to use `false` as a stand-in for default.
So, this specification will also use `false` has the less-than-ideal meaning of
boolean emptiness.

An example of emptiness in mathematics: 

**3 + 0 = 3** ==> *Adding nothing to 3 results in 3.*

#### UNKNOWN

In computer science, a value that is not yet known (or might not ever be
known), is called a `NULL` or `NUL`. There is a theoretical value, but it is
simply not known in the context.

In JSON, a "value type" of `null` is given to represent this concept.

This is why a list such as `[3, 2, null, -5]` has four values, not three. One of them is
simply not known.

[reference from SQL for
NULL](https://learnsql.com/blog/what-is-null-in-sql/#:~:text=Simply%20put%2C%20an%20SQL%20NULL,until%20the%20order%20has%20arrived.)

Note: verbally, is it more accurate to say "this value is null" than "this
value equals null."

Note: this is *not* the same thing as a `NIL`. `NIL` is an empty pointer or
reference often used in computer programming. So, `NIL` is the EMPTY state for a
value type of "pointer". JSON does not explicitly store pointer values, so `NIL`
is never used in JSON.

Note: `"null"` and `null` are different things in JSON. The first is a four
letter string containing the word "null", the second is an actual null.

An example of NULL in mathematics: 

**3 + NULL = NULL**  ==> *Adding something to 3 results in something else.*

So, if later, the first NULL is found to be 5, then the second NULL will become 8. The expression
would be more commonly written like **3 + x = y**, since mathematic variables, such as x and y are
unknown (NULL) until they are known.

#### VOID

There are multiple names for this concept. In the programming language C it is
called `void`. In paperwork, it is often called "non-applicable".

It is explicitly stating that there cannot be a value, not simply that it
is not known.

In JSON, a VOID is represented by ommision. In other words, you can
represent VOID by *not* including in the document. Because JSON structure is 
dynamic, this works quite well.

An example using real world objects: 

- a spider has 8 legs,
- a worm has zero legs (EMPTY),
- an unknown creature trapped in a box has an unknown number of legs (NULL), but
- an asteroid can't have legs; it is a silly question (VOID).

An example in mathematics: 

**3 + ~~x~~ = 3** ==> *There is no x, so it is not added to 3, which results in 3.*

### Example of All Three in One document

With the following document for values A, B, C, and D:

```json
{
  "A": [
    1,
    2,
    3
  ],
  "B": [
  ],
  "C": null
}
```

- "A" has a list that is not empty.
- "B" has a list that is EMPTY.
- "C" has an UNKNOWN value.
- "D" is VOID as it is ommitted.

Sadly, some public programming libraries and frameworks cannot easily distinquish 
between these three conditions and tend to convolute them.

## NUMBERS

The normalizing numbers in JSON spec as at least 5 challenges:

1. (infinity) Numbers can be infinite in size. Is this realistic?
2. (diff exp) With exponent support there are multiple ways to express the exact same 
   number. (`1.23E4` vs `12.3E3`)
3. (marker) The exponent marker can either be lower case `e` or upper case `E`.
4. (sig) There is no discussion of significance (or the related concept of 
   precision). 
   (e.g. `1.00` vs `1.000`) Is significance stored or ignored?
5. (bin) Numbers are expressed in decimal but a common source-of-truth for JSON, 
   Javascript, stores them as binary floats; not decimal floats. Binary is 
   base 2 and Decimal is base 10 (2 * 5). This causes conversion errors in some 
   circumstances. (This is not a bug but a philosophical difference between 
   numbering systems.) Most libraries follow this convention storing in binary.

This section of the specification resolves these issues, but acknowledges that
the chosen answer is not perfect for all users.

Numbers are to be stored in a manner that complies with IEEE 754-2008 decimal 
128-bit standard using DPD encoding as it commonly expressed in libraries.
This does NOT mean that an app or library that reads or 
writes JSONNORM compliant documents must store the numbers in decimal. It 
simply means that the expression of the number must match that format.

So, the chosen answer to the above restrictions are:

1. (infinity) Numbers are limited to 11 digits of precision. That is, the main 
   number cannot have more than 11 decimal digits. If converting to JSONNORM
   the remaining digits are rounded per the well known 
   [BANKERS ROUNDING](https://wiki.c2.com/?BankersRounding) algorithm.
   The exponent is limited to the range −6143 to +6144. If a greater/lesser
   exponent is needed, the document is not expressable in JSONNORM.
   Examples: `1.23456789012345` becomes `1.2345678901`. `9876543210987654` becomes
   `9.8765432110E15` (notice the round-up).
2. (diff exp) Numbers needing an exponent are expressed in scientific notation,
   which has a single leading digit before the decimal point. Other numbers do
   not use the exponent. So, `10.0E4` becomes `1.00E5`.
   And, `1.00E2` becomes `100`.
3. (marker) Use upper case `E`. If the exponent is negative, prefix with a `-`, 
   but if the exponent is positive do NOT prefix with a plus sign. 
   Examples: `3.243E20` and `9.999900E-338`
4. (sig) Significance and precision are expressable. So, `1.0` and `1.000` are
   different numbers.
   This specification does not require that the app using those numbers actually
   honors precision or not. It is, however, expressed and stored where possible.
5. (bin) Numbers are expressed per the IEEE 754-2008 128-bit DPD-encoding 
   characteristics. Individual apps might or might not actually honor those
   characteristics, but documents will be expressed as if they were.

Leading zeroes are not permmitted. This is already part of the JSON spec.

Note: The IEEE 754-2008 encoding allows for infinity (`+inf` `-inf`) and not-a-number
(`NaN`). However, neither JSON or JSONNORM documents allow for such expressions.

