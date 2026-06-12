---
name: Nushell
contributors:
    - ["Joshua Nussbaum", "https://www.github.com/0x4D5352"]
filename: learn_nu.nu
translators:
---

Nushell is a cross-platform shell powered by a similarly-named programming
language (Nu) that treats structured data as a first-class citizen. Written in
Rust, Nushell takes inspiration from its mother language, as well as PowerShell,
TypeScript, and functional languages like Clojure or Haskell. Nu can be used as
a shell or as a scripting language; Many use Nu and Nushell interchangably, or
use Nushell when talking about it as a shell and Nu when talking about it as a
language. For the purposes of this tutorial, the latter will be used.

```nu
#!/usr/bin/env nu
# Much like other shells, nu scripts can be initialized with a shebang:
# https://en.wikipedia.org/wiki/Shebang_(Unix)

# The hash symbol prefixes comments, as you'd expect.

####################################################
## Nu as a shell - Coming from Bash/Zsh
####################################################

# In many ways, Nushell looks like a traditional shell, with commands and args:
echo "Hello world!" # => Hello world!

# However, in many ways it is not:
"Hello world!" # => Hello world!

# Commands are separated by newlines or semicolons, like you'd expect:
mkdir learning_nu; cd learning_nu; touch hello; touch world; touch foo

# Most common shell commands will operate as you expect, as nushell exposes the
# Rust coreutils/uutils library as part of its builtins:
pwd # => ~/learning_nu or /Users/USER/learning_nu, etc.

# However, Nushell diverges when output can be more than just text:
ls
# ╭───┬───────┬──────┬──────┬──────────╮
# │ # │ name  │ type │ size │ modified │
# ├───┼───────┼──────┼──────┼──────────┤
# │ 0 │ foo   │ file │  0 B │ now      │
# │ 1 │ hello │ file │  0 B │ now      │
# │ 2 │ world │ file │  0 B │ now      │
# ╰───┴───────┴──────┴──────┴──────────╯

# This is a table - the primary representation of all structured data in Nushell.
# The layout should look familiar to anyone who has used a shell more than once:
bash -c "ls -lh"
# total 0
# -rw-r--r--@ 1 mussar  staff     0B Feb  7 13:29 foo
# -rw-r--r--@ 1 mussar  staff     0B Feb  7 13:29 hello
# -rw-r--r--@ 1 mussar  staff     0B Feb  7 13:29 world

# Nushell's 'ls' command has a long format as well (you may have to scroll ->):
ls -l
# ╭─────┬─────────┬────────┬──────────┬────────────┬─────────────┬─────────────┬──────────────┬──────────┬─────────┬───────┬──────────┬───────────┬───────────╮
# │   # │  name   │  type  │  target  │  readonly  │    mode     │  num_links  │    inode     │   user   │  group  │ size  │ created  │ accessed  │ modified  │
# ├─────┼─────────┼────────┼──────────┼────────────┼─────────────┼─────────────┼──────────────┼──────────┼─────────┼───────┼──────────┼───────────┼───────────┤
# │   0 │ foo     │ file   │          │ false      │ rw-r--r--   │           1 │   1166959052 │ mussar   │ staff   │   0 B │ now      │ now       │ now       │
# │   1 │ hello   │ file   │          │ false      │ rw-r--r--   │           1 │   1166959050 │ mussar   │ staff   │   0 B │ now      │ now       │ now       │
# │   2 │ world   │ file   │          │ false      │ rw-r--r--   │           1 │   1166959051 │ mussar   │ staff   │   0 B │ now      │ now       │ now       │
# ╰─────┴─────────┴────────┴──────────┴────────────┴─────────────┴─────────────┴──────────────┴──────────┴─────────┴───────┴──────────┴───────────┴───────────╯

# Much like other shells, Nushell allows you to create aliases for command replacement:

alias ll = ls -l # now, calling 'll' will produce the same output as above

# Any time a command can produce columnar output, the Nushell equivalent will
# produce a table. If you want to use a command that has been replaced by a
# Nushell built-in command, prefix it with a caret (^), which signifies that
# you are calling an external command (referred to in nushell as 'extern'):
^ls # => foo     hello   world
^ls -l
# total 0
# -rw-r--r--@ 1 mussar  staff  0 Feb  7 13:40 foo
# -rw-r--r--@ 1 mussar  staff  0 Feb  7 13:40 hello
# -rw-r--r--@ 1 mussar  staff  0 Feb  7 13:40 world

# The power of structured data in your shell comes into play once you begin
# working with pipelines. Let's look at the list of running processes:
ps
# ╭─────┬───────┬───────┬──────────────────┬─────────┬────────┬──────────┬──────────╮
# │   # │  pid  │ ppid  │       name       │ status  │  cpu   │   mem    │ virtual  │
# ├─────┼───────┼───────┼──────────────────┼─────────┼────────┼──────────┼──────────┤
# │   0 │ 27026 │ 11174 │ nu               │ Running │   4.58 │  59.9 MB │ 445.8 GB │
# │   1 │ 27023 │     1 │ mdworker_shared  │ Sleep   │   0.00 │  22.6 MB │ 445.9 GB │
# │     │       │       │ more rows here...│         │        │          │          │
# │ 546 │   620 │     1 │ distnoted        │ Sleep   │   0.00 │  11.7 MB │ 445.8 GB │
# │ 547 │   440 │     1 │ loginwindow      │ Sleep   │   0.00 │ 157.5 MB │ 446.1 GB │
# ├─────┼───────┼───────┼──────────────────┼─────────┼────────┼──────────┼──────────┤
# │   # │  pid  │ ppid  │       name       │ status  │  cpu   │   mem    │ virtual  │
# ╰─────┴───────┴───────┴──────────────────┴─────────┴────────┴──────────┴──────────╯

# Much like POSIX shells pass strings between pipes (OUT | IN), Nushell passes
# strings between pipes. Let's use the 'sort-by' command with the 'mem' column:
ps | sort-by cpu
# ╭─────┬───────┬───────┬────────────────────────────────┬─────────┬───────┬──────────┬──────────╮
# │   # │  pid  │ ppid  │              name              │ status  │  cpu  │   mem    │ virtual  │
# ├─────┼───────┼───────┼────────────────────────────────┼─────────┼───────┼──────────┼──────────┤
# │   0 │ 27763 │     1 │ mdworker_shared                │ Sleep   │  0.00 │  22.4 MB │ 445.9 GB │
# │   1 │ 27727 │     1 │ mdworker_shared                │ Sleep   │  0.00 │  22.6 MB │ 445.9 GB │
# │   2 │ 27628 │     1 │ AudiovisualThumbnailExtension  │ Sleep   │  0.00 │  22.2 MB │ 445.9 GB │
# │     │       │       │ more rows here...              │         │       │          │          │
# │ 547 │   651 │     1 │ WindowManager                  │ Sleep   │ 17.79 │  45.9 MB │ 445.8 GB │
# │ 548 │  1159 │     1 │ ghostty                        │ Sleep   │ 24.80 │ 229.0 MB │ 446.6 GB │
# ├─────┼───────┼───────┼────────────────────────────────┼─────────┼───────┼──────────┼──────────┤
# │   # │  pid  │ ppid  │              name              │ status  │  cpu  │   mem    │ virtual  │
# ╰─────┴───────┴───────┴────────────────────────────────┴─────────┴───────┴──────────┴──────────╯

# We can pass flags to these filter commands like any other, and chain multiple
# filters together to whittle down to the specific results we want:
 ps | sort-by cpu --reverse | where status == Running and cpu > 5
# ╭───┬───────┬───────┬───────────────────────────┬─────────┬──────┬──────────┬──────────╮
# │ # │  pid  │ ppid  │           name            │ status  │ cpu  │   mem    │ virtual  │
# ├───┼───────┼───────┼───────────────────────────┼─────────┼──────┼──────────┼──────────┤
# │ 0 │ 27026 │ 11174 │ nu                        │ Running │ 6.44 │  73.9 MB │ 445.8 GB │
# │ 1 │ 17817 │ 17814 │ Element Helper (Renderer) │ Running │ 5.29 │ 535.9 MB │   1.9 TB │
# ╰───┴───────┴───────┴───────────────────────────┴─────────┴──────┴──────────┴──────────╯

# You can access the values of specific rows and columns using cell-paths,
# accesed via dot notation. Nu uses the special variable $in to reference the
# output of the previous pipe:

ps | sort-by cpu --reverse | where status == Running and cpu > 5 | $in.0.name
# => nu

# if a row/column might be null, you can specify it as an optional value using
# the question mark (?) postfix operator, as in this contrived example:
ps | sort-by cpu --reverse | where status == Running and cpu > 5 | first 1 | $in.1?.name
# returns `null`; without the question mark the above would raise an error.

# a more real-world example:
http get https://api.restful-api.dev/objects | get data.Price
# Error: nu::shell::column_not_found
#
#   × Cannot find column 'Price'
#    ╭─[entry #4:1:10]
#  1 │ http get https://api.restful-api.dev/objects | get data.Price
#    ·          ─────────────────┬─────────────────            ──┬──
#    ·                           │                               ╰── cannot find column 'Price'
#    ·                           ╰── value originates here
#    ╰────
#
http get https://api.restful-api.dev/objects | get data.Price?
# ╭────┬────────╮
# │  0 │        │
# │  1 │        │
# │  2 │        │
# │  3 │        │
# │  4 │        │
# │  5 │        │
# │  6 │        │
# │  7 │        │
# │  8 │        │
# │  9 │        │
# │ 10 │        │
# │ 11 │ 419.99 │
# │ 12 │ 519.99 │
# ╰────┴────────╯

# As a functional language, Nu supports all of the familiar functional programming
# concepts. Let's perform a map operation on every item that appears in the table
# shown above where we print the value for each corow using the 'each' command.
# To use the 'each' command, you provide a command sequence wrapped in a closure ({})
ps | sort-by cpu --reverse | where status == Running and cpu > 5 | each { echo $in }
# ╭───┬───────┬───────┬───────────────────────────┬─────────┬───────┬──────────┬──────────╮
# │ # │  pid  │ ppid  │           name            │ status  │  cpu  │   mem    │ virtual  │
# ├───┼───────┼───────┼───────────────────────────┼─────────┼───────┼──────────┼──────────┤
# │ 0 │ 17817 │ 17814 │ Element Helper (Renderer) │ Running │ 16.78 │ 541.8 MB │   1.9 TB │
# │ 1 │ 28173 │ 11174 │ nu                        │ Running │  5.90 │  78.5 MB │ 445.8 GB │
# ╰───┴───────┴───────┴───────────────────────────┴─────────┴───────┴──────────┴──────────╯

# You may have noticed that the result of this operation is not a pair of strings,
# but a list of items corresponding to the rows from the table produced by 'ps'.
# This is because, like other functional languages, every command in Nu has a
# return value, including 'echo'. Nushell's echo command is an identity function,
# where it simply returns whatever was passed to it as an argument. We will discuss
# lists in the next section, but if you want behavior similar to what you expect
# with POSIX echo, you would utilize 'print', which prints the value it is passed
# as an argument to STDOUT - even if that value is not a string:
ps
| sort-by cpu --reverse                 # in Nushell, pipelines and closures
| where status == Running and cpu > 5   # can be spread out over multiple lines
| each {                                # for better readbility.
    print $in
}
# ╭─────────┬───────────────────────────╮
# │ pid     │ 17817                     │
# │ ppid    │ 17814                     │
# │ name    │ Element Helper (Renderer) │
# │ status  │ Running                   │
# │ cpu     │ 11.21                     │
# │ mem     │ 541.2 MB                  │
# │ virtual │ 1.9 TB                    │
# ╰─────────┴───────────────────────────╯
# ╭─────────┬──────────╮
# │ pid     │ 28173    │
# │ ppid    │ 11174    │
# │ name    │ nu       │
# │ status  │ Running  │
# │ cpu     │ 5.77     │
# │ mem     │ 78.8 MB  │
# │ virtual │ 445.8 GB │
# ╰─────────┴──────────╯
# ╭────────────╮
# │ empty list │
# ╰────────────╯

# The empty list at the end is due to the fact that 'each' will return the value
# for each iteration of its execution, but 'print' explicitly does NOT return
# any value. You can learn more about this behavior, or any other Nushell command,
# by calling it with the '-h' or '--help' command, or by passing it as an arugment
# to the 'help' command:
help print
# Print the given values to stdout.
#
# Unlike echo, this command does not return any value (print | describe will return "nothing").
# Since this command has no output, there is no point in piping it with other commands.
#
# print may be used inside blocks of code (e.g.: hooks) to display text during execution without interfering with the pipeline.
# ... rest of the help display truncated for space.

# If you make a mistake, Nushell attempts to be as helpful as possible with its
# error messaging, taking inspiration the format of the Rust compiler:
ehco "oops i made a mistake"
# Error: nu::shell::external_command
#
#   × External command failed
#    ╭─[entry #121:1:1]
#  1 │ ehco "oops i made a mistake"
#    · ──┬─
#    ·   ╰── Command `ehco` not found
#    ╰────
#   help: `ehco` is neither a Nushell built-in or a known external command

# The last thing to note before we move on is that some symbols you may expect
# to work from a POSIX shell do not, as Nushell made a deliberate choice to NOT
# adhere to POSIX compliance. For example, the < and > symbols are interpreted
# by Nu as the less than and greater than comparison operators, respectively.
# If you wish to save the result of a command or pipeline to a file, use 'save':
echo 'test' | save test_1.txt; cat test_1.txt # => test

# This will not work with 'print', as that prints directly to stdout and 'save'
# operates on the input from the pipeline. To handle stdout and stderr, use the
# out> or o>, err> or e>, or out+err> or o+e> redirection operators.

cat test_1.txt o> test_2.txt e> err.log
cat test_1.txt test_2.txt does_not_exist.txt out+err> combined.txt

# For more information, check out https://www.nushell.sh/book/thinking_in_nu.html


####################################################
## Nu as a Language, Pt. 1: Datatypes and Operators
####################################################

# Of course, Nu has numbers:
3 # => 3

# Nu can do math on those numbers:
1 + 1       # => 2
420 - 69    # => 351
12 * 34     # => 408
3 / 4       # => 0.75

# Integers can accept bitwise operations:
5 bit-xor 2     # => 7
7 bit-xor 2     # => 5
2 bit-shl 3     # => 16

# Floor division rounds towards negative infinity:
5 // 3      # => 1
-5 // 3     # => -1
5.0 // 3.0  # => 1.0  # It works on floats, too!
-5.0 // 3.0 # => -2.0

# The result of division is always a float:
3 / 1       # => 3.0
3.0 / 1     # => 3.0

# Infinity is also a float:
Infinity    # => inf
# the 'describe' command gives the type and structure of the previous pipe:
Infinity
| describe  # => float

# Modulo operations use the 'mod' operator:
7 mod 3     # => 1
# i mod j have the same sign as j:
-7 mod 3    # => 2

# Exponentiation (x ** y, x to the yth power):
2 ** 3      # => 8

# Enforce precedence with parentheses:
1 + 3 * 2   # => 7
(1 + 3) * 2 # => 8

# Binary and Hexidecimal are just integers:
0xDEADBEEF  # => 3735928559
0b10101001  # => 169

# Unless you enclose them in brackets, which makes them binary
0x[DEADBEEF]    # 0x[DE AD BE EF] would work as well
# Length: 4 (0x4) bytes | printable whitespace ascii_other non_ascii
# 00000000:   de ad be ef
0b[10101001]    # 0b[1 0 1 0 1 0 0 1] spaces are ignored in brackets
# Length: 1 (0x1) bytes | printable whitespace ascii_other non_ascii
# 00000000:   a9

# Octal works for binary, too!
0o[127]         # 0o[1      2       7] use however many you wish!
# Length: 1 (0x1) bytes | printable whitespace ascii_other non_ascii
# 00000000:   57

# Nu also has number-like things, such as dates:
2001-09-11      # => Tue, 11 Sep 2001 00:00:00 +0000 (24 years ago)

# You can do subtraction on dates, which gives you durations:
2026-02-07 - 2026-01-01 # => 5wk 2day

# Or do addition/subtraction on dates with durations:
2026-02-07 + 1day       # => Sun, 8 Feb 2026 00:00:00 +0000 (in 37 minutes)
2026-02-07 - 52wk       # => Sat, 8 Feb 2025 00:00:00 +0000 (a year ago)

# Note: currently, only durations with a fixed length are supported.

# Nu also supports file-sizes and mathematical operations upon them:
1mb             # => 1.0 MB
1mb - 500kb     # => 500.0 kB
1mb * 2         # => 2.0 MB
1mb mod 1.3kb   # => 300 B

# Boolean values are primitives (note the casing; True/False are byte streams):
true        # => true
false       # => false

# Negate with not:
not true    # => false
not false   # => true

# Boolean Operators (note the casing):
true or false    # => true
true and false   # => false

# Most comparison operators work on most datatypes:
true == true            # => true
1 != 1                  # => false
2026-02-07 > 2026-02-05 # => true
1mb <= 1kb              # => false
1wk >= (7day)           # => true
0xF <= 0b1011           # => false
0x[F] == 0b[1111]       # => true
0o[7] != 0x[7]          # => false

# ... but not all:
0x[F] <= 0b[01]
# Error: nu::parser::operator_unsupported_type
#
#   × The '<=' operator does not work on values of type 'binary'.
#    ╭─[entry #113:1:1]
#  1 │ 0x[F] <= 0b[01]
#    · ──┬── ─┬
#    ·   │    ╰── does not support 'binary'
#    ·   ╰── binary
#    ╰────


# Strings can come in many forms; single quotes cannot use escapes and cannot
# contain single quotes within the string:
'hello world'       # => hello world

# All string types (but one) accept and preserve newlines:
'hello
dolly'
# => hello
# => dolly

# Double quotes allow for C-style backslash escapes, and as such must escape
# any literal backslashes:
"The\nEnd"
# => The
# => End
"Look, a backslash: \\" # => Look, a backslash: \

# If not a reserved keyword or in the command position, i.e. the first word after
# a newline or semicolon, nushell allows 'bare word' strings:
print hello     # => hello

# String interpolation allows you to perform f-string style variable substitution:

let foo = 'world' # more on variables in Pt. 2.
$'hello, ($foo)'    # => hello, world
$"goodbye,\n($foo)"
# => goodbye,
# => world

# Strings surrounded by backticks (`) are interpreted as if they were bare words
# on the command line, but capture whitespace:

