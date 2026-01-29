# dalimit

Some functions for working with delimiters in fragments of Janet
source code.

## Functions

Provided functions include:

* `close-delims` - given a fragment of Janet source code, tries [1] to
  return a "delimiter-closed" version of the fragment.

* `closing-delims` - given a fragment of Janet source code, tries to
  return an array of what characters could "close-up" the unclosed
  delimiters

Note that these functions only try to handle the simple case of a
fragment that is missing closing delimiters at the "end".

See below for concrete example usages,

## Usages

### close-delims


```janet
(close-delims "(")
# =>
"()"

(close-delims "'(")
# =>
"'()"

(close-delims "~(smile breathe")
# =>
"~(smile breathe)"

(close-delims "{:a 1\n:b 2")
# =>
"{:a 1\n:b 2}"

(close-delims "[1 2 3 5 8")
# =>
"[1 2 3 5 8]"

(close-delims "{:a 1 :b [:x :y")
# =>
"{:a 1 :b [:x :y]}"

(def maybe-code
  (string "(defn hi\n"
          "  [x]\n"
          "  (+ 3 (* 8\n"
          "          (- 2 1)"))

(close-delims maybe-code)
# =>
(string maybe-code ")))")
```

### closing-delims

```janet
(closing-delims "(defn a")
# =>
@[")"]

(closing-delims "(defn a)")
# =>
@[]

(closing-delims "(defn a))")
# =>
nil

(closing-delims "{:a 1")
# =>
@["}"]

(closing-delims "[:x :y")
# =>
@["]"]

(closing-delims
  (string "{:a 1\n"
          " :b"))
# =>
@["}"]

(closing-delims
  (string "(defn my-fn\n"
          "  [x]\n"
          "  (+ x 1"))
# =>
@[")" ")"]

(closing-delims `"nice string"`)
# =>
@[]

(closing-delims `"not quite a string`)
# =>
@[`"`]

(closing-delims
  (string "``\n"
          "  hello"))
# =>
@["`" "`"]

(closing-delims `("what is going on?)`)
# =>
@[`"` `)`]

(closing-delims "``hello``")
# =>
@[]

(closing-delims "``hello```")
# =>
@["`"]

(closing-delims "@`hello")
# =>
@["`"]

(closing-delims "1")
# =>
@[]

(closing-delims "")
# =>
@[]
```

## Footnotes

[1] If the fragment ends with a line comment (without a trailing
newline), by default `close-delims` does not give a correct result,
e.g.

```janet
(close-delims "[:a # hello")
# =>
"[:a # hello]"
```

To get a result like `[:a # hello\n]`, pass a truthy value as
the optional second argument:

```janet
(close-delims "[:a # hello" :try-harder)
# =>
"[:a # hello\n]"
```

The default behavior for `close-delims` is because otherise it would
interfere with other cases (because of the extra newline) and in
practice the trailing line comment case has not surfaced in the
context of editor use.

