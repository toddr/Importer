# NAME

Importer - Alternative but compatible interface to modules that export symbols.

# DESCRIPTION

This module acts as a layer between [Exporter](https://metacpan.org/pod/Exporter) and modules which consume
exports. It is feature-compatible with [Exporter](https://metacpan.org/pod/Exporter), plus some much needed
extras. You can use this to import symbols from any exporter that follows
[Exporters](https://metacpan.org/pod/Exporters) specification. The exporter modules themselves do not need to use
or inherit from the [Exporter](https://metacpan.org/pod/Exporter) module, they just need to set `@EXPORT` and/or
other variables.

# \*\*\* EXPERIMENTAL \*\*\*

This module is still experimental. Anything can change at any time. Testing is
currently VERY insufficient.

# SYNOPSYS

    # Import defaults
    use Importer 'Some::Module';

    # Import a list
    use Importer 'Another::Module' => qw/foo bar baz/;

    # Import a specific version:
    use Importer 'That::Module' => '1.00';

    # Require a sepcific version of Importer
    use Importer 0.001, 'Foo::Bar' => qw/a b c/;

    foo()
    bar()
    baz()

    # Remove all subroutines imported by Importer
    no Importer;

# WHY?

There was recently a discussion on p5p about adding features to [Exporter](https://metacpan.org/pod/Exporter).
This conversation raised some significant concerns, those are listed here, in
addition to others.

- The burden is on export consumers to specify a version of Exporter

    Adding a feature to [Exporter](https://metacpan.org/pod/Exporter) means that any consumer module that relies on
    the new features must depend on a specific version of [Exporter](https://metacpan.org/pod/Exporter). This seems
    somewhat backwords since [Exporter](https://metacpan.org/pod/Exporter) is used by the module you are importing
    from.

- Exporter.pm is really old/crazy code

    Not much more to say here. It is very old, it is very crazy, and if you break
    it you break EVERYTHING.

- Using a modules import() for exporting makes it hard to give it other purposes

    It is not unusual for a module to want to export symbols and prvide import
    behaviors. It is also not unusual for a consumer to only want 1 or the other.
    Using this module you can import symbols without also getting the `import()`
    side effects.

    In addition, moving forward, modules can specify exports and have a custom
    `import()` without conflating the two. A module can tell you to use Importer
    to get the symbols, and to use the module directly for behaviors. A module
    could also use Importer within its own `import()` method without the need to
    subclass [Exporter](https://metacpan.org/pod/Exporter), or bring in its `import()` method.

- There are other exporter modules on cpan

    This module normally assumes an Exporter uses [Exporter](https://metacpan.org/pod/Exporter), so it looks for the
    variables and method [Exporter](https://metacpan.org/pod/Exporter) expects. However, other exporters on cpan can
    override this using the `IMPORTER_MENU()` hook.

# COMPATABILITY

This module aims for 100% compatabilty with every feature of [Exporter](https://metacpan.org/pod/Exporter), plus
added features such as import renaming.

If you find something that works differently, or not at all when compared to
[Exporter](https://metacpan.org/pod/Exporter) please report it as a bug, unless it is noted as an intentional
feature (like import renaming).

# IMPORT PARAMETERS

    use Importer $IMPORTER_VERSION, $FROM_MODULE, $FROM_MODULE_VERSION, @SYMBOLS;

- $IMPORTER\_VERSION (optional)

    If you provide a numeric argument as the first argument it will be treated as a
    version number. Importer will d a version check to make sure it is at least at
    the requested version.

- $FROM\_MODULE (required)

    This is the only required argument. This is the name of the module to import
    symbols from.

- $FROM\_MODULE\_VERSION (optional)

    Any numeric argument following the `$FROM_MODULE` will be treated as a version
    check against `$FROM_MODULE`.

- @SYMBOLS (optional)

    Symbols you wish to import. If no symbols are specified then the defaults will
    be used.

# SUPPORTED FEATURES

## RENAMING SYMBOLS AT IMPORT

_This is a new feature,_ [Exporter](https://metacpan.org/pod/Exporter) _does not support this on its own._

You can rename symbols at import time using a specification hash following the
import name:

    use Importer 'Some::Thing' => (
        foo => { -as => 'my_foo' },
    );

You can also add a prefix and/or postfix:

    use Importer 'Some::Thing' => (
        foo => { -prefix => 'my_foo' },
    );

Using this syntax to set prefix and/or postfix also works on tags and patterns
that are specified for import, in which case the prefix/postfix is applied to
all symbols from the tag/patterm.

## @EXPORT\_FAIL

Use this to list subs that are not available on all platforms. If someone tries
to import one of these, Importer will hit your ` $from-`export\_fail(@items) >
callback to try to resolve the issue. See [Exporter.pm](https://metacpan.org/pod/Exporter.pm) for documentation of
this feature.

## %EXPORT\_TAGS

This module supports tags exactly the way [Exporter](https://metacpan.org/pod/Exporter) does.

    use Importer 'Some::Thing'  => ':DEFAULT';

    use Importer 'Other::Thing' => ':some_tag';

## /PATTERN/ or qr/PATTERN/

You can import all symbols that match a pattern. The pattern can be supplied a
string starting and ending with '/', or you can provide a `qr/../` reference.

    use Importer 'Some::Thing' => '/oo/';

    use Importer 'Some::Thing' => qr/oo/;

## EXLUDING SYMBOLS

You can exlude symbols by prefixing them with '!'.

    use Importer 'Some::Thing'
        '!foo',         # Exclude one specific symbol
        '!/pattern/',   # Exclude all matching symbols
        '!' => qr/oo/,  # Exclude all that match the following arg
        '!:tag';        # Exclude all in tag

# UNIMPORT PARAMETERS

    no Importer;    # Remove all sub brought in with Importer

    no Importer qw/foo bar/;    # Remove only the specified subs

**Only subs can be unimported**.

**You can only unimport subs imported using Importer**.

# CLASS METHODS

- Importer->import($from)
- Importer->import($from, $version)
- Importer->import($from, @imports)
- Importer->import($from, $from\_version, @imports)
- Importer->import($importer\_version, $from, ...)

    This is the magic behind `use Importer ...`.

- Importer->import\_into($from, $into, @imports)
- Importer->import\_into($from, $level, @imports)

    You can use this to import symbols from `$from` into `$into`. `$into` may
    either be a package name, or a caller level to get the name from.

- Importer->unimport()
- Importer->unimport(@sub\_name)

    This is the magic behind `no Importer ...`.

- Importer->unimport\_from($from, @sub\_names)
- Importer->unimport\_from($level, @sub\_names)

    This lets you remove imported symbols from `$from`. `$from` my be a package
    name, or a caller level.

# USING WITH OTHER EXPORTER IMPLEMENTATIONS

If you want your module to work with Importer, but you use something other than
[Exporter](https://metacpan.org/pod/Exporter) to define your exports, you can make it work be defining the
`IMPORTER_MENU` method in your package. As well other exporters can be updated
to support Importer by putting this sub in your package:

    sub IMPORTER_MENU {
        my $class = shift;
        my ($into, $caller) = @_;

        return (
            export      => \@EXPORT,      # Default exports
            export_ok   => \@EXPORT_OK,   # Other allowed exports
            export_tags => \%EXPORT_TAGS, # Define tags
            export_fail => \@EXPORT_FAIL, # For subs that may not always be available
            generate    => \&GENERATE,    # Sub to generate dynamic exports
        );
    }

    sub GENERATE {
        my $class = shift;

        my ($symbol) = @_;

        ...

        return $ref;
    }

All exports must be listed in either `@EXPORT` or `@EXPORT_OK` to be allowed.
`%EXPORT_TAGS`, `@EXPORT_FAIL`, and `\&GENERATE` are optional.

# SOURCE

The source code repository for symbol can be found at
`http://github.com/exodist/Importer`.

# MAINTAINERS

- Chad Granum &lt;exodist@cpan.org>

# AUTHORS

- Chad Granum &lt;exodist@cpan.org>

# COPYRIGHT

Copyright 2015 Chad Granum &lt;exodist7@gmail.com>.

This program is free software; you can redistribute it and/or
modify it under the same terms as Perl itself.

See `http://dev.perl.org/licenses/`
