# Built-in Functions

Hiphops comes with a good selection of built-in functions to make common tasks within pipelines easier and allow for more powerful logic.

It's best practice to keep complex logic within apps and containerised steps, but there's no need to write scripts for basic functions such as decoding/encoding JSON, or decision level logic.

If you've written Terraform, many of these functions will be familiar to you.

## abs


The `abs` function returns the absolute value of the given number. If the number is zero or positive then it is returned as-is. If it is negative then it is multiplied by -1 to make it positive.

Example: 

```hcl
abs(42) // Returns 42
abs(0) // Returns 0
abs(-34.56) // Returns 34.56 
```

---

## alltrue/anytrue

### alltrue

The `alltrue` function accepts n-many boolean args and returns true if they are all true

Example: 

```hcl
alltrue(true, 1 == 1, event.foo == "bar") // Returns true
alltrue(true, 1 == 2, event.foo == "bar") // Returns false
```

### anytrue

The `anytrue` function accepts n-many boolean args and returns true if any of them are true

Example: 

```hcl
anytrue(true, 1 == 1, event.foo == "bar") // Returns true
anytrue(true, 1 == 2, event.foo == "bar") // Also returns true
```

---

## can/try


The `can` function evaluates a given expression and returns true if successful, false otherwise. This is useful to check if a property exists

Example: 

```hcl
can(event.possibly_unset.property)
```

The `try` function is similar to `can`. It accepts n-many expressions and returns the result of the first one that succeeds (or an error, if none succeed)

Example: 

```hcl
try(event.possibly_unset.branch == "foo", false)
try(event.possibly_unset, "mydefault") // Useful for setting default values
```

---

## ceil/floor

### ceil

The `ceil` function returns the next integer that is greater than or equal to the given value, which may be fractional.

Example: 

```hcl
ceil(1) // Returns 1
ceil(1.2) // Returns 2
```

### floor

The `floor` function returns the integer that is less than or equal to the given value, which may be fractional.

Example:

```hcl
floor(1) // Returns 1
floor(1.8) // Returns 1
```

---

## chomp


The `chomp` function removes newline characters at the end of a string.

Example: 

```hcl
chomp("Hello world\n") \\ Returns "Hello world"
chomp("Hello world\n\n") \\ Returns "Hello world"
chomp("Hello world\r\n") \\ Returns "Hello world"
```

---

## coalesce


The `coalesce` function returns the first argument that isn't null or empty.

All arguments must be convertible to the same type, so mixing objects and strings for example will error.

The `...` symbol can be used to expand a list as arguments

Example: 

```hcl
coalesce(1, 2) // Returns 1
coalesce("a", "b") // Returns "a"
coalesce("", "b") // Returns "b"
coalesce("", "b") // Returns "b"
coalesce(null, 1, null, 2) // Returns 1
coalesce([null, 2, 3]...) // Returns 2

// Beware of unexpected results due to automatic type conversion!
coalesce(1, "a") // Returns "1" <- Note: This is a string, despite the input being a number
```

---

## compact


The `compact` function removes null or empty string values from a list of strings

Example: 

```hcl
compact([null, "a", "", "b", "c"]) // Returns ["a", "b", "c"]
```

---

## concat


The `concat` function combines n-many lists into a single list

Example: 

```hcl
concat([1, 2], [1, 2], [2, 3]) // Returns [1, 2, 1, 2, 2, 3]
```

---

## csv