touch `hello world` # creates a file 'hello world'
rm 'hello world' # in some instances, the command won't require the backticks

# Globs are similar to strings, but function as a tool for working with the
# filesystem as you might expect from a shell language like Bash:

ls *.nu
# ╭───┬─────────────┬──────┬──────┬─────────────╮
# │ # │    name     │ type │ size │  modified   │
# ├───┼─────────────┼──────┼──────┼─────────────┤
# │ 0 │ learn_nu.nu │ file │  0 B │ 7 hours ago │
# ╰───┴─────────────┴──────┴──────┴─────────────╯

# Globs only work as bare word strings or within backtick-quoted strings:

ls '*.nu'
# Error: nu::shell::error
#
#   × No matches found for DoNotExpand("*.nu")
#    ╭─[entry #49:1:4]
#  1 │ ls '*.nu'
#    ·    ───┬──
#    ·       ╰── Pattern, file or folder not found
#    ╰────
#   help: no matches found
ls `*.nu`
# ╭───┬─────────────┬──────┬──────┬─────────────╮
# │ # │    name     │ type │ size │  modified   │
# ├───┼─────────────┼──────┼──────┼─────────────┤
# │ 0 │ learn_nu.nu │ file │  0 B │ 7 hours ago │
# ╰───┴─────────────┴──────┴──────┴─────────────╯

