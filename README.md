# NAME

Perl::Critic::Policy::InputOutput::ProhibitRepeatedPrints - Build the string, then print it once.

# VERSION

version 0.001

# Perl::Critic::Policy::InputOutput::ProhibitRepeatedPrints

Consecutive prints to the same filehandle should be one print with the string built
first:

```perl
print {*STDERR} "Configuration in $dir\n";      # reported
print {*STDERR} "  copied: $what\n";
print {*STDERR} "  built here: $other\n";

my $said = "Configuration in $dir\n";           # what it means
$said .= "  copied: $what\n";
$said .= "  built here: $other\n";
print {*STDERR} $said;
```

A heredoc usually reads better still, and is what this most often wants to
become:

```
print {*STDERR} <<"SAID";
Configuration in $dir
  copied: $what
  built here: $other
SAID
```

The point is not the number of system calls, though this can matter in a tight loop.
We write code to be read by other people, and reducing the amount of information
per-line respects the reader's time.

## What counts as the same handle

Three forms, and they are only a run when the handle matches:

```
print "a\n";               # STDOUT, implicitly
print {*STDERR} "b\n";     # a block
print $fh "c\n";           # a lexical handle
```

So a print to STDOUT directly after one to STDERR is two messages rather than a
run, and is not reported.  `printf` and `say` count alongside `print`: they
write to the same handle and the string can be built ahead of them just the
same.

## PROHIBITED

```
print "one\n";
print "two\n";

printf "%s\n", $a;
print "and\n";

print {*STDERR} "one\n";
print {*STDERR} "two\n" if $why;
```

## ALLOWED

```perl
print "one\n";
print {*STDERR} "two\n";        # a different handle

print "one\n";
$thing->do_something;
print "two\n";                  # not consecutive

print "$_\n" for @lines;        # one statement, however many lines
```

A statement modifier does not split a run -- the second print above is still
part of one -- but a `for` modifier makes a single statement that prints many
times, which is already the thing this policy asks for and cannot be built into
a string without a `join`.

# CONFIGURATION

## minimum\_violations

How many consecutive prints to the same handle are too many.  Two by default;
raise it where a short run is house style.

```
[InputOutput::ProhibitRepeatedPrints]
minimum_violations = 3
```

## allow

Handles never reported however many times they are printed to in a row, spelled
the way the source spells them: `{*STDERR}` with its braces, `$fh` with its
sigil.  Empty by default.

```
[InputOutput::ProhibitRepeatedPrints]
allow = {*STDERR}
```

# CAVEATS

Two consecutive prints are not always one message.  Example:

```perl
my $file = write_thing()
print "Wrote $file\n";

print "Defining and starting $domain...\n";
...
```

The first concludes one step and the second announces the next.  Joining them
would say something the code does not mean, and a blank line between two prints
is not something the policy can read an intention from -- a run of lines that
really is one message often has one too.

There is no rule here that would tell those apart, so where this happens the
answer is `## no critic (InputOutput::ProhibitRepeatedPrints)` with a reason,
or the handle in `allow`, rather than a rewrite that makes the code worse to
satisfy a policy.

## METHODS

What [Perl::Critic::Policy](https://metacpan.org/pod/Perl%3A%3ACritic%3A%3APolicy) asks of a policy, answered here rather than called
from anywhere.

### supported\_parameters

`minimum_violations`, how many consecutive prints to one handle are too many,
and `allow`, the handles never reported however many times they are printed to
in a row.  Both are described under ["CONFIGURATION"](#configuration).

# BUGS

Please report any bugs or feature requests on the bugtracker website
[https://github.com/teodesian/perl-critic-policy-prohibitrepeatedprints/issues](https://github.com/teodesian/perl-critic-policy-prohibitrepeatedprints/issues)

When submitting a bug or request, please include a test-file or a
patch to an existing test-file that illustrates the bug or desired
feature.

# AUTHORS

Current Maintainers:

- George S. Baugh <george@troglodyne.net>

# COPYRIGHT AND LICENSE

Copyright (c) 2026 Troglodyne LLC

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:
The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
