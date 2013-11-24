# NAME

Devel::PartialDump - Partial dumping of data structures, optimized for argument printing.

# VERSION

version 0.16

# SYNOPSIS

    use Devel::PartialDump;

    sub foo {
        print "foo called with args: " . Devel::PartialDump->new->dump(@_);
    }

    use Devel::PartialDump qw(warn);

    # warn is overloaded to create a concise dump instead of stringifying $some_bad_data
    warn "this made a boo boo: ", $some_bad_data

# DESCRIPTION

This module is a data dumper optimized for logging of arbitrary parameters.

It attempts to truncate overly verbose data, in a way that is hopefully more
useful for diagnostics warnings than

    warn Dumper(@stuff);

Unlike other data dumping modules there are no attempts at correctness or cross
referencing, this is only meant to provide a slightly deeper look into the data
in question.

There is a default recursion limit, and a default truncation of long lists, and
the dump is formatted on one line (new lines in strings are escaped), to aid in
readability.

You can enable it temporarily by importing functions like `warn`, `croak` etc
to get more informative errors during development, or even use it as:

    BEGIN { local $@; eval "use Devel::PartialDump qw(...)" }

to get DWIM formatting only if it's installed, without introducing a
dependency.

# SAMPLE OUTPUT

- `"foo"`

        "foo"
- `"foo" => "bar"`

        foo: "bar"
- `foo => "bar", gorch => [ 1, "bah" ]`

        foo: "bar", gorch: [ 1, "bah" ]
- `[ { foo => ["bar"] } ]`

        [ { foo: ARRAY(0x9b265d0) } ]
- `[ 1 .. 10 ]`

        [ 1, 2, 3, 4, 5, 6, ... ]
- `"foo\nbar"`

        "foo\nbar"
- `"foo" . chr(1)`

        "foo\x{1}"

# ATTRIBUTES

- max\_length

    The maximum character length of the dump.

    Anything bigger than this will be truncated.

    Not defined by default.

- max\_elements

    The maximum number of elements (array elements or pairs in a hash) to print.

    Defaults to 6.

- max\_depth

    The maximum level of recursion.

    Defaults to 2.

- stringify

    Whether or not to let objects stringify themselves, instead of using
    ["StrVal" in overload](http://search.cpan.org/perldoc?overload#StrVal) to avoid side effects.

    Defaults to false (no overloading).

- pairs

    Whether or not to autodetect named args as pairs in the main `dump` function.
    If this attribute is true, and the top level value list is even sized, and
    every odd element is not a reference, then it will dumped as pairs instead of a
    list.

# EXPORTS

All exports are optional, nothing is exported by default.

This module uses [Sub::Exporter](http://search.cpan.org/perldoc?Sub::Exporter), so exports can be renamed, curried, etc.

- warn
- show
- show\_scalar
- croak
- carp
- confess
- cluck
- dump

    See the various methods for behavior documentation.

    These methods will use `$Devel::PartialDump::default_dumper` as the invocant if the
    first argument is not blessed and `isa` [Devel::PartialDump](http://search.cpan.org/perldoc?Devel::PartialDump), so they can be
    used as functions too.

    Particularly `warn` can be used as a drop in replacement for the built in
    warn:

        warn "blah blah: ", $some_data;

    by importing

        use Devel::PartialDump qw(warn);

    `$some_data` will be have some of it's data dumped.

- $default\_dumper

    The default dumper object to use for export style calls.

    Can be assigned to to alter behavior globally.

    This is generally useful when using the `warn` export as a drop in replacement
    for `CORE::warn`.

# METHODS

- warn @blah

    A wrapper for `dump` that prints strings plainly.

- show @blah
- show\_scalar $x

    Like `warn`, but instead of returning the value from `warn` it returns its
    arguments, so it can be used in the middle of an expression.

    Note that

        my $x = show foo();

    will actually evaluate `foo` in list context, so if you only want to dump a
    single element and retain scalar context use

        my $x = show_scalar foo();

    which has a prototype of `$` (as opposed to taking a list).

    This is similar to the venerable Ingy's fabulous and amazing [XXX](http://search.cpan.org/perldoc?XXX) module.

- carp
- croak
- confess
- cluck

    Drop in replacements for [Carp](http://search.cpan.org/perldoc?Carp) exports, that format their arguments like
    `warn`.

- dump @stuff

    Returns a one line, human readable, concise dump of @stuff.

    If called in void context, will `warn` with the dump.

    Truncates the dump according to `max_length` if specified.

- dump\_as\_list $depth, @stuff
- dump\_as\_pairs $depth, @stuff

    Dump `@stuff` using the various formatting functions.

    Dump as pairs returns comma delimited pairs with `=>` between the key and the value.

    Dump as list returns a comma delimited dump of the values.

- format $depth, $value
- format\_key $depth, $key
- format\_object $depth, $object
- format\_ref $depth, $Ref
- format\_array $depth, $array\_ref
- format\_hash $depth, $hash\_ref
- format\_undef $depth, undef
- format\_string $depth, $string
- format\_number $depth, $number
- quote $string

    The various formatting methods.

    You can override these to provide a custom format.

    `format_array` and `format_hash` recurse with `$depth + 1` into
    `dump_as_list` and `dump_as_pairs` respectively.

    `format_ref` delegates to `format_array` and `format_hash` and does the
    `max_depth` tracking. It will simply stringify the ref if the recursion limit
    has been reached.

# AUTHOR

יובל קוג'מן (Yuval Kogman) <nothingmuch@woobling.org>

# COPYRIGHT AND LICENSE

This software is copyright (c) 2008 by יובל קוג'מן (Yuval Kogman).

This is free software; you can redistribute it and/or modify it under
the same terms as the Perl 5 programming language system itself.

# CONTRIBUTORS

- David Golden <dagolden@cpan.org>
- Florian Ragwitz <rafl@debian.org>
- Jesse Luehrs <doy@tozt.net>
- Karen Etheridge <ether@cpan.org>
- Leo Lapworth <web@web-teams-computer.local>