# For more information, see https://www.nushell.sh/book/working_with_strings.html

# Lists are zero-indexed ordered sequences of zero or more values of any type:
[1 2 three true]
# ╭───┬───────╮
# │ 0 │     1 │
# │ 1 │     2 │
# │ 2 │ three │
# │ 3 │ true  │
# ╰───┴───────╯

# Select parts of a list with 'first' or 'last':
[1 2 3 4 5] | first 2
# ╭───┬───╮
# │ 0 │ 1 │
# │ 1 │ 2 │
# ╰───┴───╯
 [1 2 3 4 5] | last 2
# ╭───┬───╮
# │ 0 │ 4 │
# │ 1 │ 5 │
# ╰───┴───╯

# Grab specific elements of a list with 'get':
[1 2 3] | get 1 # => 2

# Create a new list with 'select':
[1 2 3] | select 0 2
# ╭───┬───╮
# │ 0 │ 1 │
# │ 1 │ 3 │
# ╰───┴───╯

# Modify lists with 'insert', 'update', 'prepend', and 'append':
[0 1 2 3 4 5 6] | insert 6 7
# ╭───┬───╮
# │ 0 │ 0 │
# │ 1 │ 1 │
# │ 2 │ 2 │
# │ 3 │ 3 │
# │ 4 │ 4 │
# │ 5 │ 5 │
# │ 6 │ 7 │
# │ 7 │ 6 │
# ╰───┴───╯
[tom dick harry] | update 1 lisa
# ╭───┬───────╮
# │ 0 │ tom   │
# │ 1 │ lisa  │
# │ 2 │ harry │
# ╰───┴───────╯
[1kb 1mb] | prepend 1b | append 1gb
# ╭───┬────────╮
# │ 0 │    1 B │
# │ 1 │ 1.0 kB │
# │ 2 │ 1.0 MB │
# │ 3 │ 1.0 GB │
# ╰───┴────────╯