The `csv` function decodes a CSV formatted string into a list of maps. It implements the CSV format as defined in [RFC4180](https://www.ietf.org/rfc/rfc4180.txt)

Example: 

```hcl
csv("a,b\n1,2\n3,4") // Returns [{"a"=1, "b"=2}, {"a"=3, "b"=4}]
```

---

## env


The `env` function returns a given environment variable's value, or the default if it doesn't exist. The returned value will always be a string.

> Note: This executes on the hiphops instance serving/running workflows. If there are multiple instances with disparate values for env vars, the behaviour could be unexpected as pipelines steps are shared across them.

Example:

```hcl
// Assuming env var MY_ENV=foo
env("MY_ENV", "nice_default") // Returns "foo"

// Assuming no matching env var
env("NO_SUCH_ENV_VAR", "nice_default") // Returns "nice_default"
```

---

## file

The `file` function returns the contents of a file relative to the `example.hops` file it is called from. The returned value will always be a string.

Example:

```hcl
// Assuming file "myfile.txt" in the same directory and containing "Hello!"
file("myfile.txt") // Returns "Hello!"

// Assuming no matching file
file("no_such_file.txt") // Returns ""
```

---

## flatten


The `flatten` function reduces n-many nested lists into a single, flat list

Example: 

```hcl
flatten([1, 2, 3], [], [4, [5, 6]]) // Returns [1, 2, 3, 4, 5, 6]
```

---

## format


The `format` function produces a string by formatting values according to a template string. It is similar to the `fmt.Sprintf` in golang.

Example: 

```hcl
format("%s world!", "Hello") // Returns "Hello world!"
format("%s world %d!", "Hello", 2) // Returns "Hello world 2!"
```

Available template values:

|Verb|Result|
|----|------|
|`%%`|Percent sign literal (i.e. escaped)|
|`%b`|Convert to int and represent as binary|
|`%d`|Convert to int and represent as decimal|
|`%e`|Convert to number and represent as scientific notation|
|`%E`|Same as `%e` but uses uppercase E to introduce the exponent|
|`%f`|Convert to number and represent as decimal fraction|
|`%g`|Same as `%e` for large exponents, same as `%f` otherwise|
|`%G`|Same as `%E` for large exponents, same as `%f` otherwise|
|`%o`|Convert to int and represent as octal|
|`%q`|Convert to string and represent as JSON quoted string|
|`%s`|Insert string|
|`%t`|Insert boolean as 'true' or 'false' without quotes|
|`%v`|Default formatting based on the value's type|
|`%#v`|JSON serialization of the value - see function `jsonencode()` for an alternative|
|`%x`|Convert to int and represent as lowercase hexadecimal|
|`%X`|Same as `%x`, but uppercase hexadecimal|

---

## formatdate


The `formatdate` function reformats a timestamp given in [RFC3339](https://datatracker.ietf.org/doc/html/rfc3339) syntax into another time syntax defined by a given format string.

The supported format sequences are:

```
YY       Year modulo 100 zero-padded to two digits, like "06".
YYYY     Four (or more) digit year, like "2006".
M        Month number, like "1" for January.
MM       Month number zero-padded to two digits, like "01".
MMM      English month name abbreviated to three letters, like "Jan".
MMMM     English month name unabbreviated, like "January".
D        Day of month number, like "2".
DD       Day of month number zero-padded to two digits, like "02".
EEE      English day of week name abbreviated to three letters, like "Mon".
EEEE     English day of week name unabbreviated, like "Monday".
h        24-hour number, like "2".
hh       24-hour number zero-padded to two digits, like "02".
H        12-hour number, like "2".
HH       12-hour number zero-padded to two digits, like "02".
AA       Hour AM/PM marker in uppercase, like "AM".
aa       Hour AM/PM marker in lowercase, like "am".
m        Minute within hour, like "5".
mm       Minute within hour zero-padded to two digits, like "05".
s        Second within minute, like "9".
ss       Second within minute zero-padded to two digits, like "09".
ZZZZ     Timezone offset with just sign and digit, like "-0800".
ZZZZZ    Timezone offset with colon separating hours and minutes, like "-08:00".
Z        Like ZZZZZ but with a special case "Z" for UTC.
ZZZ      Like ZZZZ but with a special case "UTC" for UTC.
```

Example:

```hcl
formatdate("h:mm", "2023-02-01T12:04:00-01:00") // Returns "12:04"
```

---

## glob/xglob

The `glob` function accepts a string or list of strings and a glob pattern or list of patterns. Returns true if any strings match any patterns.

`xglob` has accepts the same arguments, but returns true only if all input strings match at least one pattern.

Example: 

```hcl
glob("branch_name", "branch*") // True
glob("ranch_name", "branch*") // False
glob(["README.md", "styles.css"], "*.css") // True
glob(["README.md", "styles.css"], ["*.css", "*.tf"]) // True
```

In general if you've used bash globbing, you'll know the syntax. 

More precisely, the full `glob()`/`xglob()` syntax for patterns is:

```
'**'        matches any sequence of characters (including across /).
            must be a path component by itself. (e.g. "a/**/b" matches "a/x/b", "a/x/y/b", "a/b", but not "ab").
'*'         matches any sequence of non-/ characters
'?'         matches any single non-/ character
'[' [ '^' ] { character-range } ']'
            character class (must be non-empty)
c           matches character c (c != '*', '?', '\\', '[')
'\\' c      matches character c

character-range:

c           matches character c (c != '\\', '-', ']')
'\\' c      matches character c
lo '-' hi   matches character c for lo <= c <= hi
```

---

## indent


The `indent` function adds the given number of spaces to all lines in a multi-line string, excluding the first line.

Example: 

```hcl
indent(4, "map:\none: 1\ntwo: 2")
// Returns:
// map:
//    one: 1
//    two: 2
```

---

## index


The `index` function finds the index for a given value in a list. If the given value is not present, this function will error

Example: 

```hcl
index([1, 2, 3], 2) // Returns 1
```

---

## int


The `int` function returns an int from a fractional number, rounding towards 0

Example: 

```hcl
int(1.8) // Returns 1
```

---

## join/split


The `join` function concatenates all elements of a list of strings, joined by the separator.

Example: 

```hcl
join(".", ["www", "example", "com"]) // Returns "www.example.com"
```

Conversely, the `split` function splits a string at all occurrences of the separator, returning a list of strings.

Example: 

```hcl
split(".", "www.example.com") // Returns ["www", "example", "com"]
split(".", "") // Returns [""]
split(".", "example") // Returns ["example"]
```

---

## jsondecode/jsonencode

### jsondecode

The `jsondecode` function decodes a JSON string to a native representation.

JSON is as defined in [RFC7159](https://datatracker.ietf.org/doc/html/rfc7159)

JSON values will be converted as follows:

|JSON|Hiphops|
|----|-------|
|`String`|`string`|
|`Number`|`number`|
|`Bool`|`bool`|
|`null`|`null`|
|`Object`|`object(attributes determined as per this table)`|
|`Array`|`tuple(attributes determined as per this table)`|

Example:

```hcl
jsondecode("{\"1\": 2, \"a\":\"b\"}") // Returns {"1"=2, "a"="b"}
```

### jsonencode

The `jsonencode` function encodes a value into a JSON string.

values will be converted to JSON as follows:

|Hiphops|JSON|
|----|-------|
|`string`|`String`|
|`number`|`Number`|
|`bool`|`Bool`|
|`null`|`null`|
|`list`|`Array`|
|`set`|`Array`|
|`tuple`|`Array`|
|`object`|`Object`|
|`map`|`Object`|

> Note: The conversion between Hiphops and JSON is lossy, so using `jsonencode` on the output from `jsondecode` may not always result in identical objects.

Example:

```hcl
jsonencode({"a"=1, "b"="c"}) // Returns "{\"a\":1,\"b\":\"c\"}"
```

---

## keys/values/zipmap

### keys

The `keys` function returns the keys from a map as a list. Returned list is sorted lexicographically

Example:

```hcl
keys({a="z", b="y", c="x"}) // Returns ["a", "b", "c"]
```

### values

The `values` function returns the values from a map as a list. Returned list is sorted lexicographically by their original _keys_ within the map.

Example: 

```hcl
values({a="z", b="y", c="x"}) // Returns ["z", "y", "x"]
```

### zipmap


The `zipmap` function returns a map from a list of keys and a list of values 'zipping' them together.

Arguments are: `zipmap(keys, values)`

Both arguments must be of the same length.

If the same key appears multiple times, then only the value with the highest index will appear in the output.

Example: 

```hcl
zipmap(["a", "b"], [1, 2]) // Returns {"a"=1, "b"=2}
zipmap(["a", "a"], [1, 2]) // Returns {"a"=2}
```

---

## length


The `length` function returns the length of a list, map, or string.

For a string, the length will be the number of characters.

For a list or map, the length will be the number of elements.


Example: 

```hcl
length([1, 1, 1]) // Returns 3
length("Hello world!") // Returns 12
length({a=1, b=2}) // Returns 2
```

---

## lookup

The `lookup` function returns the value for a given key in a map, returning the default value if the key does not exist.

Arguments are: `lookup(map, key, default)`

Example: 

```hcl
lookup({a="a_val", b="b_val"}, "a", "nice_default") // Returns "a_val"
lookup({a="a_val", b="b_val"}, "c", "nice_default") // Returns "nice_default"
```

---

## lower/upper/title

### lower

The `lower` function converts letters in a string to lowercase.

Example: 

```hcl
lower("WWW.EXAMPLE.COM") // Returns "www.example.com"
```

### upper

The `upper` function converts letters in a string to uppercase.

Example: 

```hcl
upper("www.example.com") // Returns "WWW.EXAMPLE.COM"
```

### title

The `title` function converts the first letter of each word in a string to uppercase.

Example: 

```hcl
title("the meaning of life is 42") // Returns "The Meaning Of Life Is 42"
```

---

## max/min

### max

The `max` function takes one or more number inputs and returns the greatest number.

Example: 

```hcl
max(1, 5, 99, 2) // Returns 99
```

### min

The `min` function function takes one or more number inputs and returns the smallest number.

Example: 

```hcl
min(1, 5, 99, 2) // Returns 1
```

---

## merge


The `merge` function takes n-many maps or objects, and returns a single merged map or object containing a set of elements from all arguments.

If the same key is defined in more than one input, the later positioned input takes precedence.

Example:

```hcl
merge({a=1, b=2}, {c=1, d=2}) // Returns {a=1, b=2, c=1, d=2}
merge({a=1, b=2}, {b=1, c=2}) // Returns {a=1, b=1, d=2}
merge({a=1, b=2}, {b=[1,2]}, {a=1, c=2}) // Returns {a=1, b=[1,2], c=2}
```

---

## range


The `range` function returns a list of numbers from start, limit, and step values

Arguments are: `range(start, limit, step)`

However, arguments can be omitted to change the behaviour (detailed in examples below)

Example:

```hcl
range(0, 6, 2) // Returns [0, 2, 4]
range(1, 3, 0.5) // Returns [1, 1.5, 2, 2.5]

range(1, 3) // Returns [1, 2]
range(3, 1) // Returns [3, 2]

range(2) // Returns [0, 1]
range(-2) // Returns [0, -1]
```

---

## regex/regexreplace/replace

### regex

The `regex` function applies a regular expression to a string and returns a list of all matches.

> Note: Hiphops' `regex` function is the same as Terraform's `regexall`, rather than their plain `regex`.
> Because the base Terraform `regex` throws an error in the case of no match, it's less suitable in cases like ours where you're likely to use it on unknown/dynamic data.

(Mostly) Quoting from the Terraform docs, our `regex`...

> returns a list with one element per match. That is:
>
> If the pattern has no capture groups at all, the result is a list of strings.
>
> If the pattern has one or more unnamed capture groups, the result is a list of lists.
>
> If the pattern has one or more named capture groups, the result is a list of maps.

and ...

> can also be used to test whether a particular string matches a given pattern, by testing whether the length of the resulting list of matches is greater than zero.

In general, `regex` uses a Perl-like syntax, which is the most familiar form for most programmers. For the full `regex` syntax spec, see [Golang's regexp syntax docs](https://pkg.go.dev/regexp/syntax#hdr-Syntax)

Example:

```hcl
regex("[0-9]+", "123abc456") // Returns ["123", "456"]
```

### regexreplace

The `regexreplace` function searches a string for substrings matching a regex, and replaces each occurence with a given replacement string. The substring argument must be a valid regular expression.

The replacement string can reference captured strings by using `$1`, `$2`, ... `$n` (or `$name` for named captured groups)

Arguments are: `regexreplace(string, regexstring, replacement)`

Example: 

```hcl
regexreplace("hello everyone!", "[h]+", "j") // Returns "jello everyone!"
```

### replace

The `replace` function searches a string, replacing each occurrence of a substring with a replacement string.

Arguments are: `replace(string, substring, replacement)`

Example: 

```hcl
replace("www.example.com", ".", "-") // Returns "www-example-com"
```

---

## reverse/sort

### reverse

The `reverse` function takes a sequence and returns a new sequence with the same elements in reverse order

Example: 

```hcl
reverse(["a", "b", "c"]) // Returns ["c", "b", "a"]
```

### sort


The `sort` function sorts a list of strings lexicographically

Example: 

```hcl
sort(["c", "a", "d", "z"]) // Returns ["a", "c", "d", "z"]
```


---

## setintersection/setproduct/setunion

### setintersection

The `setintersection` function returns the intersection of n-many given sets. That is, it returns a new set containing only the elements that all of the input sets have in common.

Example: 

```hcl
setintersection(["a", "b", "c"], ["b", "c"], ["c", "d"]) // Returns ["c"]
```

### setproduct


The `setproduct` function computes the Cartesion product for n-many given sets. That is it returns all of the possible combinations of elements from all of the input sets.

Example: 

```hcl
setproduct(["linux", "windows"], ["arm", "intel"])
// Returns
// [
//   ["linux", "arm"],
//   ["linux", "intel"],
//   ["windows", "arm"],
//   ["windows", "intel"],
// ]
```

### setunion


The `setunion` function returns the union of the input sets. That is, it takes n-many sets and returns a single set containing all set's elements.

Example: 

```hcl
setunion(["a", "b", "c"], ["c", "d"], ["c", "d", "e"]) // Returns ["a", "b", "c", "d", "e"]
```

---

## slice


The `slice` function extracts a consecutive subset of elements from within a list.

Arguments are: `slice(list, startindex, endindex)`

`startindex` is the first index to be included

`endindex` is the first index to be excluded

`slice` returns an error if either index is outside the bounds of the `list`.

Example: 

```hcl
slice(["1", "2", "3", "4"], 1, 3) // Returns ["2", "3"]
```

---

## substr


The `substr` function returns a substring from a string by offset and length.

Arguments are: `substr(string, offset, length)`

Example: 

```hcl
substr("Hello world", 1, 3) // Returns "ell" 
```

---

## template

The `template` function accepts a filename and variables. The template will be loaded from the filename and rendered using the variables. The result is returned as a string.

The template file path is relative to `example.hops` file.

The template syntax is Django like. Documentation [here](https://django.readthedocs.io/en/1.7.x/topics/templates.html).

> Note: Unlike Django, HTML escaping (where variables are made HTML safe before rendering) is disabled by default. To enable it, see below.

Example:

```hcl
template("mytemplate.txt", { "accountId": "anaccount", "password": "asecret" })
```

Where `template.txt` contains:

```
Your account and password are {{ accountId }}:{{ password }}
```

Returns:

```
Your account and password are anaccount:asecret
```

When rendering HTML, in order to turn on HTML escaping of variables, add `"autoescape": true` to the variables.

Example of unescaped variable (which could come from a malicious user):

```
<script>alert('xss');</script>
```

Example after escaping:

```
&lt;script&gt;alert(&#39;xss&#39;);&lt;/script&gt;
```

---

## timeadd


The `timeadd` function adds a duration to a timestamp

Example: 

```hcl
timeadd("2023-01-02T00:00:00Z", "1m") // Returns "2023-01-02T00:01:00Z"
```

---

## tobool/tonumber/tostring

### tobool

The `tobool` function converts an input to a bool. This function can cast "true" and "false" to the corresponding boolean value. Passing `null` will return simply return `null`.

Example: 

```hcl
tobool("true") // Returns true
tobool("false") // Returns false
tobool(true) // Returns true
tobool(false) // Returns false
tobool(null) // Returns null

// Everything else is an error, e.g.
tobool("True") // Error!
```

### tonumber

The `tonumber` function converts a string containing a decimal representation of a number to a native number type. Accepts strings (meeting the above criteria), null and numbers. Anything else throws an error

Example: 

```hcl
tonumber("123") // Returns 123
tonumber(1) // Returns 1
tonumber(null) // Returns null

// Everything else is an error, e.g.
tonumber("one") // Error!
```

### tostring

The `tostring` function converts any `string`, `number`, or `bool` to a string. Any other input throws an error.

Example: 

```hcl
tostring(123) // Returns "123"
tostring(false) // Returns "false"
tostring("hello") // Returns "hello"
tostring(null) // Returns null <- Note it's the actual `null` value, not a string.

// Everything else is an error, e.g.
tostring([]) // Error!
```

---

## tolist/toset

### tolist

The `tolist` function converts a set to a list. Given sets are unordered, the initial list order is unreliable.

Due to how Hiphops re-evaluates expressions between steps in a pipeline, the order may change between `calls`. See the `sort` function to establish a reliable order for lists of strings.

Example: 

```hcl
tolist(["a", "b", "c"]) // Returns ["a", "b", "c"]
```

### toset

The `toset` function converts a list to a set

Useful for removing duplicate values in a list, or preparing a list to be passed to a function that requires a set as input.

The set returned will be converted to a single type of element, with lists of mixed types being converted to the most general type `[most general] (string > number > boolean) [least general]`

Sets are unordered.

Example: 

```hcl
toset(["a", "b", "b", 1]) // Returns ["b", "a", "1"]
```

---

## trim/trimprefix/trimspace/trimsuffix

### trim

The `trim` function removes the given characters from the start and end of the a string.

Arguments are: `trim(string, chars_to_remove_string)`

Example: 

```hcl
trim("hello", "oh") // Returns "ell"
trim(" what?", " ") // Returns "what?"
trim("abcb", "b") // Returns "abc"
```

### trimprefix

The `trimprefix` function is the same as `trim`, but only removes from the start of the string.

Example: 

```hcl
trimprefix("hello", "oh") // Returns "ello"
```

### trimspace

The `trimspace` function removes space characters from the start and end of a string.
Space characters are as defined by Unicode, which includes spaces, tabs, and newline characters.

Example: 

```hcl
trimspace("  Hello world!\n") // Returns "Hello world!"
```

### trimsuffix

The `trimsuffix` function is the same as `trim`, but only removes from the end of string.

Example: 

```hcl
trimsuffix("hello", "oh") // Returns "hell"
```

---

## versiontmpl


The `versiontmpl` function generates a CalVer or random 'pet name' version according to a template.
It accepts a template string and returns a string.

Example: 

```hcl
versiontmpl("v[calver]") // Generates CalVer for current date, e.g. v2023.01.02
versiontmpl("[adj]-[pet]") // Generates an Ubuntu style 'pet' name version, e.g. happy-hedgehog
```

Template values are surrounded by `[]`. Unknown values will be left as-is. Can include arbritrary text (such as the leading `v` in the first example above)

All supported template values:

- `[pet]` Random animal 'pet' name from curated list
- `[adj]` Random adjective from curated list
- `[adv]` Random adverb from curated list
- `[calver]` CalVer for current date in yyyy.mm.dd format
- `[yyyy]` Current four digit year
- `[yy]` Current two digit year
- `[mm]` Current two digit month, zero padded
- `[m]` Current digit month, non zero padded
- `[dd]` Current two digit day of month, zero padded
- `[d]` Current digit day of month, non zero padded

---
