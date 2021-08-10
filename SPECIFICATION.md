# Improperties File Format Specification
This document entails the specification for the improperties file format. <br/>
The improperties file format derives from the [properties](https://docs.oracle.com/cd/E23095_01/Platform.93/ATGProgGuide/html/s0204propertiesfileformat01.html) file format, which allows for a simple string key to string value system. While improperties maintains string-only values, it allows for keyed object and list data structures. <br/>
Improperties is meant to be fully compatible with existing properties files, and should be fully able to parse any properties file.

## Example File
```
foo = bar
abc = def

some_months ->
  - January
  - February
  - March
  - April
--

stuff ->
  red = blue
  green = yellow
  other_colors ->
    - orange
    - purple
    - brown
  --
  logins ->
    - ->
      user = root
      pass = root
    --
    - ->
      user = person
      pass = 12345
    --
  --
--
```

## File Extension
Improperties files may use the following extensions: <br/>
- `.improperties`
- `.imprpt`
<br/>

## File Format
### Keys and Values
An improperties file may contain keys and their associated values on a single line, separated by either `=` or `:`. <br/>
The separator may have one whitespace character before or after, which will be excluded from the key or value if present. <br/>
`key=value`, `key = value`, `key= value`, and `key =value` are all valid ways to notate that the string `key` may result in the string `value`. The same applies should the `=` characters be `:` instead. <br/><br/>
The value string begins after the first occurrence of `=` or `:` in the line (or after the first whitespace character if present). The value string may contain additional `=` or `:` symbols, however the key may only contain these symbols if prefixed with `\`. <br/>
For example, `smiley\:-) : =D` notates that the string `smiley:-)` may result in the string `=D`. <br/>
If a key or value must contain a whitespace, it must have `\` before the whitespace. The key or value `two\ words` denotes the string `two words`.
Whitespace characters in the value string will not be ignored. For example, `key=value ` notates that the string `key` will result in the string `value `. <br/>
Values may end their lines with `\` (not followed by a whitespace) to include the next line as part of the same value.
### Comments
An improperties file may contain comment lines beginning with `#` or `!`. Comments may also start in the middle of lines (with the use of previously stated characters), and will continue until the end of the line. <br/>
To use the `#` or `!` character in a key or value, it must be prefixed with `\`. For example, `\#foo=bar\!` denotes that the string `#foo` will result in the string `bar!`. <br/>
Characters in comments are completely ignored by the parser.
### Object Declarations
An improperties file may declare keys to result in objects which contain their own string key to value pairs. <br/>
An object is declared with the object's string key followed by the separator `->` and a newline. The object is then terminated with `--`. <br/>
An object declaration may not contain `=` or `:`. If it must contain those characters, the characters must be prefixed with `\`. For example, the object key `1+1\=2` denotes the string `1+1=2`.
If the `->` separator has a whitespace character before it, it will be ignored and will not count as part of the object's key. <br/>
The last `->` found on an object declaration line will be considered the object declaraction separator, meaning any occurrences of this string within the object key will remain part of the object key. For example, an object declaration line `arrow->key ->` declares an object with the key `arrow->key`. The line `arrow-> ->` declares an object with the key `arrow->`.
Despite not being required, it is convention to indent members of an object forward. Key to value members may be declared as normal within an object, and will become members of said object. <br/>
```
key = value
server ->
  ip = foo
  port = bar
  credentials -> # a nested object
    username = dummy
    password = test
  --
  debug = no
--
```
With this example, `key` results in `value` like usual. However, `server` results in an object in which `server`'s member `ip` would result in `foo`, and `server`'s member `credentials` would result in yet another object with its own key-value pairs. <br/>
Objects can be indexed, with the values at each index (starting from 0) being the corresponding string key. For example, `server`'s 0th item would be the string `ip`, and its 2nd item would be the string `credentials`.
### List declarations
An improperties file may declare keys to result in lists which contain a set of ordered, indexed values.
A list is declared exactly like an object, with the key followed by `->` (optionally with a space before, not counted as part of the key) and a newline following. It is also terminated with `--` like an object. <br/>
Elements in a list, however, must be prefixed with `-`. An object is recognized as a list when its first member begins with a `-`, and does not contain the `:` or `=` separators. This defines the first member as a list element, rather than a key value pair. <br/>
A list element may have a whitespace after the `-`, which will not be counted as part of the element if it is a string. <br/>
If the first character in a list element is `-`, it must be prefixed with `\`. For example, the list element `- \-_-` denotes the string `-_-`.
If a list element is to contain `=` or `:`, those characters must be prefixed with `\` no matter where they occur. For example, the list element `- smiley\ \:-)\ \=D\ face` denotes the string `smiley :-) =D face`. <br/>
A list element may also be an object or list. An object/list list element must be `- ->` or `-->` followed by a newline. Said object/list must be terminated with `--`. <br/>
Object/list declarations containing no elements may be treated as an object or a list interchangeably. They do not have a concrete object/list type, and are simply empty structures.
```
key = value
teacher ->
  name = Joe
  age = 1984