# Map over lists like you could tables using commands like 'each':
[1 2 3] | each { $in * 2 }
# ╭───┬───╮
# │ 0 │ 2 │
# │ 1 │ 4 │
# │ 2 │ 6 │
# ╰───┴───╯

# Filter using commands like 'where':
[1 2 3] | where { $in mod 2 == 1}
# ╭───┬───╮
# │ 0 │ 1 │
# │ 1 │ 3 │
# ╰───┴───╯

# Reduce using commands like... 'reduce':
 [1 2 3] | reduce {|elt, acc| $acc - $elt }   # => -4
# ((1 - 2) - 3)

# Perform conditional operations with lists using 'in' or 'has':
1 in [1 2 3]    # => true
[1 2 3] has 6   # => false
6 not-in [1 2]  # => true
[3 2] not-has 3 # => false

# Ranges are sequences of numbers separated by two periods, with an optional
# stride value that dictates the step distance from the start to the end:
1..5
# ╭───┬───╮
# │ 0 │ 1 │
# │ 1 │ 2 │
# │ 2 │ 3 │
# │ 3 │ 4 │
# │ 4 │ 5 │
# ╰───┴───╯
2..4..10
# ╭───┬────╮
# │ 0 │  2 │
# │ 1 │  4 │
# │ 2 │  6 │
# │ 3 │  8 │
# │ 4 │ 10 │
# ╰───┴────╯

