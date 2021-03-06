# elm-decode-pipeline

A library for building decoders using the pipeline [`(|>)`](http://package.elm-lang.org/packages/elm-lang/core/3.0.0/Basics#|>)
operator and plain function calls.

## Motivation

It's common to decode into a record that has a `type alias`. Here's an example
of this from the [`object3`](http://package.elm-lang.org/packages/elm-lang/core/3.0.0/Json-Decode#object3)
docs:

```elm
type alias Job = { name : String, id : Int, completed : Bool }

point : Decoder Job
point =
  object3 Job
    ("name" := string)
    ("id" := int)
    ("completed" := bool)
```

This works because a record type alias can be called as a normal function. In
that case it accepts one argument for each field (in whatever order the fields
are declared in the type alias) and then returns an appropriate record built
with those arguments.

The `objectN` decoders are straightforward, but require manually changing N
whenever the field count changes. This library provides functions designed to
be used with the `|>` operator, with the goal of having decoders that are both
easy to read and easy to modify.

## Examples

Here is a decoder built with this library.

```elm
import Json.Decode exposing (int, string, float, Decoder)
import Json.Decode.Pipeline exposing (decode, required, optional, hardcoded)


type alias User =
  { id : Int
  , email : Maybe String
  , name : String
  , percentExcited : Float
  }


userDecoder : Decoder User
userDecoder =
  decode User
    |> required "id" int
    |> required "email" (nullable string) -- `null` decodes to `Nothing`
    |> optional "name" string "(fallback if name is `null` or not present)"
    |> hardcoded 1.0
```

In this example:

* `decode` is a synonym for [`succeed`](http://package.elm-lang.org/packages/elm-lang/core/3.0.0/Json-Decode#succeed) (it just reads better here)
* `required "id" int` is similar to `("id" := int)`
* `optional` is like `required`, but if the field is either `null` or not present, decoding does not fail; instead it succeeds with the provided fallback value.
* `hardcoded` does not look at the provided JSON, and instead always decodes to the same value.

You could use this decoder as follows:

```elm
Json.Decode.decodeString
  userDecoder
  """
    {"id": 123, "email": "sam@example.com", "name": "Sam Sample"}
  """
```

The result would be:

```elm
{ id = 123
, email = "sam@example.com"
, name = "Sam Sample"
, percentExcited = 1.0
}
```

Alternatively, you could use it like so:

```elm
Json.Decode.decodeString
  userDecoder
  """
    {"id": 123, "email": "sam@example.com", "percentExcited": "(hardcoded)"}
  """
```

In this case, the result would be:

```elm
{ id = 123
, email = "sam@example.com"
, name = "(fallback if name not present)"
, percentExcited = 1.0
}
```

---
[![NoRedInk](https://cloud.githubusercontent.com/assets/1094080/9069346/99522418-3a9d-11e5-8175-1c2bfd7a2ffe.png)][team]
[team]: http://noredink.com/about/team
