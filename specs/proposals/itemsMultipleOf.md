# Proposal: `itemsMultipleOf` Keyword

## Abstract

This document proposes a new `itemsMultipleOf` keyword for the JSON Schema
Validation vocabulary. The keyword asserts that the length of an array instance
is a multiple of a given strictly positive integer.

## Motivation

JSON Schema provides symmetric numeric constraint keywords:

| Numeric keyword | Array-length analogue |
|---|---|
| `minimum` | `minItems` |
| `maximum` | `maxItems` |
| `multipleOf` | *(missing)* |

No keyword exists to assert that an array's length is divisible by a given
value. This gap makes it unnecessarily verbose to validate fixed-width tuple
streams such as coordinate pairs, RGB triples, or any encoding that stores
records as a flat array with a known field count.

## Specification

### Keyword name

`itemsMultipleOf`

### Vocabulary

This keyword belongs to the JSON Schema Validation vocabulary alongside
`minItems` and `maxItems`.

### Value

The value of `itemsMultipleOf` MUST be a strictly positive integer (i.e., a
JSON number with no fractional part and a value greater than or equal to 1).

### Assertion behavior

If the instance is an array, this keyword asserts that
`instance.length % value === 0`. The instance is valid if and only if the
remainder is zero.

If the instance is not an array, this keyword MUST be ignored (it does not
assert anything and does not affect validation).

### Annotation behavior

This keyword produces no annotation.

## Examples

### Coordinate pairs

A flat array that encodes an arbitrary number of 2-D coordinate pairs:

```json
{
  "type": "array",
  "items": { "type": "number" },
  "itemsMultipleOf": 2
}
```

- `[1, 2, 3, 4]` — valid (length 4, 4 % 2 === 0)
- `[1, 2, 3]` — invalid (length 3, 3 % 2 !== 0)
- `[]` — valid (length 0, 0 % 2 === 0)
- `"hello"` — valid (not an array; keyword is ignored)

### RGB triples

A flat array that encodes an arbitrary number of RGB colour triples:

```json
{
  "type": "array",
  "items": { "type": "integer", "minimum": 0, "maximum": 255 },
  "itemsMultipleOf": 3
}
```

- `[255, 0, 128, 0, 255, 64]` — valid (length 6, 6 % 3 === 0)
- `[255, 0]` — invalid (length 2, 2 % 3 !== 0)

## Interaction with Other Keywords

`itemsMultipleOf` composes naturally with `minItems` and `maxItems`. All three
keywords are evaluated independently; an instance must satisfy each one that is
present.

```json
{
  "type": "array",
  "items": { "type": "number" },
  "minItems": 2,
  "maxItems": 20,
  "itemsMultipleOf": 2
}
```

The schema above accepts non-empty arrays of numbers whose length is even and
does not exceed 20.

## Alternatives Considered

### Sub-schema approach: `arrayLength: { multipleOf: n }`

One alternative is to introduce an `arrayLength` keyword that accepts a full
numeric sub-schema applied to the instance length:

```json
{ "arrayLength": { "multipleOf": 2 } }
```

While this approach is more general, it introduces additional schema nesting,
increases implementation complexity, and makes schemas harder to read at a
glance. The single-keyword form `itemsMultipleOf` is self-contained, easier to
teach, and directly mirrors the existing `multipleOf` keyword for numbers.
Given that the only length constraint not yet covered is divisibility, a
dedicated keyword is preferred over a full sub-schema mechanism.