# Ranges can go backawards:
5..1 
# ╭───┬───╮
# │ 0 │ 5 │
# │ 1 │ 4 │
# │ 2 │ 3 │
# │ 3 │ 2 │
# │ 4 │ 1 │
# ╰───┴───╯

# ... or negative:
1..-4
# ╭───┬────╮
# │ 0 │  1 │
# │ 1 │  0 │
# │ 2 │ -1 │
# │ 3 │ -2 │
# │ 4 │ -3 │
# │ 5 │ -4 │
# ╰───┴────╯

# ... and can even be floats!
(1.0)..(1.2)..(2.0)
# ╭───┬──────╮
# │ 0 │ 1.00 │
# │ 1 │ 1.20 │
# │ 2 │ 1.40 │
# │ 3 │ 1.60 │
# │ 4 │ 1.80 │
# │ 5 │ 2.00 │
# ╰───┴──────╯

# While they look like lists, ranges are a standalone data type and are lazily
# evaluated. They are most often used near the beginning of a pipeline as an
# input to an iterator, or as an argument to a slicing function:

'hello' | str substring 1..3    # => ell

# Records are key-value pairs that associate string keys with datatype values:
{language: nushell began: 2019-08-23 is_cool: true}
# ╭──────────┬─────────────╮
# │ language │ nushell     │
# │ began    │ 6 years ago │
# │ is_cool  │ true        │
# ╰──────────┴─────────────╯

