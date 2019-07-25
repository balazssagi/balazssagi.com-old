---
title: "TIL#1: Getting Keys of an Enum as String Literals in TypeScript"
date: 2019-07-22
layout: layouts/post.njk
---

Let's say you have an Enum like this:

``` typescript
enum Color = {
  Primary,
  Secondary
}
```

To get a string literal type of keys you can use `keyof typeof`.

``` typescript
type ColorKeys = keyof typeof Color // => 'Primary' | 'Secondary'
```

This is equivalent to:
``` typescript
type ColorKeys = 'Primary' | 'Secondary'
```

See the [related section of the TypeScript documentation](https://www.typescriptlang.org/docs/handbook/enums.html#enums-at-compile-time).