--
days_of_week ->
  - mon
  - tue
  - wed
  - thu
  - fri
  - sat
  - sun
--
students ->
  - ->
    name = Foo
    age = 46
  --
  - ->
    name = Bar
    age = 512
  --
--
```
With this example, `key` results in `value` as expected. `teacher`'s member `name` results in `Joe`. <br/>
However `days_of_week` does not contain any keyed members, as it is a list. This makes it purely indexable. `days_of_week`'s 4th element results in `fri`. <br/>
`students` is also a list, however its elements are objects. `students`'s 0th element's member `age` results in `46`.

# Parser Unit Tests
Below are some unit tests that can be run on a parser, to ensure it complies with the Improperties specification. <br/>
### Unit Testing File:
```
# comment1
! comment2

key=value # comment3

eq1 = eqvalue1
eq2=eqvalue2
eq3 =eqvalue3
eq4= eqvalue4

col1 : colvalue1
col2:colvalue2
col3 :colvalue3
col4: colvalue4

mlalpha = a\
  b\
  c\
  d\

wsp1\ =wspvalue1
wsp2\  = wspvalue2
wsp\ 3 = wspvalue3
wsp4 = wsp\ value\ 4

\=bseq1\= = bseqvalue1
\=bseq2\= : bseqvalue2
\:bscol1\: = bscolvalue1
\:bscol2\: : bscolvalue2

-hyp1 = hypvalue1
--hyp2 = hypvalue2

-> = arrvalue

object1->
  key=value
--

object2 ->
  key=value
--

objempty ->
--

nestobj ->
  nested ->
    key=value
  --
  nested2 ->
    - elem0
    - elem1
    - elem2
  --
--

list1->
  - a
  - b
--

list2 ->
  - elem0
  -elem1
  - \-_-
  -\-O-
  - ->
  --
  -->
  --
  - \->
--

nestlist ->
  - ->
    key=value
  --
  - ->
    key=value2
  --
  - ->
    - elem0
    - elem1
    - elem2
  --
--

objwith->arr ->
--

objwith-> ->
--
```
<br/>

### Testing the File

When interfacing with a parsed representation of the above file:

- The key `# comment1` must result in the indication that such a key is undefined.
- The key `! comment2` must result in the indication that such a key is undefined.
- The key `key` must result in the value `value`.
- The key `eq1` must result in the value `eqvalue1`.
- The key `eq2` must result in the value `eqvalue2`.
- The key `eq3` must result in the value `eqvalue3`.
- The key `eq4` must result in the value `eqvalue4`.
- The key `col1` must result in the value `colvalue1`.
- The key `col2` must result in the value `colvalue2`.
- The key `col3` must result in the value `colvalue3`.
- The key `col4` must result in the value `colvalue4`.
- The key `mlalpha` must result in the value `abcd`.
- The key `wsp1 ` must result in the value `wspvalue1`.
- The key `wsp2 ` must result in the value `wspvalue2`.
- The key `wsp 3` must result in the value `wspvalue3`.
- The key `wsp4` must result in the value `wsp value 4`.
- The key `=bseq1=` must result in the value `bseqvalue1`.
- The key `=bseq2=` must result in the value `bseqvalue2`.
- The key `:bscol1:` must result in the value `bscolvalue1`.
- The key `:bscol2:` must result in the value `bscolvalue2`.
- The key `-hyp1` must result in the value `hypvalue1`.
- The key `--hyp2` must result in the value `hypvalue2`.
- The key `->` must result in the value `arrvalue`.
- The key `object1` must result in an object, whose key `key` must result in the value `value`.
- The key `object2` must result in an object, whose key `key` must result in the value `value`.
- The key `objempty` must result in an empty structure that may be interchangeably accessed as an object or list.
- The key `nestobj` must result in an object, whose key `nested` must result in another object, whose key `key` must result in the value `value`.
- The key `nestobj` must result in an object, whose key `nested2` must result in a list, whose:
  - `0` element must result in the value `elem0`,
  - `1` element must result in the value `elem1`,
  - `2` element must result in the value `elem2`
- The key `list1` must result in a list, whose:
  - `0` element must result in the value `a`,
  - `1` element must result in the value `b`
- The key `list2` must result in a list, whose:
  - `0` element must result in the value `elem0`,
  - `1` element must result in the value `elem1`,
  - `2` element must result in the value `-_-`,
  - `3` element must result in the value `-O-`,
  - `4` element must result in an empty structure that may be interchangeably accessed as an object or list,
  - `5` element must result in an empty structure that may be interchangeably accessed as an object or list,
  - `6` element must result in the value `->`
- The key `nestlist` must result in a list, whose:
  - `0` element must result in an object, whose key `key` must result in the value `value`,
  - `1` element must result in an object, whose key `key` must result in the value `value2`,
  - `2` element must result in a list, whose:
    - `0` element must result in the value `elem0`,
    - `1` element must result in in the value `elem1`,
    - `2` element must result in in the value `elem2`
- The key `objwith->arr` must result in an empty structure that may be interchangeably accessed as an object or list.
- The key `objwith->` must result in an empty structure that may be interchangeably accessed as an object or list.