# spaces, commas, and newlines can all delimit key-value pairs:
{name: bob, age: 30, occupation: 'Software Developer'}
# ╭────────────┬────────────────────╮
# │ name       │ bob                │
# │ age        │ 30                 │
# │ occupation │ Software Developer │
# ╰────────────┴────────────────────╯

# Pull values out of records using 'get':
{language: nushell began: 2019-08-23 is_cool: true} | get language  # => nushell

# Create new records using 'select':
{
    language: nushell
    began: 2019-08-23
    is_cool: true
} | select language is_cool
# ╭──────────┬─────────╮
# │ language │ nushell │
# │ is_cool  │ true    │
# ╰──────────┴─────────╯

# Modify records with 'insert' and 'update':
{language: nushell began: 2019-08-23 is_cool: true}
| insert has_xinyminutes { true }
# ╭─────────────────┬─────────────╮
# │ language        │ nushell     │
# │ began           │ 6 years ago │
# │ is_cool         │ true        │
# │ has_xinyminutes │ true        │
# ╰─────────────────┴─────────────╯
{name: bob, age: 30, occupation: 'Software Developer'}
| update occupation { 'Goat Farmer' }
# ╭────────────┬─────────────╮
# │ name       │ bob         │
# │ age        │ 30          │
# │ occupation │ Goat Farmer │
# ╰────────────┴─────────────╯

# Iterate over a record using 'items':
{one: 1, two: 2, three: 3} | items {|key, value| $'($key): ($value)' }

# ╭───┬──────────╮
# │ 0 │ one: 1   │
# │ 1 │ two: 2   │
# │ 2 │ three: 3 │
# ╰───┴──────────╯

# Tables are just lists of records or named lists of lists:
[{name: bob, age: 30}, {name: alice, age: 30}]
[[name, age]; [bob, 30], [alice, 30]]
# both produce the following:
# ╭───┬───────┬─────╮
# │ # │ name  │ age │
# ├───┼───────┼─────┤
# │ 0 │ bob   │  30 │
# │ 1 │ alice │  30 │
# ╰───┴───────┴─────╯

# Cell-paths are data types like any other, except that they refer to an inner
# value within a structured value like a list, record, or table:

[{name: bob, age: 30}, {name: alice, age: 30}] | $in.name
# ╭───┬───────╮
# │ 0 │ bob   │
# │ 1 │ alice │
# ╰───┴───────╯
[{name: bob, age: 30}, {name: alice, age: 30}] | $in.0
# ╭──────┬─────╮
# │ name │ bob │
# │ age  │ 30  │
# ╰──────┴─────╯

# A cell-path can be stored as a variable and used later, using an optional
# leading dollar sign:

let idx = $.1
[{name: bob, age: 30}, {name: alice, age: 30}] | get $idx
# ╭──────┬───────╮
# │ name │ alice │
# │ age  │ 30    │
# ╰──────┴───────╯

# Anonymous functions, also referred to as lambdas, are closures in Nushell.
# They can also be stored as a variable for use later in commands like filters:

let greater_than_five = {|x| $x > 5}
[0 2 4 6 8 10] | where $greater_than_five
# ╭───┬────╮
# │ 0 │  6 │
# │ 1 │  8 │
# │ 2 │ 10 │
# ╰───┴────╯

