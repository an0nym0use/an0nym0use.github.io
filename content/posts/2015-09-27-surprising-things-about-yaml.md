---
date: 2015-09-27T00:00:00Z
title: Surprising things about YAML
url: surprising-things-about-yaml
---

YAML is one of the more popular data markup languages around. It is most often compared to JSON, which is ironic, because JSON is a subset of YAML. The appeal of YAML is clear: it allows very clear, very terse data descriptions. For those with an afternoon to kill, the [YAML spec][spec] has a bunch of neat features.

## This ain't JSON

...which means that **yay** we get comments! No explanation necessary.

```yaml
---
# The next line will blow your mind
key: value # See!
```

## Indicator characters

YAML nodes, apart from having _content_, can also have a _tag_ property. Tags give the parser more information about the content, a common use being the node or content type. For example, using explicit indicators for the two following documents yields different results:

```yaml
---
? key
: value

# This is equivalent to:
key: value
```

```yaml
---
! key
: value

# This is equivalent to:
key
```

Tags are really helpful for declaring content types. The YAML spec offers ways to be more explicit:

```yaml
---
string_value: !!str value
float_value: !!float 5
moar_float: !!float 2.2e-7
integer_value: !!int "5"
```

## Split strings over multiple lines

Splitting content over multiple lines can make YAML much more readable. We can also control how the multi-line text is interpreted.

```yaml
---
description: A depressed man suffering
  from insomnia meets a strange soap
  salesman named Tyler Durden.

# Fold content and ignore lines of indendation
key: >4
    - hey
    - donkey

# Equivalent to
key: "- hey - donkey\n"
```

## DRY up that data

Software like Rails use a couple awesome features of YAML in their config files: Anchors, aliases, and hash merges.

```yaml
---
# Establishes an anchor.
# Anchors can be referenced elsewhere in the document.
source: &DEFAULT
  awesome:
    - thing
    - stuff

# Expands the content of the anchor into the alias.
sauce: *DEFAULT

# We can merge the keys from two hashes as well.
sriracha:
  <<: *DEFAULT
  cool: elon musk
```

## Convenient values

The YAML node type definitions have a bunch of convenient ways to write data:

```yaml
---
key:      null        # empty content
bool1:    ON          # true
bool2:    off         # false
float:    NaN
float2:   Infinity
```

## Streams

A single string can have multiple documents within it. Parsers will commonly only return the first document on a stream, making YAML work well for streaming data applications.

```yaml
---
document: one
...

---
document: two
...

---
document: three
...
```

## Conclusion

These are just a few things that YAML can do out of the box. Thanks to the original YAML developers for making such an expressive way to serialize your data!


[spec]: http://www.yaml.org/spec/1.2/spec.html