# See the full list of data types: https://www.nushell.sh/book/types_of_data.html
# See the full list of operators: https://www.nushell.sh/book/operators.html

####################################################
## Nu as a Language, Pt. 2: Variables and Functions
####################################################

# By default, variables in Nu are immutable and declared via 'let':
let x = 2

# Variables are access with the dollar sign prefix, like in Bash:
$x      # => 22

# Operate upon variables as you would the original data:

$x + 2  # => 4
$x * 3  # => 6

# If you attempt to mutate a variable, an error will be thrown:
$x = 3
# Error: nu::parser::assignment_requires_mutable_variable
#
#   × Assignment to an immutable variable.
#    ╭─[entry #196:1:1]
#  1 │ $x = 3
#    · ─┬
#    ·  ╰── needs to be a mutable variable
#    ╰────
#   help: declare the variable with `mut`, or shadow it again with `let`

# To reassign a variable, use 'let' again:
let x = 3; $x   # => 3

# 'mut' is a special keyword that allows you to mutate variables once defined:
mut y = 3
$y      # => 3
$y = 5
$y      # => 5

# However, this is discouraged as Nu is designed to be a functional language,
# and it is preferable to use idioms of the paradigm such as maps and filters to
# operate upon variables. The result of a pipeline is always new data, allowing
# the original data to stay intact. We will discuss 

# Nushell does have a concept of constants, which exist to allow Nu scripts and
# Nushell REPL users to safely evaluate variables that were defined as part of
# the code as written; This is because Nushell behaves slightly differently from
# other interpreted languages like Python or Bash.

let this_file = 'learn_nu.nu';
source $this_file
# Error: nu::parser::error
#
#   × Error: nu::shell::not_a_constant
#   │
#   │   × Not a constant.
#   │    ╭─[entry #7:1:8]
#   │  1 │ source $this_file
#   │    ·        ─────┬────
#   │    ·             ╰── Value is not a parse-time constant
#   │    ╰────
#   │   help: Only a subset of expressions are allowed constants during parsing. Try using the 'const' command or typing the value literally.
#   │
#    ╭─[entry #7:1:8]
#  1 │ source $this_file
#    ·        ─────┬────
#    ·             ╰── Encountered error during parse-time evaluation
#    ╰────

const this_file = './learn_nu.nu'
source $this_file # loads the source code and runs it in the current env context

# If you've ever run a bash script that half-completed before raising an error,
# you know that certain classes of errors can only be caught at runtime due to
# the way in which most interpreted languages handle the evaluation step of the
# REPL or code execuion. Nushell diverges from interpreted languages in this way
# and functions more like a compiled language, in that it both parses AND evaluates
# the entire source code or command line. This enables it to perform static analysis
# that allows error catching and performance optimizations which are difficult to
# achieve in other interpreted lanaguges.

# For more information, see: https://www.nushell.sh/book/how_nushell_code_gets_run.html

# As you saw with the 'items' command, you can define custom variable names inside
# of closures in order to use them throughout the command sequence, even after you
# have mutated the variable throughout the pipeline:

[1 2 3] | each {|x| $x + 2 | $in * $x }
# ╭───┬────╮
# │ 0 │  3 │ # 1 + 2 (3) * 1 = 3
# │ 1 │  8 │ # 2 + 2 (4) * 2 = 8
# │ 2 │ 15 │ # 3 + 2 (5) * 3 = 15
# ╰───┴────╯

# Nu is a gradually typed language, similar to TypeScript. Variables are cast
# to their datatype at assignment, and can be reassigned to new data types as
# they can in Python. However, you can specify a type at declaration to ensure
# type safety when sharing or working on larger projects.

let number: int = 'foo'
# Error: nu::parser::type_mismatch
#
#   × Type mismatch.
#    ╭─[entry #16:1:19]
#  1 │ let number: int = 'foo'
#    ·                   ──┬──
#    ·                     ╰── expected int, found string
#    ╰────

# Type declarations for variables are not common, but as we're about to see, they
# can be quite useful when writing more complex logic.

# For simple pipelines, aliasing the command within parentheses can often be enough:
alias git-log-parse = (git log --oneline | parse '{hash} {commit}')
git-log-parse # when run in a git repo like learnxinyminutes, shows results like:
# ╭───┬──────────┬─────────────────────────────────────────────────────────────────────╮
# │ # │   hash   │                               commit                                │
# ├───┼──────────┼─────────────────────────────────────────────────────────────────────┤
# │ 0 │ 97b60079 │ feat: add nu.md; coming from bash, data and ops, start of variables │
# │ 1 │ 11a924e4 │ [lua/it] Traduzione italiana Lua translation (#5464)                │
# │ 2 │ f87f00d1 │ [rink/en] Add Rink tool (#5297)                                     │
# ╰───┴──────────┴─────────────────────────────────────────────────────────────────────╯

# Howeer, if you require a more complex interaction, you can define your own
# functions, which Nu refers to as custom commands:

def greet [name] {
    $"Hello, ($name)!"
}

greet Joshua        # => "Hello, Joshua!"

# Custom commands, like all other expressions in Nu, implicitly return their
# final value. You can explicitly return using the `return (value)` syntax, but
# this is typically used as part of branching conditional logic, which we will
# discuss in the next section.

# Custom commands can work within pipelines, just like builtins and externs:
greet Joshua | str downcase     # => 'hello, joshua!'

def double [] { each {|num| 2 * $num } }; [1 2 3] | double
# ╭───┬───╮
# │ 0 │ 2 │
# │ 1 │ 4 │
# │ 2 │ 6 │
# ╰───┴───╯

# Custom commands can also have "subcommands" similar to Python or Golang:

def "pow squared" [num] { $num ** 2 }
def "pow cubed"   [num] { $num ** 3 }

2
| pow squared   # => 2 ** 2 = 4
| pow cubed     # => 4 ** 3 = 64

# Parameter definitions are just Nu lists, and can be separated by spaces,
# newlines, or commas. Much like with optional values in structured data,
# parameters can be specified as optional with a question mark postfix:

def pow [num, power?] { $num ** ($power | default 2) }
pow 4           # => 16
pow 4 4         # => 256

# Paramaters themselves are just Nu variable declarations that are bound at
# runtime, and benefit from the same gradual typing as variables:

def safe-pow [
    num: int
    power?: int
] {
    $num ** ($power | default 2)
}

safe-pow 2      # => 4
safe-pow "2" 4
# Error: nu::parser::parse_mismatch
#
#   × Parse mismatch during operation.
#    ╭─[entry #10:1:10]
#  1 │ safe-pow "2" 4
#    ·          ─┬─
#    ·           ╰── expected int
#    ╰────

# Custom commands can also define named flags, with an optional short flag name:
def greet [name: string --age (-a): int] {
    { 
        name: $name
        age: $age   # => even if called with -a, Nu will use the full name
    }
}

# If a flag is defined without any type declarations, it is treated as a switch:

def language [name --favorite] { { name: $name is_favorite: $favorite } }
language nu     # => no flag defaults to false
# ╭─────────────┬───────╮
# │ name        │ nu    │
# │ is_favorite │ false │
# ╰─────────────┴───────╯
language nu --favorite
# ╭─────────────┬──────╮
# │ name        │ nu   │
# │ is_favorite │ true │
# ╰─────────────┴──────╯

# When dealing with lists, records, tables, or custom commands that have an
# arbitrary number of arguments, the collection can be unpacked using the
# spread (...) operator, referred to in Nu as the 'rest' operator.
def multi-greet [...names: string] {
  for $name in $names {
    print $"Hello, ($name)!"
  }
}

multi-greet Tom Dick Harry
# => Hello, Tom!
# => Hello, Dick!
# => Hello, Harry!

# For more information, see https://www.nushell.sh/book/custom_commands.html

####################################################
## Nu as a Language, Pt. 3: Iterables & Control Flow
####################################################

# As discussed before, Nu has commands for the holy trinity of functional
# programming iterators - map ('each'), filter ('where'), and reduce ('reduce').
# Closures and pipelines can also be thought of as control flow:
1..                     # ranges can be unbounded - be careful!
| each {|x| $x * 3 }
| where $it mod 2 == 1  # $it is a special variable for where "row conditions"
| first 10              # setting a bounds, otherwise reduce goes forever
| reduce {|elt, acc| $elt + $acc}
# => 300

# Nu also has two options for conditional operators: 'if' and 'match'.
# If statements work as you expect:

let test = 10
if $test mod 3 == 0 {
    "this won't trigger"
} else if $test < 5 {
    "this also won't trigger"
} else {
    'finally, this triggers'
}

# Match statements are similar to switch statements or other pattern matches:
match $test {
    1 => 'nope',
    5 => 'not this one',
    'pie' => 'yes, you can do this',
    _ => 'this is a catch-all, which other languages might call default'
}

# TODO: do try catch and skip/take while/until

# While generally discouraged, Nu does support loops with 'for' and 'while'.
# These can be helpful in situations where you need a mutable variable, or when
# building custom errors as part of script or module development. For the sake
# of not making this long documentation even longer, I will leave that information
# as an exercise for the reader: https://www.nushell.sh/book/control_flow.html#loops
```

For more information, read the [Nu Book](https://www.nushell.sh/book/) or join
the [Discord community](https://discord.gg/NtAbbGn).
