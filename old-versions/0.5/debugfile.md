# Debugfile format specification

Version 0.5 — 26 January 2023

* [1. Objective and scope][section1]
* [2. Introduction][section2]
* [3. Format basics][section3]
    * [3.1. Overall formatting][section3.1]
    * [3.2. Normalization and whitespace][section3.2]
    * [3.3. Error handling][section3.3]
    * [3.4. Basic structure][section3.4]
    * [3.5. Conditional inclusion][section3.5]
* [4. Directives][section4]
    * [4.1. Identification][section4.1]
        * [`@debugfile`][directive-debugfile]
    * [4.2. Conditional inclusion][section4.2]
        * [`@always`][directive-always]
        * [`@ifdef`][directive-ifdef]
        * [`@ifemu`][directive-ifemu]
        * [`@ifnotdef`][directive-ifnotdef]
        * [`@ifnotemu`][directive-ifnotemu]
        * [`@else`][directive-else]
    * [4.3. Declarations][section4.3]
        * [`@sym`][directive-sym]
        * [`@alias`][directive-alias]
        * [`@var`][directive-var]
        * [`@str`][directive-str]
    * [4.4. Action groups][section4.4]
        * [`@group`][directive-group]
        * [`@endgroup`][directive-endgroup]
    * [4.5. Included files][section4.5]
        * [`@include`][directive-include]
        * [`@symfile`][directive-symfile]
    * [4.6. Configuration][section4.6]
        * [`@radix`][directive-radix]
        * [`@signedness`][directive-signedness]
    * [4.7. Miscellaneous][section4.7]
        * [`@warning`][directive-warning]
        * [`@error`][directive-error]
* [5. Actions][section5]
    * [5.1. Basic action structure][section5.1]
    * [5.2. Syntax overview][section5.2]
    * [5.3. Expressions][section5.3]
        * [Numeric constants][expressions-numbers]
        * [Symbols][expressions-symbols]
        * [Variables][expressions-variables]
        * [Memory accesses][expressions-memory]
        * [Operators][expressions-operators]
        * [Constant expressions][expressions-constant]
        * [Address expressions][expressions-address]
    * [5.4. The condition field][section5.4]
    * [5.5. The address subfield][section5.5]
    * [5.6. Flags][section5.6]
* [6. Commands][section6]
    * [6.1. Basic emulator commands][section6.1]
        * [`break` command][command-break]
        * [`reset` command][command-reset]
        * [`message` command][command-message]
        * [`alert` command][command-alert]
    * [6.2. Action-related commands][section6.2]
        * [`enable` command][command-enable]
        * [`disable` command][command-disable]
        * [`toggle` command][command-toggle]
    * [6.3. State modification commands][section6.3]
        * [`set` command][command-set]
        * [`jump` command][command-jump]
    * [6.4. Control flow commands][section6.4]
        * [`nop` command][command-nop]
        * [`done` command][command-done]
        * [`skip` command][command-skip]
        * [`if` command][command-if]
        * [`else` command][command-else]
* [7. Strings][section7]
    * [7.1. String formatting][section7.1]
    * [7.2. Escape sequences][section7.2]
    * [7.3. Expression escape sequences][section7.3]
    * [7.4. Selection escape sequences][section7.4]
    * [7.5. Character replacement escape sequences][section7.5]
* [8. Example file][section8]
* [9. Specification compliance][section9]
    * [9.1. Partial compliance][section9.1]
    * [9.2. Minimum required features][section9.2]
* [10. Closing remarks][section10]
* [11. Version history][section11]

[section1]: #1-objective-and-scope
[section2]: #2-introduction
[section3]: #3-format-basics
[section3.1]: #31-overall-formatting
[section3.2]: #32-normalization-and-whitespace
[section3.3]: #33-error-handling
[section3.4]: #34-basic-structure
[section3.5]: #35-conditional-inclusion
[section4]: #4-directives
[section4.1]: #41-identification
[section4.2]: #42-conditional-inclusion
[section4.3]: #43-declarations
[section4.4]: #44-action-groups
[section4.5]: #45-included-files
[section4.6]: #46-configuration
[section4.7]: #47-miscellaneous
[section5]: #5-actions
[section5.1]: #51-basic-action-structure
[section5.2]: #52-syntax-overview
[section5.3]: #53-expressions
[section5.4]: #54-the-condition-field
[section5.5]: #55-the-address-subfield
[section5.6]: #56-flags
[section6]: #6-commands
[section6.1]: #61-basic-emulator-commands
[section6.2]: #62-action-related-commands
[section6.3]: #63-state-modification-commands
[section6.4]: #64-control-flow-commands
[section7]: #7-strings
[section7.1]: #71-string-formatting
[section7.2]: #72-escape-sequences
[section7.3]: #73-expression-escape-sequences
[section7.4]: #74-selection-escape-sequences
[section7.5]: #75-character-replacement-escape-sequences
[section8]: #8-example-file
[section9]: #9-specification-compliance
[section9.1]: #91-partial-compliance
[section9.2]: #92-minimum-required-features
[section10]: #10-closing-remarks
[section11]: #11-version-history
[command-alert]: #alert-command
[command-break]: #break-command
[command-disable]: #disable-command
[command-done]: #done-command
[command-else]: #else-command
[command-enable]: #enable-command
[command-if]: #if-command
[command-jump]: #jump-command
[command-message]: #message-command
[command-nop]: #nop-command
[command-reset]: #reset-command
[command-set]: #set-command
[command-skip]: #skip-command
[command-toggle]: #toggle-command
[directive-alias]: #alias
[directive-always]: #always
[directive-debugfile]: #debugfile
[directive-else]: #else
[directive-endgroup]: #endgroup
[directive-error]: #error
[directive-group]: #group
[directive-ifdef]: #ifdef
[directive-ifemu]: #ifemu
[directive-ifnotdef]: #ifnotdef
[directive-ifnotemu]: #ifnotemu
[directive-include]: #include
[directive-radix]: #radix
[directive-signedness]: #signedness
[directive-str]: #str
[directive-sym]: #sym
[directive-symfile]: #symfile
[directive-var]: #var
[directive-warning]: #warning
[expressions-address]: #address-expressions
[expressions-constant]: #constant-expressions
[expressions-memory]: #memory-accesses
[expressions-numbers]: #numeric-constants
[expressions-operators]: #operators
[expressions-symbols]: #symbols
[expressions-variables]: #variables

* * *

## 1. Objective and scope

The present document describes a textual file format that contains information regarding debugging actions to be taken
when executing a Game Boy or Game Boy Color program.
The main intended consumer of this information is the emulator executing the program; while hardware debuggers aren't
explicitly ruled out, they are not the intended audience for this file format.

This specification does not intend to provide an origin for those debugging actions; they may be manually typed up or
generated by some code generation tool (e.g., an assembler or a compiler).

## 2. Introduction

In order to debug Game Boy programs, developers may need to specify a list of debugging actions that the emulator must
take on their behalf.
These actions are simple commands, such as halting execution or printing information to a debug console, that are
taken when a certain condition is true, such as the program reading from or executing a certain memory address.

It therefore becomes necessary to have a common way to specify and persist these actions, so that they can be supplied
to the emulator on start-up (instead of having to manually enter them every time the program is debugged); this also
allows for tools (such as assemblers) to generate these lists of actions, and it enables an easy way of sharing them.

In order to permit the easy generation and editing of these lists of actions, a textual format was chosen for them,
which allows users to perform simple modifications in a text editor.
Therefore, this document describes such a textual format, a _debugfile_, to be used alongside existing metadata file
(such as symfiles) as a debugging aid.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",
"NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119][rfc2119]
when, and only when, they appear in all capitals, as shown here.

[rfc2119]: https://www.rfc-editor.org/rfc/rfc2119

## 3. Format basics

This section describes the basic rules and constraints that make up the debugfile format.
The actual contents are defined in a later section.

### 3.1. Overall formatting

Debugfiles MUST be plain text files, encoded in [UTF-8][rfc3629], without BOM.
(Note that US ASCII, being a subset of UTF-8, is a valid encoding.)
There must be no C0 control characters (that is, characters with codepoints below 32) other than line feeds (codepoint
10), horizontal tabulations ("tabs", codepoint 9) and carriage returns (codepoint 13).

The file is broken down into lines, which are delimited by line feed characters.
The last line of the file MAY end with a line feed character; this MUST NOT cause any differences in interpretation of
the file.

Carriage return characters MAY appear before line feed characters; if they do, they MUST be considered to be part of
the line delimiter.
Carriage return characters MUST NOT be used in any other context; a carriage return character not immediately followed
by a line feed character is an error.

[rfc3629]: https://www.rfc-editor.org/rfc/rfc3629

### 3.2. Normalization and whitespace

Normalization describes a process that eliminates superfluous changes to a debugfile.
While emulators interpreting such files are not required to actually perform normalization, they MUST interpret the
file "as if" normalization had happened — for instance, by ignoring trailing whitespace as if it had been eliminated
by normalization.

If normalization is actually performed, the file provided by the user MUST NOT be overwritten with a normalized
version unless the user explicitly requests this.

Normalization is applied to each line of the file individually, and it involves the following steps:

1. Replace all tab characters with an implementation-defined non-zero number of space characters per tab.
   This number MUST be the same for all tab characters in the file.
   Consecutive tab characters MUST be replaced individually: for instance, if an implementation replaces each tab with
   5 spaces, 3 tab characters in a row must be replaced with 15 spaces.
2. Delete all leading and trailing spaces from the line.
3. If the line is now empty, or if its first character is a semicolon, discard it.
   (A line whose first non-whitespace character is a semicolon is considered a comment.)
   Discarded lines are not considered at all by any further processing of the file: for instance, any reference to the
   "next line" ignores discarded lines, and the requirement for [a `@debugfile` directive][directive-debugfile] in the
   first line of the file (explained in [section 3.4][section3.4]) applies to the first line that hasn't been
   discarded.

### 3.3. Error handling

There are several kinds of errors that can be detected while handling a debugfile.

The most obvious kind of error is a syntax error, which consists of some content in the file that is not conformant to
this specification.
If such an error is detected, the implementation MUST report an error to the user and MUST NOT use the file other than
for further error detection; the error SHOULD tell the user the location of the invalid content and the reason why it
was considered invalid.

If the file references other files (for instance, through [a `@symfile` directive][directive-symfile]), and those
files are not available, the implementation MUST report this to the user; it SHOULD NOT use the contents of the main
file in that case other than for further error detection.
Syntax errors in referenced files MUST be treated like syntax errors in the main file.

Errors other than syntax errors (such as an expression containing a name that is neither an existing symbol nor a
valid variable name) can often be detected when the file is initially read in.
Implementations SHOULD attempt to detect these errors when possible, giving them the same treatment described above
for syntax errors.

It is also possible that some emulator will not implement the full range of functionality described in this
specification, only achieving [partial compliance][section9.1].
If this is detected when the file is initially read in, the emulator SHOULD inform the user that some functionality
specified in the file is not supported; the emulator MAY ignore those actions or directives it doesn't implement and
only follow the ones it does.
The emulator MAY also treat this as a hard error, in which case it MUST report the error to the user on initial
read-in and disregard the data in the file.
Any errors reported due to missing functionality SHOULD indicate which functionality was requested by the file that
the emulator does not implement.

If a portion of a debugfile is excluded by a conditional inclusion directive, implementations MUST NOT report any
errors for unsupported functionality within the excluded portion.

### 3.4. Basic structure

Lines that begin with a `@` character are called "directives".
They are intended to be processed immediately (i.e., during the initial read-in of the file), and they perform several
utility functions, such as defining additional symbols or setting the default base for numeric constants.
Each directive has a name, which MUST immediately follow the `@` character (without any whitespace); unrecognized
directive names are a syntax error.
All directive names are valid identifiers (as specified in [section 4.3][section4.3]).

Lines that begin with multiple `@` characters (at least two) are reserved for private use, for emulators that need to
define their own extensions to the format.
These lines should always be conditionally included in debugfiles in order to avoid errors.
Implementations MUST NOT issue syntax errors for these lines if they are excluded by a conditional inclusion
directive, regardless of content.
The semantics of private use lines are implementation-defined.

Any other lines are called "actions".
An action line specifies an action that the emulator must take when a certain event occurs and some conditions are
satisfied.
Actions are usually specified entirely in one line, but they may be broken down into several: if an action line ends
in a colon or semicolon, it continues on the next line.
If an action line is continued into the next line, the next line MUST be an action line as well; if it isn't (i.e., if
the next line is a directive or a private use line, or if the continued line is the last non-empty, non-comment line
of the file), a syntax error occurs.

The very first line of the file MUST be [a `@debugfile` directive][directive-debugfile]; this serves both as a
versioning system for the format and as a way of telling debugfiles apart.
If this directive is missing, a syntax error occurs.

### 3.5. Conditional inclusion

Some directives are used to control conditional inclusion.
These directives specify a condition; the following lines MUST be ignored if the condition is not met.
All conditions are evaluated on initial file read-in.
Conditions remain active until a new conditional inclusion directive appears or until the end of the file is reached.

Since there is no "end of condition" directive, conditional inclusion directives cannot be nested.
This is intentional.
Conditional inclusion directives themselves are never conditional.

Emulators must define a (hopefully unique) name and version for themselves.
Conditional inclusion directives can refer to these values in order to tailor debugfiles to specific emulators.
Emulator names and versions MUST be between 1 and 50 characters long, and they MUST contain only ASCII letters,
digits, and the characters `#$%&*+-.?@_`; the name MUST begin with a letter and the version MUST begin with a digit.
Emulators MUST also be able to compare their own version numbers in order to determine which one is greater.
Subject to these restrictions, emulator names and versions are arbitrary and chosen by their authors; emulator names
SHOULD be chosen making a reasonable effort to keep them unique.

All implementations MUST fully support all conditional inclusion directives, as these are the only mechanism that can
be used to prevent the use of unsupported features in [partially-compliant emulators][section9.1].

## 4. Directives

Directives perform various utility functions within a debugfile.
They are processed immediately during initial file read-in, and they can affect how the remainder of the file is
processed.
Directives are meant to have a support role within this specification; they are described before actions only because
some directives can affect the actions that come after them.

All valid directives are listed in this section, indicating their syntax and semantics.
Any directives not listed here (other than private-use directives) MUST be considered syntax errors.

### 4.1. Identification

This directive identifies the file as a debugfile.

#### `@debugfile`

Syntax: `@debugfile <version>`

This directive identifies the file as a debugfile, and indicates which version of the specification is being followed.
Valid version numbers are sequences of one to three numbers separated by period characters (`.`); leading zeros are
forbidden, and trailing zero components can be appended or removed, provided that the overall number of components
does not exceed three (e.g., `4.2.0` and `4.2` represent the same version number).
The current version number for this specification is given at the top of the document, with all trailing zero
components removed.

This directive MUST appear as the first line of the file.
If it appears more than once in the file, all instances of it MUST indicate compatible version numbers.
Implementations MUST issue errors and MUST NOT process the file any further if this directive is not present, if they
don't support the specification version the directive indicates, or if the directive appears multiple times in the
file indicating incompatible version numbers.
Two version numbers are considered compatible if they are equal (after removing all trailing zero components), or if
their first component is equal and non-zero.

If a file included via [the `@include` directive][directive-include] contains one or more `@debugfile` directives,
those directives MUST indicate version numbers compatible with the parent file.
(Included and transcluded files are the main reason why multiple `@debugfile` directives are allowed in the first
place.)
Including files that target incompatible versions of the specification is an error.

If multiple `@debugfile` directives, whether in the same physical file or through included files, specify different
(but compatible) version numbers of the specification, implementations MUST process the entire debugfile as the most
recent version specified across all those directives.
(Amongst two different version numbers, the most recent one is the one with the higher value in the first component
that differs.)
However, if the first `@debugfile` directive does not specify that most recent version, implementations MAY instead
process the file using the declared version until the point where a more recent version is declared, at which point
they are REQUIRED to use a version no earlier than the one declared.

### 4.2. Conditional inclusion

These directives allow [conditional inclusion][section3.5] of parts of the file.
They remain in effect until the end of the file or until a new conditional inclusion directive is found; conditional
inclusion directives are never conditional themselves, and thus nesting them is explicitly not possible.

As long as a conditional inclusion directive is in effect, implementations MUST ignore any lines (other than other
conditional inclusion directives and, at their choice, [`@debugfile`][directive-debugfile] directives) if the
condition given by the directive is false.

These directives MUST be supported by all implementations, since they are the only mechanism available to exclude
features not supported by some particular implementation.

If a file is [included][directive-include] inside a conditionally-included block, the entire file is either included
or ignored (although conditional inclusion directives inside the included file MUST be processed as normal); likewise,
if an included file creates conditionally-included blocks, those conditions end at the end of the included file.
In other words, conditional inclusion directives in a parent file and the files it includes MUST NOT interact in any
way.

#### `@always`

Syntax: `@always`

This is a conditional inclusion directive whose condition is always true.
In other words, this directive cancels all previous conditions for conditional inclusion, indicating that the
following lines (up to the next conditional inclusion directive or the end of the file) MUST be processed.

#### `@if`

Syntax: `@if <expression>`

This is a conditional inclusion directive that evaluates an expression.
The expression MUST be a [constant expression][expressions-constant].

This directive's condition is true if the expression's value is not zero.
The expression MUST be evaluated according to the current base and signedness in force, as modified by any prior
[`@radix`][directive-radix] and [`@signedness`][directive-signedness] directives, if any.

#### `@ifdef`

Syntax: `@ifdef <identifier>`

This is a conditional inclusion directive that is true if the identifier in question is defined.
The identifier may be a symbol or a variable name; variables MUST be prefixed with `@` for the purposes of this
directive.

Symbols are considered defined regardless of whether they come from external sources (such as implicit symfiles) or
from the debugfile itself (such as [`@sym`][directive-sym] and [`@symfile`][directive-symfile] directives).
Likewise, [variables][expressions-variables] are considered defined regardless of whether they are defined by the
emulator or via a [`@var`][directive-var] directive.

If a symbol or variable is defined in the debugfile itself, it MUST NOT be considered defined until the point of its
definition.

#### `@ifemu`

Syntax: `@ifemu <emulator spec> [, <emulator spec> [, <emulator spec> ...]]`

This is a conditional inclusion directive that is true if the emulator matches the given emulator specification.
Several specifications can be given, separated by commas; if that is the case, the condition is true if the emulator
matches any of the given specifications.
Emulator specifications are matched against the name and version of the emulator; valid values for names and versions
are detailed in [section 3.5][section3.5].

An emulator specification can have any of the following formats:

* Just the emulator name.
  The specification matches if the specified name is equal to the emulator's name.
  There are no wildcards or regular expressions: the name must match exactly.
  Name matching is case-insensitive.
* Emulator name and version, separated by one or more spaces; for example, `fooemu 3.8`.
  The specification matches if the specified name is equal to the emulator's name and the specified version is
  considered equal to the emulator's version.
  (This is shorthand for the equality comparison given below: `fooemu 3.8` is equivalent to `fooemu = 3.8`.)
* Emulator name, comparison operator and version; for example, `fooemu > 3.6`.
  There MUST be at least one space between the name and the operator; there MAY be spaces between the operator and the
  version.
  Valid operators are `<` (earlier than), `>` (later than), `=` or `==` (equal), `<>` or `!=` (not equal), `>=` (later
  or equal) and `<=` (earlier or equal).
  The specification matches if the specified name is equal to the emulator's name and the comparison between the
  specified version and the emulator's version, as given by the comparison operator, is true.
* Multiple comparisons; for example, `fooemu > 3.5 < 3.8 <> 3.7.2`.
  There MUST be at least one space before every operator; there MAY be spaces after the operators.
  The specification matches if the specified name is equal to the emulator's name and all the version comparisons are
  simultaneously true.

Since versions are only checked when the names match, version comparisons can be handled by each emulator in any way
they deem suitable.
Each emulator MUST be able to determine which one of two valid version numbers is later than the other, in order to be
able to resolve the comparison; if the specified version is not a valid version number (i.e., it doesn't match the
formatting rules for that emulator), the comparison MUST be considered to fail.
In particular, invalid version numbers MUST NOT raise any errors.

If version numbers contain alphabetical characters, version comparisons SHOULD be handled in a case-insensitive manner
(for example, by case-folding the version number string).

Note that the characters `<`, `>`, `!` and `=` are not allowed in either emulator names or version numbers.
Additionally, version numbers are required to begin with a digit.
Therefore, it is always possible to split the version number from the comparison operator in an unambiguous way.

If more than one emulator specification is given, there MAY be spaces around the commas used to separate them.
These spaces MUST be ignored when processing the emulator specifications.

#### `@ifnotdef`

Syntax: `@ifnotdef <identifier>`

This is the negated form of the [`@ifdef`][directive-ifdef] directive.
The condition is true if the corresponding [`@ifdef`][directive-ifdef] directive's condition would be false.

#### `@ifnotemu`

Syntax: `@ifnotemu <emulator spec> [, <emulator spec> [, <emulator spec> ...]]`

This is the negated form of the [`@ifemu`][directive-ifemu] directive.
The condition is true if the emulator matches none of the given emulator specifications.

All considerations given for the [`@ifemu`][directive-ifemu] directive are valid for `@ifnotemu`.

#### `@else`

Syntax: `@else [<condition>]`

This is a conditional inclusion directive whose condition is only evaluated if previous directives' conditions were
false.

The `@else` directive MUST NOT be the first conditional inclusion directive in the file.
If any prior conditional inclusion directive up to (and including) the most recent non-`@else` conditional inclusion
directive in the file was considered true, this directive's condition is considered automatically false.
Otherwise, its condition is evaluated.
(Note: this is equivalent to the handling of `else` and `else if` statements in most programming languages.)

The condition of an `@else` directive can be any of the other conditional inclusion directives (_without_ the leading
`@`, since the `@` merely marks the line as a directive), following the corresponding directive's syntax and
semantics.
(For example, `@else ifemu foobar > 3.14` is a valid directive.)
If the condition is omitted, it defaults to [`always`][directive-always].

### 4.3. Declarations

These directives declare elements so that further directives and action lines can use them.
Declarations MUST appear before the first use of the declared elements.

Every declared element has an identifier.
Identifiers MUST be purely ASCII strings, and they MUST only contain letters, numbers, and the following characters:
`$#.@_`.
Additionally, their first character MUST be a letter or an underscore.
Identifiers are case-sensitive; each declared identifier MUST be unique.

#### `@sym`

Syntax: `@sym <name> <address>`

This directive declares a symbol; any further reference to the declared symbol MUST use the address declared by this
directive.
Additionally, if the emulator keeps a list of symbols for the loaded program (e.g., loaded from a symfile), it SHOULD
add the declared symbol to that internal list.

If the declared symbol collides with a symbol loaded from an implicit source such as a symfile, the address given in
this directive takes precedence.
However, the symbol MUST NOT collide with any other symbol declared in the debugfile.
Implementations MAY show a warning on initial read-in of the debugfile if a declared symbol collides with a symbol
loaded from an implicit source.

The address can be any constant [address expression][expressions-address]; this expression will use the current base
and signedness in force (as declared by the [`@radix`][directive-radix] and [`@signedness`][directive-signedness]
directives, if any).
If this address expression is banked, so will be the declared symbol; otherwise, the symbol will be unbanked.

#### `@alias`

Syntax: `@alias <name> <reference>`

This directive declares a symbol alias, that is, a symbol that refers to another symbol.
This may be used to be able to name symbols with invalid characters in them, or simply to give a shorter or better
name to an existing symbol.
The newly-declared symbol becomes equivalent to the aliased symbol for all intents and purposes.

If the declared alias collides with a symbol loaded from an implicit source such as a symfile, the alias declared by
this directive takes precedence.
However, the alias MUST NOT collide with any other symbol declared in the debugfile.
Implementations MAY show a warning on initial read-in of the debugfile if a declared alias collides with a symbol
loaded from an implicit source.

The reference (i.e., the aliased symbol) MUST be given as a quoted string; no escape sequences are allowed within it.
The referenced symbol MUST exist; if it is declared in the debugfile itself, the alias MUST come later in the
debugfile than the symbol.
If the aliased symbol is declared externally (e.g., in a symfile), it MUST NOT be overridden by any declarations in
the debugfile.

#### `@var`

Syntax: `@var <name> <value>`

This directive declares a variable to be used in expressions.
The name MUST begin with an underscore and it MUST NOT collide with any other existing variable; variable names that
don't begin with an underscore are reserved for the implementation (including the required variables listed in the
[corresponding section][expressions-variables]).

The value is the initial value of the variable, and it can be any [constant expression][expressions-constant]; that
expression will use the current base and signedness in force, as given by the [`@radix`][directive-radix] and
[`@signedness`][directive-signedness] directives.

User-defined variables MUST be reinitialized to their initial value whenever the emulator is reset (for example, via
the use of [the `reset` command][command-reset], through a menu entry or a hotkey, etc.).

#### `@str`

Syntax: `@str <name> <value>`

This directive declares a named string that can be referenced by commands that display messages to the user and by
other named strings.
Strings live in their own namespace: the given name MUST NOT collide with any other string declared in the debugfile,
but it MAY be equal to some symbol or variable.

The value MUST be a constant string, between quotes.
The format for the value is identical to the format used in commands that take a string as an argument; refer to
[section 7][section7] for further details.

### 4.4. Action groups

These directives allow grouping actions into groups, in order to be able to execute actions upon those named groups
(such as disabling all actions within a group).
Actions that come before [a `@group` directive][directive-group] are not considered to belong to any group.

#### `@group`

Syntax: `@group <name> [<display name>]`

This directive declares that the actions that follow it belong to the specified group.
If there was already a `@group` directive in effect, this directive replaces it; it is not necessary to explicitly
close groups with [`@endgroup`][directive-endgroup] before opening a new group.

The name of the group is an identifier that must follow the same rules given in [section 4.3][section4.3].
If the same group name is given multiple times in a debugfile, all of the actions covered by the corresponding
`@group` directives are considered to belong to the same group; in other words, reusing a group name appends actions
to that group.

The display name is an OPTIONAL constant quoted string; no escape characters are allowed within it.
If this string is given, and the emulator's interface allows grouping debugging actions, it SHOULD use the given
display name as the name for the group.

If multiple `@group` directives refer to the same group name, and at least two of them specify a display name for the
group, they MUST all specify the same display name.
If different display names are specified for the same group, the emulator MAY, at its discretion, raise an error or
use any of the display names given.

#### `@endgroup`

Syntax: `@endgroup`

This directive closes a [`@group`][directive-group] directive; actions that follow will not be considered to belong to
any group.

### 4.5. Included files

These directives allow a debugfile to refer to other files.
The specified file paths may be relative or absolute.
Implementations MAY impose restrictions on allowable file paths for security reasons.

File paths MUST be given as quoted strings; no escape sequences are allowed within them.

#### `@include`

Syntax: `@include <filepath>`

This directive includes a separate debugfile as part of the one where it appears.
Circular inclusions (that is, chains of inclusions that result in a file directly or indirectly including itself, also
known as recursive inclusions) are not allowed; it is RECOMMENDED that implementations check for this.

#### `@symfile`

Syntax: `@symfile <filepath>`

This directive includes a symfile into the debugging information for the loaded program.
If the included symfile has already been automatically loaded by the implementation, it MAY reload its symbols.

A symfile is a text file where each nonempty line contains either a comment or a symbol declaration.
Comments begin with zero or more whitespace characters followed by a semicolon; they are ignored.
Symbol declarations contain an address and a symbol name separated by one or more whitespace characters; the address
is always hexadecimal, and it can be a banked address (given as a `<bank>:<address>` pair) or an unbanked address
(given as a single hexadecimal value).

Implementations MAY automatically load symfiles even in the absence of any `@symfile` directives; this is part of the
normal behavior of debuggers.

### 4.6. Configuration

These directives allow configuring some internal parameters of the debugfile, such as the default base in force for
numeric constants.
They take effect immediately for the directives and actions that follow them; they may be given several times to
change the configuration at different points in the file.

[Included files][directive-include] MUST have a configuration separate from their parent files; in other words, they
neither inherit nor modify the configuration from the file that includes them.

The defaults for each configuration option are stated in the description of the directives themselves.

#### `@radix`

Syntax: `@radix 2|10|16`

Default: 10

This directive specifies the default base in force for numeric constants.
Numeric constants that don't contain a prefix character indicating their base will use this value as their base.
Valid values are 2, 10 and 16.

#### `@signedness`

Syntax: `@signedness signed|unsigned`

Default: `unsigned`

This directive specifies whether arithmetic expressions in actions will use signed or unsigned arithmetic; this
affects divisions and right shifts.

### 4.7. Miscellaneous

These directives don't fit in any of the above categories.

#### `@warning`

Syntax: `@warning <message>`

This directive contains warning text for the user; this is intended to be used within a conditionally-included section
to warn the user about missing features or actions.
The emulator SHOULD display the warning message to the user during initial read-in of the file, and it SHOULD allow
the user to abort the load of the debugfile.

The message MUST be a quoted string; no escape sequences are allowed within it.

#### `@error`

Syntax: `@error <message>`

This directive is equivalent to the `@warning` directive, but it generates a hard error.
If this directive is encountered and not excluded by a conditional inclusion directive, the error text MUST be shown
to the user and the debugfile MUST NOT be loaded.

## 5. Actions

Actions are a series of responses that the user requests the emulator to automatically take when a certain condition
is met, such as stopping emulation when a particular memory address is written to.
Actions are the main focus of this specification; the goal of a debugfile is to define these actions for a particular
program.

### 5.1. Basic action structure

An action is comprised of two components: a _condition_ and one or more _commands_.
The condition indicates when the action will execute, and the commands indicate what the action will do when it
executes.

The condition is bound to one or more addresses, and it triggers when the addresses in question are read, executed or
written to.
The least restrictive condition is one that is true for all reads, writes and executions; such a condition would
trigger at least once per instruction executed.

Commands are the actual steps that the emulator must take when the condition becomes true.
Commands are simple debugging steps, such as stopping execution and breaking into the debugger, or printing a debug
message to some debug console.
An action can have any number of commands, which MUST be executed in sequence.

It is possible for many actions' conditions to be true simultaneously.
If this is the case, the emulator MUST queue all of them for execution in any order it wishes.
The commands for each action MUST be executed in sequence, and they MUST NOT be interleaved with other actions'
commands.

An action can be enabled or disabled.
Actions are enabled by default, unless their flags set them as disabled.
A disabled action behaves as if its condition was always false, i.e., it never triggers.

Emulators MAY offer the ability to edit some or all of the properties of an action.
For instance, they may allow enabling or disabling actions, as well as deleting them.
An edited action SHOULD behave as if it had been entered into the debugfile in whatever form it results from the
edits; in other words, such edits SHOULD take precedence over the originally-entered data in the debugfile.

### 5.2. Syntax overview

The overall syntax of an action line is as follows:

```
<condition>: <command> [; <command> [; <command> ...]]
```

There MAY be zero or more whitespace characters at either side of the colon and semicolons; such whitespace characters
are meaningless and ignored.
Also, an action MAY be broken down into more than one line right after the colon and/or any of the semicolons shown
above, as explained in [section 3.4][section3.4]; action lines MUST NOT be broken down anywhere else.
The trailing colon or semicolon at the end of a line indicates that the action has been broken down into several lines
and that the next line continues the same action.

If an action is broken down into several lines, these lines MUST be consecutive; in other words, after an action line
ending with a colon or semicolon, the action MUST be continued on the immediately following line.
(Note that blank lines and comment lines are considered stripped by the normalization process described in
[section 3.2][section3.2], and thus they are not considered for this rule; if the second line is separated from the
first by blank and/or comment lines only, it is still considered to immediately follow the first line.)
An action line ending with a colon or semicolon MUST NOT be the last line of the debugfile and it MUST NOT be followed
by a directive or a private use line.

Semicolons are used as command delimiters, not command terminators.
In particular, this means that the last command of an action MUST NOT be followed by a semicolon.
Additionally, all actions MUST have at least one command.
There is no explicit upper bound on the number of commands per action, but implementations MAY impose a reasonable
limit.
Such a limit SHOULD NOT interfere with intended uses of debugfiles.

### 5.3. Expressions

Many components within an action use expressions to specify values.
Expressions have similar syntax regardless of where they appear, and thus they are described here.

Expressions use a single data type: 32-bit integers.
All values in an expression are 32-bit integers, and operators manipulate those values.
Expressions can contain numeric constants, symbols, variables, memory accesses and operators.
All operators that result in a signed value use two's complement arithmetic.

Expressions may be signed or unsigned; this is determined by the action's flags.
This only affects some operators and variables; the exact cases where the signedness makes a difference are indicated
below.
For example, division is signed or unsigned depending on the signedness of the expression where it appears.

#### Numeric constants

Numeric constants are sequences of digits, optionally prefixed by a base indicator character, that evaluate to a
constant non-negative numerical value.
The value of a numeric constant MUST fit in 32 bits.

Numeric constants can be prefixed with the characters `%`, `#` or `$` to indicate that they are respectively in base
2, 10 or 16; if no prefix is present, the constant is in the default base in force, as given by
[the `@radix` directive][directive-radix].
For instance, the constants `%10100`, `#20` and `$14` all specify the same value, and so does the constant `20` if the
default base in force is 10.
The hexadecimal digits `A` through `F` can be written in uppercase or lowercase.

Numeric constants that are not prefixed by a base indicator character MUST begin with a digit.
This is only relevant if the default base in force is 16; in that case, a numeric constant MUST NOT begin with the
digits `A` through `F`.
(This can be avoided by adding an additional `0` in front, or by explicitly prefixing the constant with a `$`
character: for example, the value `FF` can be written as `0FF` or `$FF`.)

Numeric constants MUST NOT use characters that are not valid digits for the chosen base, either implicitly or via a
prefix.
A numeric constant with invalid digits is a syntax error.

If a signed expression contains a numeric constant that evaluates to a value that has its upper bit set (i.e., a value
that is greater than or equal to `$80000000`), that constant MUST be converted to a negative value via two's
complement arithmetic.

#### Symbols

Symbols are constants, specified in a symfile or declared explicitly in a [`@sym`][directive-sym] or
[`@alias`][directive-alias] directive, that evaluate to an address.
The address in question may be banked or unbanked; a banked address contains an explicit bank number in addition to
the address portion.
Unless otherwise stated in a specific context, bank numbers are generally ignored; banked symbols evaluate to their
address portions when used in an expression.

Symbols always evaluate to an unsigned value, i.e., they are zero-extended into 32 bits.

A symbol's name can collide with a variable's name.
If this is the case, using its name in an expression MUST evaluate to the symbol, not the variable.
The use of a name in an expression that is neither a valid symbol name nor a valid variable name MUST be considered an
error.

Only symbols with valid names (according to the rules stated in [section 4.3][section4.3]) may be used in expressions.
For symbols that don't comply with these rules, an alias needs to be declared (using the [`@alias`][directive-alias]
directive) in order to be able to refer to them.

If a symbol's address portion falls within an unbanked memory area and its bank number is zero, it SHOULD be treated
as an unbanked symbol in order to facilitate its usage.
(Banked memory areas are defined in the subsection regarding [memory accesses][expressions-memory].)

A banked symbol whose address falls at the beginning of a memory region may have a bank number that would be valid for
the region immediately preceding it.
For example, a symbol might have an address of `02:C000` (assuming that 2 is a valid SRAM bank for that program).
This is common in past-the-end symbols, where a symbol points to the end of some memory block.
Implementations MUST NOT consider this an error.

#### Variables

Variables are values that represent some part of the program's state, such as a register.
They come from two sources: the emulator provides a fixed set of variables that the user can use, and the user can
define additional variables through [the `@var` directive][directive-var].
User-defined variables only change when the user explicitly changes them (for instance, via a command); emulator
variables can also change to reflect changes in the program state.
All user-defined variables MUST begin with an underscore; emulator variables MUST NOT begin with one.

A variable's name can collide with a symbol's name; in this case, using its name in an expression MUST evaluate to the
symbol, not the variable.
In order to allow using the variable in this case, the name can be prefixed with a `@` character to indicate that it
SHALL evaluate to the variable.
Variable names MAY be prefixed with `@` even if there is no collision — for example, the expression `@bc` MUST
evaluate to the `bc` variable, regardless of whether a `bc` symbol exists.
The variable `@` is an exception, and it MUST NOT be prefixed with an additional `@`.

The use of a name in an expression that is neither a valid variable name nor a valid symbol name, as well as the use
of a `@`-prefixed name that is not a valid variable name, MUST be considered an error.

Emulators MUST provide at least the variables indicated below.
Emulators MAY provide any additional variables they wish, including alternate casings for the following variables,
provided that none of them begin with an underscore.

|Variables                        |Bits|Signedness|Description                                                       |
|:--------------------------------|---:|:--------:|:-----------------------------------------------------------------|
|`a`, `b`, `c`, `d`, `e`, `h`, `l`|   8|Context   |CPU single-byte registers.                                        |
|`f`                              |   8|Unsigned  |CPU flags register; it is always unsigned.                        |
|`af`, `bc`, `de`, `hl`           |  16|Context   |CPU register pairs.                                               |
|`sp`, `pc`                       |  16|Unsigned  |CPU two-byte registers; they are always unsigned.                 |
|`z`, `cy`                        |   1|Unsigned  |Zero and carry flags. Always 0 or 1.                              |
|`ime`                            |   1|Unsigned  |Interrupt master enable flag. Always 0 or 1.                      |
|`@`                              |  16|Unsigned  |Address of current instruction. MUST NOT be prefixed with `@`.    |
|`bank`                           | Any|Unsigned  |Current bank (at the location of `@`); 0 if that area is unbanked.|
|`rombank`                        | Any|Unsigned  |Current ROM bank. MUST be 0 for ROMs without banking.             |
|`srambank`                       | Any|Unsigned  |Current SRAM bank. MUST be `$FFFFFFFF` for programs without SRAM. |
|`sramenable`                     |   2|Signed    |1 if SRAM is enabled, 0 if disabled, -1 if no SRAM is present.    |
|`target`                         |  16|Unsigned  |Address that caused the action to fire.                           |
|`op`                             |   2|Unsigned  |Current operation (0: read, 1: write, 2: execute, 3: read+write). |
|`value`                          |   8|Context   |Value being read or written, or opcode byte for execution actions.|

The "signedness" column in the table above indicates how the variables' values are extended into 32 bits: they are
zero-extended if unsigned, or sign-extended if signed.
If the signedness is "context", it means that they take the signedness of the expression they appear in.

The "bits" column of the table above indicates the natural width of the value, i.e., the number of bits the value
actually has.
The value will be extended to 32 bits when used in an expression, since all values are 32 bits wide in expressions.
The value "any" in that column indicates that the variable can have any natural width — the emulator must
select a reasonable width for the variable.
Emulators MAY select different widths for different debugged programs.
For example, the `rombank` variable will typically be as wide as the actual ROM bank selection registers in the
banking hardware for the particular program being debugged.

The `pc` variable contains the value of the `pc` register, which points to the _next_ instruction (i.e., to the end of
the current instruction); on the other hand, the `@` variable points to the instruction currently being executed.
Therefore, they will always differ by one, two or three.
For example, if the currently executing instruction is a `swap a` instruction (which is two bytes long) at address
`$1234`, then `@` will be equal to `$1234` and `pc` will be equal to `$1236`.

#### Memory accesses

A memory access indicates that a value MUST be retrieved from memory in order to evaluate the expression.
Memory accesses can retrieve values that are 8, 16 or 32 bits in width; if the value is not 32 bits wide, it MUST be
extended into 32 bits according to the signedness of the expression (zero-extended if unsigned, sign-extended if
signed).
The retrieval of a value from memory MUST NOT be considered a memory read; in particular, it MUST NOT trigger any
actions that watch the address in question.

Memory accesses that are wider than 8 bits will read several bytes of memory.
These bytes can be collected into a single value using either little or big endianness; this is indicated for each
access.

The syntax for a memory address is any of the following: `[<address>]`, `[<address>!]`, `[<address>!!]`,
`[<address>?]` or `[<address>??]`.
These forms respectively indicate 8-bit, 16-bit little-endian, 32-bit little-endian, 16-bit big-endian and 32-bit
big-endian access.
For example, `[$4000]` is an 8-bit access, `[$5000?]` is a 16-bit big-endian access, and `[@bc!!]` is a 32-bit
little-endian access.

Memory accesses may be banked or unbanked.
For banked memory areas, banked accesses use a specific bank, while unbanked accesses use whichever bank is currently
loaded.
Accesses to unbanked memory areas or accesses that cross memory areas (such as a 16-bit access at address `$9FFF`)
MUST be unbanked.
A memory area is considered banked if bank switching is possible in that area, and unbanked otherwise; emulators MAY
consider all of ROM, VRAM, SRAM and/or WRAM to be banked even when no bank switching is possible in some or all of
those areas.

The `<address>` part of the memory access may be any [address expression][expressions-address].
There MAY be arbitrary whitespace after the `[` character or before the `]` character; there MAY also be arbitrary
whitespace before the `!`, `!!`, `?` or `??` signs at the end of a multi-byte access.
This whitespace is meaningless and ignored.

#### Operators

Operators indicate a calculation that is carried out on their operands in order to produce a new result.
All operators take 32-bit values as their operands and generate a 32-bit result; if their result overflows 32 bits, it
MUST be truncated to 32 bits.
Some operators behave differently depending on whether they are signed or unsigned; operators always take the
signedness of the expression they appear in.
Signed calculations use two's complement arithmetic.

Operators can be unary or binary.
Binary operators go between their two operands, while unary operators go before their single operand.
Operators MAY be surrounded by arbitrary amounts of whitespace; this whitespace is meaningless.
Unary operators MUST appear at the beginning of an expression or parenthesized subexpression; emulators MAY relax this
rule, but they are also allowed to issue an error instead if the rule is broken.

Parentheses can be used to group all or part of an expression into a subexpression; such a subexpression MUST be
evaluated into a single value before being used as an operand to some other operator.
In other words, parentheses indicate that the subexpression takes precedence above the regular precedence rules.
Parentheses MAY be surrounded by arbitrary amounts of whitespace; this whitespace is meaningless.

Unary operators MUST be evaluated before binary operators.
If an implementation allows many of them to appear together (violating the rule about unary operators appearing at the
beginning of an expression or subexpression, since only the first of such operators can be at the beginning), they
MUST be evaluated right to left.
Valid unary operators are the following ones, all having the same precedence:

|Operator|Description                                                                                                |
|:------:|:----------------------------------------------------------------------------------------------------------|
|`-`     |Two's complement of its operand.                                                                           |
|`+`     |Results in its operand unmodified. Added for completeness.                                                 |
|`~`     |One's complement of its operand, i.e., unary NOT.                                                          |
|`&`     |MUST be applied to a symbol; results in that symbol's bank. If the symbol is unbanked, the result is 0.    |
|`!`     |1 if the operand is zero, or 0 otherwise.                                                                  |
|`!!`    |0 if the operand is zero, or 1 otherwise.                                                                  |

If an implementation allows multiple unary operators to appear together, the `!!` operator MAY be implemented as two
consecutive instances of the `!` operator instead of being a valid operator by itself.

Binary operators MUST be evaluated in descending order of precedence; ties are broken in left-to-right order.
Valid binary operators are:

|Operator  |Precedence|Description                                                                                   |
|:--------:|---------:|:---------------------------------------------------------------------------------------------|
|`<<`      |         9|Left shift. If the right operand is not between 0 and 31, the result is zero.                 |
|`>>`      |         9|Right shift, affected by signedness. Shift counts not between 0 and 32 are taken to be 32.    |
|`*`       |         8|32-bit multiplication; the result is unaffected by signedness.                                |
|`/`       |         8|32-bit division, always rounding towards zero. If the divisor is zero, the result is zero.    |
|`%`       |         8|32-bit remainder, affected by signedness. Defined as `x % y` = `x - x / y * y`.               |
|`**`      |         8|Upper 32 bits of the 64-bit full product of its operands. Affected by signedness.             |
|`+`       |         7|32-bit addition.                                                                              |
|`-`       |         7|32-bit subtraction.                                                                           |
|`&`       |         6|Bitwise AND.                                                                                  |
|`\|`      |         5|Bitwise OR.                                                                                   |
|`^`       |         5|Bitwise XOR.                                                                                  |
|`=`, `==` |         4|1 if both operands are equal, or 0 otherwise.                                                 |
|`!=`, `<>`|         4|0 if both operands are equal, or 1 otherwise.                                                 |
|`<`       |         3|Less than comparison; affected by signedness. 1 if true, 0 if false.                          |
|`>`       |         3|Greater than comparison; affected by signedness. 1 if true, 0 if false.                       |
|`<=`      |         3|Less than or equals comparison; affected by signedness. 1 if true, 0 if false.                |
|`>=`      |         3|Greater than or equals comparison; affected by signedness. 1 if true, 0 if false.             |
|`&&`      |         2|Logical AND. 1 if both operands are non-zero, or 0 otherwise.                                 |
|`\|\|`    |         1|Logical OR. 1 if either operand is non-zero, or 0 otherwise.                                  |
|`^^`      |         1|Logical XOR. 1 if exactly one operand is non-zero, or 0 otherwise.                            |

<!-- A note for readers of the raw file: GitHub-flavored Markdown requires the pipe character (`|`) to be escaped
     within tables, even if it appears between backticks.
     The actual operators for bitwise and logical OR are `|` and `||`, without any backslashes.                   -->

If there is any parsing ambiguity between two unary or two binary operators, it MUST be resolved in favour of the
longest one of the two.
If there is any parsing ambiguity between a unary and a binary operator, it MUST be resolved in favour of the binary
operator.

#### Constant expressions

An expression is said to be constant if it doesn't involve any variables or memory accesses; in other words, a
constant expression may only contain numeric constants and symbols (and operators and parentheses combining them).
Constant expressions are a subset of all expressions; while they can be used wherever an expression is called for,
some parts of this specification will explicitly require constant expressions.

The value of a constant expression MAY be precalculated when the debugfile is initially read in, since it depends on
values that cannot change at runtime.
If an emulator precalculates constant expressions this way and it also offers the ability to edit symbol addresses,
constant expressions MUST be reevaluated when the address of a symbol they refer to changes.

#### Address expressions

Address expressions are a specific kind of expression that is used in some contexts; they evaluate to a memory address
instead of a value.
An address expression may be banked or unbanked: a banked address expression refers to a specific bank/address pair,
while an unbanked address expression refers to a specific address in any bank.

Address expressions can take one of three forms: `<expression>`, `:<expression>` and `<expression>:<expression>`.
The last two forms are explicitly unbanked and banked, respectively; in the last form, the first expression is the
bank and the second one is the address.
The colon in those forms MAY be surrounded by arbitrary amounts of whitespace; this whitespace is meaningless.

The first form is just an ordinary expression.
In that case, if the first token of the expression (not including any parentheses) is a banked symbol, the address
expression is banked and takes the bank of that symbol; otherwise, the address expression is unbanked.

The address part of the expression MUST be truncated to 16 bits, ignoring the upper half of the value; overflows MUST
NOT cause an error.
Likewise, if the address expression is banked, the bank part of the expression MUST be truncated to the correct number
of bits for that address, ignoring any overflowing bits.
Any truncations MUST be carried out only after the value of the expression has been determined; in other words, the
expressions contained in the address expression MUST be evaluated to a single value using the full 32-bit precision
before being truncated.

If the address part of the expression falls within an unbanked memory area, the address expression MUST be unbanked;
however, implementations MAY accept banked address expressions with a bank number of zero for those areas.

If an address expression meets the conditions set for [constant expressions][expressions-constant], it is a constant
address expression.

### 5.4. The condition field

The condition field of an action indicates when the action will fire.
This field is comprised of three subfields: the address field, the flags field and the condition expression field.
The first two are mandatory, but the condition expression field MAY be missing; if it is, it is assumed to be the
constant expression `1`.
These subfields are separated by arbitrary amounts of whitespace.

The address and flags subfields MUST NOT contain any inner whitespace, since whitespace is used to delimit them.
The condition expression subfield MAY contain arbitrary inner whitespace, as it is the last subfield of the condition
field.
The condition field is terminated by a colon; while the address and condition expression subfields can contain
colons, the terminating colon is unambiguous because the flags subfield is mandatory and all colons within the
condition expression subfield will necessarily appear within [memory access expressions][expressions-memory] (and thus
inside brackets).

The address subfield indicates the addresses that can cause the action to fire; the flags subfield indicates which
operations over those addresses will trigger it, as well as some additional attributes for the action.
The condition expression subfield is an expression that must be non-zero for the action to fire; if the condition
expression for an action evaluates to zero whenever that action would fire, it does not.
For example, the following condition field:

```
$04:$58AB rx @bc > $1234
```

indicates that the action will fire if the byte at bank 4, address `$58AB` is read or executed, but only if register
`bc` is greater than `$1234`.

The condition field will be followed by a colon and the command list; the action line MAY be broken after the colon by
inserting a newline.
Following the example above, the full action line might look like this:

```
$04:$58AB rx @bc > $1234: break
```

The colon that terminates the condition field (the one after `$1234`) is unambiguous, because the colon in the address
subfield (`$04:$58AB`) comes before the flags subfield, and the flags subfield is mandatory.

Additional whitespace between the subfields MAY be inserted for readability, like so:

```
$04:$58AB       rx      @bc > $1234:            break
```

The condition expression subfield can be any arbitrary expression, as detailed in [section 5.3][section5.3].
The other two subfields will be explained in the following sections.

### 5.5. The address subfield

The address subfield of the condition field specifies the addresses that will be watched by the action, i.e., which
addresses can cause the action to fire.
This subfield contains one or more address specifications separated by commas; each specification may be a single
address or a range, and it may be banked or unbanked.
The address subfield MUST NOT contain any whitespace; while whitespace is normally allowed within expressions (and the
address subfield will usually contain one or more of them), whitespace is used to delimit the address and flags
subfields, and thus it cannot be included within those subfields.

A single address is given as a constant [address expression][expressions-address]; the address will be banked or
unbanked depending on whether the given expression is.

Since addresses in this subfield are constant expressions, numerical constants are handled in the manner explained in
[the corresponding part of the Expressions section][expressions-numbers].
In particular, this means that numerical constants don't default to hexadecimal (unless there is a
[`@radix 16`][directive-radix] directive in effect), and that they MUST begin with a digit or a base indicator
character.

If the address subfield specifies a range, the starting address MUST be lower than or equal to the ending address; no
wrap-around is assumed for this subfield.
Also, if the address is banked, the address or range of addresses MUST be in a banked area of memory and MUST NOT
cross memory areas.
Address ranges always include both endpoints; if a range is unbanked, but it covers a banked portion of memory, the
address or range refers to those addresses in all banks.

Ranges can be specified in one of two ways:

* `<address>--<expression>`: the start and end are given explicitly.
  The `--` delimiter represents a hyphen (indicating a range), not the subtraction operator; for instance, the range
  `$C000--$DFFF` covers all of WRAM.
* `<address>++<expression>`: the second expression indicates the length of the range.
  For instance, `$C000++$2000` covers all of WRAM as well.

In both cases, `<address>` is any constant [address expression][expressions-address], and `<expression>` is any
[constant expression][expressions-constant].
The result of `<expression>` MUST be truncated to 16 bits before computing the range; in particular, this means that
using a negative value of `<expression>` for the `++` form will not work as expected.
The `--` and `++` symbols are delimiters, not operators; there is no need to parenthesize the two expressions that
they delimit, and they don't have a precedence because they are not involved in any calculation.
For the `++` form, after truncating to 16 bits, the length expression MUST NOT evaluate to zero.

Ranges MUST NOT wrap around.
This means that, for the `--` form, the ending address MUST be greater than or equal to the starting address; for the
`++` form, after truncating both the address and the length to 16 bits, their sum MUST NOT exceed `$10000`.
(It is acceptable for the sum to be _exactly_ `$10000`, as this occurs for ranges that extend to the very end of the
addressing space, such as `$FF80++$80`.)

The `,`, `++` and `--` delimiters cannot appear naturally in expressions, and thus they will unambiguously delimit the
expressions involved in the address specifications.
For implementations that don't require unary operators to appear at the beginning of an expression or subexpression
(which [the corresponding section][expressions-operators] of this specification allows as an option), any `++` or `--`
sequences in address specifications MUST be parsed as the corresponding delimiters and not as operator sequences, even
if this would cause parsing to fail.
(For example, the specification `20--30` MUST be parsed as a range and not as `20-(-30)`; also, the specification
`2*(20++30)` MUST be parsed as a range (which will cause parsing to fail), even though parsing it as `2*(20+(+30))`
would be successful.)

If the address subfield contains two or more address specifications, the action will watch the union of the addresses
specified by all such address specifications.
(For example, a specification of `$4000,$4010--$4013,$4020++5` will watch a total of 10 addresses.)

The address subfield can also be a single `*` character.
This is shorthand for the range `$0000--$FFFF`.
The `*` character MUST appear by itself; in other words, it MUST NOT be part of a list of two or more address
specifications.
(This is not to be confused with the `*` operator, which _can_ appear any number of times in the address subfield.)

### 5.6. Flags

An action's flags indicate which operations will trigger the action, as well as some attributes for it.
The address subfield is used to specify the addresses that trigger the action; the flags refine this by stating which
operations on those addresses will cause it to fire.
Actions MUST always fire _before_ the operation is executed.

Flags come in two kinds: _operation flags_ and _attribute flags_.
Operation flags are the flags that represent operations over the memory addresses that are being watched; every action
MUST have at least one operation flag.
Attribute flags indicate an action's attributes; actions may have any number of these, including none.

Flags consist of one or two characters; multiple flags can be given by concatenating them.
Flags MUST NOT be separated by whitespace.
Two-character flags always consist of identical characters, and they are modified versions of the corresponding
one-character versions; the corresponding one-character and two-character flags MUST NOT be used simultaneously in the
same action.
Flags MUST NOT be duplicated; any particular flag may only be given once per action.
Flags MAY be given in any order; the order is immaterial.

Valid flags are as follows:

|Flag  |Type     |Description                                                                                        |
|:----:|:-------:|:--------------------------------------------------------------------------------------------------|
|`r`   |Operation|Indicates that the action will fire when an address is read.                                       |
|`w`   |Operation|Indicates that the action will fire when an address is written to.                                 |
|`ww`  |Operation|Indicates that the action will fire when a new value is written to an address (modifying write).   |
|`x`   |Operation|Indicates that the action will fire when an address is executed.                                   |
|`xx`  |Operation|Indicates that the action will fire when an address is being jumped to.                            |
|`s`   |Attribute|Sets the signedness to signed for this action's expressions, overriding the default.               |
|`ss`  |Attribute|Sets the signedness to unsigned for this action's expressions, overriding the default.             |
|`d`   |Attribute|Disables the action. The action will be loaded in disabled state, instead of enabled.              |
|`m`   |Attribute|Indicates that the action can fire multiple times per instruction.                                 |
|`b`   |Attribute|Indicates that the action will only fire when the boot ROM is enabled.                             |
|`bb`  |Attribute|Indicates that the action will fire regardless of whether the boot ROM is enabled.                 |

For actions that fire on writes, if an expression contains a memory access that retrieves the value of the address
being written to, the value _before_ the write MUST be used for that expression.
The `ww` flag indicates that the action fires when the value being written to is different from the value already
present at that address; in other words, when the expression `[target] != value` evaluates to 1.

Executing an instruction MUST NOT be considered a memory read for the purpose of firing actions.
Actions that fire on reads are only triggered by explicit memory reads.

If an instruction triggers an action multiple times (for example, by writing two bytes at once, or by reading and
writing to the same address at once), the action MUST fire only once unless the `m` flag is given.
With the `m` flag, the action MUST fire once per operation. For example, given the following actions:

```
$C100--$C101 w: message "action 1"
$C100--$C101 wm: message "action 2"
```

the instruction `ld [$C100], sp` would cause the first action to fire once but the second action to fire twice (once
per byte written).
If an action with the flags `xm` was created for the address range containing that instruction, that action would fire
three times, since three bytes are being executed.

If a multi-byte instruction is executed, actions that fire on execution MUST fire if any part of the instruction lies
within the addresses watched by the action.
If the action does not include the `m` flag, it fires only once; the `target` variable MUST be set to point to the
first byte of the instruction that lies within the watched addresses.
On the other hand, actions that fire on jump targets (i.e., actions that contain the `xx` flag) MUST only fire if the
jump target matches exactly a watched address, regardless of the size of the instruction at the receiving end of the
jump.

Actions that contain the `xx` flag MUST fire before the jump is taken; in other words, the value of the `@` register
MUST point to the jump instruction (`jr`, `jp`, `call`, `ret` or `rst`) that is causing the jump.
These actions MUST NOT fire due to the execution of an interrupt handler (unless, of course, the handler is being
jumped to or called explicitly).
These actions also MUST NOT fire due to jumps that are not taken.
For these actions, the value of the `target` variable will be the target of the jump.

For the purpose of firing actions that trigger on instruction execution, undefined instructions MUST be cosidered to
be one byte long; any action firing on the execution of such an instruction MUST fire before the emulator handles the
error condition that results from attempting to execute it.
The `stop` and `halt` instructions MUST also be considered to be one byte long, regardless of whether they are
followed by a `nop` instruction or not; if this `nop` is present and it does get executed (as opposed to being skipped
by the CPU), it MUST be handled as a separate instruction for the purpose of firing actions that trigger on
instruction execution.

For a combined read/write operation (such as `inc [hl]`), if an action fires on both reads and writes and the `m` flag
is given, the read operation MUST fire before the write.
If the action does not contain the `m` flag, the combined operation fires the action once, with the `op` variable
taking the value 3 and the `value` variable taking the value of the written byte.

If an action fires on reads and modifying writes (i.e., it contains the `r` and `ww` flags), the above behavior only
applies for bytes that are modified.
Reads followed by non-modifying writes (e.g., the execution of a `set 0, [hl]` instruction when the value at `[hl]` is
already odd) MUST be treated as pure reads for the purpose of firing the action.

For an instruction that accesses multiple bytes at once, if it causes an action watching several of those addresses
that does not contain the `m` flag to fire, the `target` and `value` variables for it MUST be set for the highest
address in common between the action and the operation.
If the action contains the `ww` flag and the operation that fires the action is a write that only modifies some of the
affected addresses, addresses that are not modified by the write MUST NOT be taken into account when determining the
highest address in common.

By default, actions MUST NOT fire as long as the boot ROM is enabled.
The `b` flag inverts this behavior, making an action fire only as long as the boot ROM is enabled; the `bb` flag
instructs the emulator to fire the action regardless of whether the boot ROM is enabled or not.
Emulators that don't emulate the boot ROM (either because they lack the capability or because they have been
configured to not do so) MUST ignore all actions containing the `b` flag; actions containing the `bb` flag MUST be
handled normally.

The `s` and `ss` flags affect all expressions in the action; this includes the address specifications, the condition
expression subfield and any expressions contained in any commands within the action.

## 6. Commands

Commands indicate the actual steps taken by the emulator when an action fires.
Actions may cause the emulator to halt execution, print a debug message, and so on.
Every action has a command list containing one or more commands; these commands MUST be executed in order when the
action fires.
Command lists MAY have any number of commands, but emulators MAY impose reasonable limits in order to prevent bugs or
security issues.
Such a limit SHOULD be high enough to not interfere with the normal, intended usage of debugfiles.

Commands in an action's command list are separated by semicolons.
Semicolons are separators, not terminators; this means that the last action in the command list MUST NOT be followed
by a semicolon.
(In fact, the line ending without a colon or semicolon at the end is what marks the end of the action.)
These semicolons MAY be surrounded by arbitrary amounts of whitespace, which is ignored.

An action line MAY be broken after any of these separating semicolons, inserting a newline.
This means that the command list continues in the next line.

Each command has its own syntax and semantics.
Commands always begin with a keyword, which is a valid identifier (according to [section 4.3][section4.3]) that
identifies the command; commands can have additional arguments after the keyword, separated by one or more spaces.
For example, the [`message`][command-message] command will be followed by a string, as in `message "Hello world!"`,
but the [`break`][command-break] command always stands by itself.

### 6.1. Basic emulator commands

These commands cause the emulator to perform basic debugging functions.

#### `break` command

Syntax: `break`

This command causes a breakpoint, i.e., it instructs the emulator to stop execution and open the debugger.

This command is idempotent: if several of these commands are executed at once (because a command list contains
multiple `break` commands or because several actions containing a `break` command fire at once), they have the same
effect as a single `break` command.
This MUST NOT cause an error.

#### `reset` command

Syntax: `reset`

This command instructs the emulator to reset itself, as if the Game Boy was power-cycled.

This command MAY be implemented in an idempotent way: several `reset` commands executed at once MAY reset the emulator
any number of times between one and the number of `reset` commands executed.
The exact number of resets executed in this case is implementation-defined and it MAY vary between executions.

#### `message` command

Syntax: `message <string>`

This command causes a debug message to be printed.
The emulator MAY output debug messages in any way it wishes; it SHOULD NOT stop emulation for that purpose.

The `<string>` argument can be either a named string declared with [the `@str` directive][directive-str] or a quoted
constant string as explained in [section 7][section7].
Expression substitutions apply regardless of the chosen format.

#### `alert` command

Syntax: `alert <string>`

This command is equivalent to the `message` command, but it is intended to bring immediate attention to the message
printed.
The emulator SHOULD display the message in a way that brings the user's attention to it, such as a message box;
emulation MUST be stopped until the user acknowledges the message.

If several `alert` commands are executed at once (because a command list contains many of them or because several
actions containing an `alert` command fire at once), the emulator SHOULD combine them into a single alert, in order to
avoid stopping emulation repeatedly.

### 6.2. Action-related commands

These commands enable or disable actions.

#### `enable` command

Syntax: `enable [<group>]`

Enables all actions within a group (as declared by [the `@group` directive][directive-group]).
If some or all actions within that group are already enabled, this command does nothing to those actions; enabling
already-enabled actions MUST NOT cause an error.

The group may be omitted from the command; if it is, it enables the action where it appears.
(This can be useful if the action has been disabled by another command from the same command list, or by a command
from an action that fired simultaneously.)

#### `disable` command

Syntax: `disable [<group>]`

Disables all actions within a group; disabling already-disabled actions MUST NOT cause an error.
This is the opposite of the [`enable`][command-enable] command.

The group may be omitted from the command, in which case it disables the action where it appears.

#### `toggle` command

Syntax: `toggle [<group>]`

Toggles the enabled/disabled status of all actions within a group, enabling the ones that are disabled and disabling
the ones that are enabled.

The group may be omitted from the command, in which case it toggles the enabled/disabled status of the action where it
appears.

### 6.3. State modification commands

These commands modify the program state, by altering a variable, a memory location or the execution of the program.

#### `set` command

Syntax: `set <lvalue> := <expression>`

This command modifies a value in the program state.
The `<lvalue>` part MUST be a [variable][expressions-variables] or a [memory access expression][expressions-memory];
the `<expression>` part can be any arbitrary expression.
The `:=` symbol used as a delimiter MAY be surrounded by arbitrary amounts of whitespace, which is ignored.
If a variable name is used for the `<lvalue>` part, it MAY be prefixed with an `@` character; this prefix MUST be used
if the variable name collides with an existing symbol name.

The behavior of this command depends on the kind of `<lvalue>` used:

* If the `<lvalue>` is a memory access, a write to that address MUST be simulated; if it is a multi-byte access, the
  bytes MUST be written in ascending order of addresses.
  This may or may not cause the address to be modified; for example, an address in WRAM will be modified by the write,
  while an address in ROM will probably cause some banking command to be executed.
  This write MUST NOT cause any actions to fire.
* If the `<lvalue>` is a user-defined variable, that variable MUST be set to the full value of the expression;
  user-defined variables are always 32 bits wide.
* If the `<lvalue>` is an emulator variable, the semantics of writing to it are defined by the emulator; in
  particular, an emulator variable MAY be treated as read-only, ignoring the writes.
  For the minimal required variables listed in
  [the "Variables" heading of the Expressions section][expressions-variables], they MUST behave as follows:
    * Writes to register variables MUST update the corresponding registers (writes to the `f` and `af` registers will
      ignore the lower four bits);
    * Writes to variables representing CPU flags (`z`, `cy`, `ime`) MUST set the corresponding CPU flag to 0 if the
      expression is zero, or to 1 otherwise;
    * Writes to the `rombank` variable MUST cause a ROM bank switch, unless the program does not use ROM banking;
    * If the program uses SRAM, writes to the `srambank` variable MUST cause a SRAM bank switch;
    * If the program uses SRAM, writes to the `sramenable` variable MUST enable SRAM if the expression is non-zero or
      disable SRAM if it is zero;
    * The remaining required variables (`@`, `target`, `op`, `value`, `bank`, and the `srambank` and `sramenable`
      variables for programs without SRAM) MUST be read-only.

If the `<lvalue>` is a memory access or an emulator variable that is narrower than 32 bits, the result of evaluating
the expression MUST be truncated to fit into the available bits; overflows MUST NOT cause an error.

Undefined variables (that is, variables that aren't provided by the emulator or already defined by the user in a
[`@var` directive][directive-var]) MUST NOT be defined by a `set` command.
In other words, attempting to set an undefined variable MUST cause an error.
Since the names of existing variables are already known as the debugfile is loaded, this error SHOULD be detected
during initial read-in of the file.

#### `jump` command

Syntax: `jump <address>`

This command causes execution of the program to continue at the specified address.
The address can be any [address expression][expressions-address].

If the address is unbanked, this command merely causes execution to continue at the designated address, as if the
address had been written to the `pc` register.
If the address is banked, the emulator MUST select the designated bank for the corresponding memory area, and then
jump to the designated address.

This command MUST NOT cause any actions to fire, including actions that trigger when the designated address is jumped
to or executed.

### 6.4. Control flow commands

These commands alter the execution of other commands.
They can be used to conditionally control the execution of the command list in order to provide multiple behaviors for
an action.

While these commands can be used to skip some commands in the list, there is no way to move backwards in it.
This is an intentional design choice, which ensures that pending commands can be safely discarded by the emulator once
they have been executed or skipped.

#### `nop` command

Syntax: `nop`

This command does absolutely nothing; it can be used as a placeholder.

#### `done` command

Syntax: `done`

This command terminates the execution of the command list; any further commands within the list MUST NOT be executed
after a `done` command is executed.
This MUST NOT affect commands executed by other actions that happen to fire simultaneously.

#### `skip` command

Syntax: `skip <expression>`

This command causes a certain number of subsequent commands to be skipped.
The number of skipped commands is determined by the given expression, which MUST be a
[constant expression][expressions-constant]; this expression MUST NOT evaluate to a negative value (if it is a signed
expression) or to a value greater than the number of remaining commands in the list.

#### `if` command

Syntax: `if [<expression>]`

This command causes the following command to be skipped if the given expression evaluates to zero; it MUST NOT be the
last command in the command list.
The given expression can be any arbitrary expression.

If the expression is omitted, the `if` command behaves identically to the previous `if` command; in other words, it
skips the following command if and only if the previous `if` command did as well.
This MUST NOT cause the expression of the previous `if` command to be evaluated again.

If an `if` command without an expression is executed before any other `if` command has been executed in the same
command list, it skips the following command, as if the condition had been false.

#### `else` command

Syntax: `else`

This command causes the following command to be skipped if the last `if` command executed did not skip the command
that followed it; in other words, it inverts the condition for the last `if` command.
This MUST NOT cause the last `if` command's expression to be evaluated again.
This command MUST NOT be the last command in the command list.

If an `else` command is executed before any `if` commands are executed in the same command list, it doesn't skip the
following command, as if the last `if` command's condition had been false.

## 7. Strings

Strings are used in several contexts within a debugfile.
They appear in several directives ([`@str`][directive-str], [`@group`][directive-group],
[`@include`][directive-include], [`@symfile`][directive-symfile], [`@warning`][directive-warning] and
[`@error`][directive-error]) as well as in some commands ([`message`][command-message] and [`alert`][command-alert]).
Strings are also the only part of a debugfile (other than comments) that can contain non-ASCII characters: the full
range of UTF-8 (except C0 control characters) is allowed within a string.

### 7.1. String formatting

Strings MUST be written within quotes (`"`), and they MUST NOT contain any embedded newlines.
Simple strings are always literal, and they contain no escape sequences; these are the strings used in the various
directives that use them.

On the other hand, the [`message`][command-message] and [`alert`][command-alert] commands allow strings that contain
escape sequences, in order to incorporate the result of evaluating an expression into those strings.
[The `@str` directive][directive-str] doesn't parse the strings passed to it, but it also accepts those escape
sequences, since those strings are evaluated when used in the commands mentioned before.
Escape sequences are described in the following section.

### 7.2. Escape sequences

Escape sequences allow inserting variable text into a string, such as the result of evaluating an expression or some
text that is subject to a condition.
Escape sequences always occur between braces (`{` and `}`); these signs MUST be properly matched when using a string
in a context that allows escape sequences.

There are three kinds of valid escape sequences:

* Expression escape sequences.
  These are of the form `{<expression>}` or `{<expression>,<format>}`.
* Selection escape sequences.
  These are of the form `{<expression>:[<name>][:[<name>]...]}`
* Character replacement escape sequences.
  These are of the form `{:<character>}`.

Arbitrary whitespace is allowed (and ignored) between the tokens inside the braces for the first two forms.
Of course, whitespace outside the braces is taken literally as part of the string.

For the purposes of determining the default base in force and whether expressions are signed or unsigned, expressions
contained within escape sequences are always evaluated in the context of the command that will display the string.

If a string contains malformed or otherwise invalid escape sequences, the emulator SHOULD display an error and SHOULD
NOT perform any escape sequence substitutions on it whenever the string is displayed.

### 7.3. Expression escape sequences

Expression escape sequences are escape sequences that contain an expression and are replaced by the expression's value
when printed.
They take the forms `{<expression>}` or `{<expression>,<format>}`; the expression can be any arbitrary expression.

The `<format>` part of the escape sequence determines how the value will be formatted when printed.
It can contain a number, a formatting character, or both; if it contains both, the number MUST precede the formatting
character.
The numbers used as part of the format specifier are always decimal, regardless of the default base in force; the
format character is _not_ a base prefix in this context.
The number MUST NOT contain more than two digits.

Valid formatting characters are as follows:

|Character|Description                                                                                               |
|:-------:|:---------------------------------------------------------------------------------------------------------|
|`#`      |Unsigned decimal number.                                                                                  |
|`$`      |Hexadecimal number. Digits `A` through `F` MUST be printed in uppercase.                                  |
|`%`      |Binary number.                                                                                            |
|`-`      |Signed decimal number, prefixed with a `-` if negative.                                                   |
|`+`      |Signed decimal number prefixed with a `-` or `+` depending on its sign. The `+` sign is used for zero.    |

The number indicates how many digits will be used to print the value.
If the number is 0, the minimal number of digits needed to represent the value MUST be used; the value will never
begin with a 0 digit unless it is equal to zero.
Otherwise, the exact number of digits indicated MUST be used to print the value; the value MUST be prefixed with 0
digits if it is shorter than that length, and if it is longer, only the last digits are used.
For signed decimal numbers, the sign is not taken into account when calculating the length.

If the number is omitted, it defaults to 0.
If the format character is omitted, it defaults to the base indicated by the current base in force (as given by
[the `@radix` directive][directive-radix]); if that base is 10, the default is `#` for unsigned expressions and `-`
for signed ones.

The `<format>` part of the escape sequence MUST NOT be empty, but it MAY be left out entirely.
If it is, both defaults (number and format character) are used.

### 7.4. Selection escape sequences

Selection escape sequences print one out of a list of named strings depending on a value.
They take the form `{<expression>:<name>:<name>:<name>...}`, where the expression is followed by one or more names
separated by colons.
The expression may be any expression, and the names may either be empty or be the name of a string declared with
[the `@str` directive][directive-str].
There may be arbitrary amounts of whitespace between tokens inside the escape sequence.

The names are assigned sequential indexes, starting from 0.
The escape sequence is replaced by the named string whose index matches the value of the expression; if the expression
matches none of those indexes (because it is negative or higher than the maximum index), it is replaced by the last
named string.
If the chosen named string is empty, the escape sequence is replaced by an empty string.

The referenced named strings MUST have been previously declared in a [`@str` directive][directive-str], and the chosen
replacement string MUST be parsed for additional escape sequences.

For example, a sequence of the form `{<expression>::<name>}` will be replaced by the named string if the expression is
true, or by nothing if the expression is false.

### 7.5. Character replacement escape sequences

Character replacement escape sequences allow inserting characters that cannot otherwise be inserted in a string.
They take the form `{:<character>}`, where `<character>` is a single lowercase letter, and they are replaced by a
character that depends on the letter used.
No whitespace is allowed inside the braces for this form.

Valid replacement characters are the following ones:

|Letter|Replacement character|
|:----:|:--------------------|
|`c`   |Closing brace (`}`)  |
|`n`   |Newline              |
|`o`   |Opening brace (`{`)  |
|`q`   |Double quotes (`"`)  |
|`t`   |Tab (U+0009)         |

Newlines (i.e., the replacement for `{:n}`) are implementation-defined; implementations may use a single line feed
character (U+000A) or a carriage return (U+000D) followed by a line feed.

## 8. Example file

The following is a full example file, in order to show various parts of the format:

```
@debugfile 0.5

@ifemu fooemu < 2.0.3beta
    @warning "Your emulator is known to have issues with debugfiles; please consider updating."
@else ifemu baremu < 1.0
    @warning "Your emulator doesn't support memory watchpoints: null pointer accesses will not be logged"

@always

@sym NULL $0000

; define a string that will contain the call site for the current function (from the stack)
@str knownloc "{@rombank, 2$}:{[sp!], 4$}"
@str unknownloc "an unknown location"
@str callsite "{sp >= $c000 && sp <= $fdfe : unknownloc : knownloc}"

; {0:xyz} in a string will always include the named string xyz
@str rstmessage "RST ${@target, 2$} triggered from {0:callsite}, resetting..."

NULL,$0038    x                                   : message rstmessage; reset

NULL          rw                                  : message "Null pointer access at {@rombank, 2$}:{@, 4$}!";
                                                    if @op == 1;
                                                        break

; track iterations of this loop
@var _iter 0
@var _total 0
FuncFoo.loop  x                                   : set _iter := _iter + 1
FuncFoo.loop  x     _iter >= 5000                 :
    set _total := _total + _iter;
    message "Looped {_iter} times (accumulated: {_total})";
    set _iter := 0

; prevent running code from RAM accidentally
$8000--$FF7F  xx                                  : alert "About to jump to a routine in RAM!"

; for HRAM, we handle it a bit differently, because the OAM wait loop is there
@group hramexec "HRAM execution prevention routines"
$FF80--$FFFF  x                                   : alert "Executing code from HRAM!"
hOAMWait.done xd                                  : toggle hramexec
@endgroup

hOAMWait      xx                                  : toggle hramexec

; check for stack underflows and overflows everywhere, but alert only once
@group stackcheck
*             xd    sp < wStack || sp > wStackTop : alert "Stack overflow/underflow!";
                                                    disable
@endgroup

; make sure that the previous check is only enabled after the stack is set up
@alias InitDone "Init.initialization_done"
$0100         xbb   sp = $FFFE                     : disable stackcheck
InitDone      x                                    : if sp = wStackTop;
                                                         enable stackcheck;
                                                     else;
                                                         alert "Stack error; sp initialized to ${sp, 4$}"
```

## 9. Specification compliance

An implementation can be said to be compliant with this specification if it implements it in full, accepting any valid
debugfile and handling it like the specification describes.
This can apply to any kind of tool (for example, it should be possible to create debugfile editors that generate
correct debugfiles based on user selections), but it is mostly relevant for emulators.

However, at the time of writing, no emulator is known to implement all the debugging capabilities required by this
specification.
In order to encourage adoption of the specification and with the purpose of achieving compliance progressively as more
features are used, this specification allows for an emulator to claim "partial compliance", provided that it follows
the criteria set in the following sections.

### 9.1. Partial compliance

Partial compliance is compliance with parts of the specification, applicable to implementations that only implement
some of its features.
For instance, some directives or commands may not be recognized, or the implementation may impose additional
restrictions not allowed by the specification, such as requiring that conditional expression subfields contain only
simple expressions.

An emulator MAY claim partial compliance with this specification if it meets all of the following criteria:

1. It implements all of the minimum required features specified in the following section;
2. It documents its own name and version numbers, allowing the use of conditional inclusion directives with it; and
3. It accurately documents which features are supported and which features are missing, buggy or only partially
   implemented.

The documentation MUST make it clear to all users what parts of the specification are usable, so that users can
readily determine if they should conditionally exclude and/or rewrite some parts of their debugfiles to use them with
the emulator in question.

### 9.2. Minimum required features

In order to claim partial compliance, an emulator MUST support at least the following features of the specification:

* The identification directive, as described in [section 4.1][section4.1];
* Conditional inclusion directives, as described in [section 4.2][section4.2];
* The [`@warning`][directive-warning] and [`@error`][directive-error] directives;
* The [`break`][command-break] command; and
* The `x` flag in [the flags subfield][section5.6].

Also, there MUST be enough features supported so that an action line of some form can be written.

## 10. Closing remarks

This specification is the result of several partially-conflicting goals.
The desire is to determine a format that allows easy recording and sharing of debugging information; however, any
features supported by such a format become a burden for implementations to handle, and thus the desire for additional
features must be balanced against the cost of implementing such features.

Several considerations within the format, such as always allowing `@` prefixes for variables, come from the desire to
support automated or tool-assisted generation of debugfiles.
Tools that generate debugfiles may not have the full symbol list available at the time of writing debugging actions,
and thus the `@` prefix helps them emit the right expressions without having to assume that a symbol with some
specific name will or will not be defined.
The same applies to the various features that allow overriding defaults, such as base prefixes or the `s` and `ss`
flags; these help those tools not have to rely on their context.

Some restrictions intend to make implementing the format easier.
For instance, unary operators are only allowed at the beginning of an expression; this ensures that expressions are
always alternating sequences of values and operators, without having to detect consecutive operators.
That is why emulators that _can_ handle the additional burden are allowed to relax this restriction, which helps them
deal with accidental breaches of this rule more gracefully.

Finally, it is explicitly not the intention to turn this format into a programming language; features like loops are
intentionally left out.
User-defined variables are a concession to the utility of tracking some values while debugging a program (such as the
iteration counter in the example in [section 8][section8]).
Moving forward, this specification is not intended to incorporate additional features that would turn the debugfile
format into a full programming language, such as negative arguments to the [`skip`][command-skip] command.

## 11. Version history

Version 0.5 (26 January 2023):

* Section "2. Introduction": updated the link to RFC 2119.
* Section "3.1. Overall formatting": added an authoritative link to the UTF-8 specification.
* Section "3.2. Normalization and whitespace": required tabs to be replaced by at least one space.
* Section "3.3. Error handling": removed permission for implementations to report syntax errors in portions of the
  debugfile that have been conditionally excluded.
* Section "3.4. Basic structure":
    * Required all (non-private-use) directive names to be valid identifiers.
    * Removed a capitalized "SHOULD" when recommending conditional inclusion of private-use directives.
* Section "3.5. Conditional inclusion": removed `!` from the list of acceptable punctuation characters for emulator
  names and versions.
* Section "4.1. Identification":
    * Required specification version numbers to contain up to three numeric components, and specified that trailing
      zero components can be left out.
    * Defined the concept of "compatible versions" and allowed debugfiles to declare multiple different specification
      versions as long as they are compatible.
* Section "4.2. Conditional inclusion":
    * Allowed implementations to obey `@debugfile` directives inside conditionally-excluded regions.
    * Added `@if`, `@ifdef` and `@ifnotdef` directives.
    * Expanded `@else` to take a condition as an optional argument, allowing chains of conditions.
    * Added `==` and `!=` as alternate operators for `=` and `<>` in `@ifemu` and `@ifnotemu`.
* Section "4.3. Declarations":
    * Allowed `#` as an identifier character.
    * Allowed implementations to warn on `@sym` and `@alias` directives that override external symbols.
* Section "4.6. Configuration": updated the `@signedness` directive to use explicit `signed` and `unsigned` tokens
  instead of digits.
* Section "5.3. Expressions":
    * Required numeric constants to contain valid digits only.
    * Required implementations to accept past-the-end symbols at the end of a memory region with a bank number
      referring to that region.
    * Added the `bank` variable.
    * Allowed emulators to select different widths for different programs for variables with a width of "any".
* Section "5.5. The address subfield": removed a recommendation for multiple address ranges not to overlap.
* Section "6. Commands": noted that command keywords are valid identifiers.
* Section "6.3. State modification commands":
    * Added `jump` command.
    * Noted that the new `bank` variable must be treated as read-only by the `set` command.
* Section "7.2. Escape sequences":
    * Renamed conditional escape sequences to selection escape sequences.
    * Modified the format of expression and selection escape sequences.
* Section "7.3. Expression escape sequences": changed the delimiter from a colon to a comma.
* Section "7.4. Selection escape sequences": renamed from "Conditional escape sequences" and rewritten entirely.
* Section "8. Example file":
    * Updated the version number in the example.
    * Added an example of the new `@else` syntax.
    * Updated the formatted strings to match the new syntax.
* Reformatted the Markdown source of the specification to add line breaks after every full stop.
* Fixed some minor text errors.

Version 0.4.1 (4 October 2020):

* Section "4.1. Identification": clarified the behavior of `@debugfile` directives in included files.
* Section "4.3. Declarations":
    * Required reinitialization of variables declared with the `@var` directive when the emulator is reset.
    * Reworded the definition of the `@str` directive to make its purpose clearer.
* Section "5.1. Basic action structure": capitalized an RFC2119 keyword to give it keyword strength.
* Section "5.3. Expressions":
    * Specified the behavior of numeric constants with the upper bit set in signed expressions.
    * Required operators to truncate down overflowing results to 32 bits, and specified two's complement arithmetic
      for signed operations.
    * Noted that address expressions are evaluated at full 32-bit precision before truncation.
* Section "5.5. The address subfield": added a note stating the difference between the `*` operator and an address
  specification of `*`.
* Section "7.2. Escape sequences":
    * Specified that expressions in escape sequences are evaluated in the context of the command that will display 
      them.
    * Added a statement recommending escape sequence error handling.
* Section "8. Example file": updated the version number in the example.
* Fixed some minor writing and formatting issues.

Version 0.4 (2 October 2020):

* Section "3.1. Overall formatting": made the prohibition of CRs not followed by LFs explicit.
* Section "3.3. Error handling":
    * Mentioned and referenced section 9.1 (Partial compliance) when describing the behavior of emulators that don't
      implement the full specification.
    * Added a restriction on reporting syntax errors on excluded portions of the debugfile, forbidding the reporting
      of errors in excluded private use lines.
    * Added a short description of errors that are not syntax errors.
* Section "4.1. Identification": required implementations to fail if the `@debugfile` directive is missing or if it
  indicates unsupported or conflicting version numbers.
* Section "4.2. Conditional inclusion":
    * Noted that conditional inclusion directives in parent files and included files don't interact.
    * Defined a new syntax for simultaneous version comparisons for the `@ifemu` and `@ifnotemu` directives, replacing
      the previous version range syntax.
    * Added a note for the `@ifemu` directive stating that an emulator name and version is equivalent to a version
      equality comparison.
* Section "4.3. Declarations":
    * Noted that later directives can make use of declarations from earlier directives.
    * Updated the `@sym` directive to take a constant address expression as the symbol's value instead of custom
      syntax.
    * Changed the `@var` directive to use constant expressions for initial values instead of numeric literals.
    * Added a comment for the `@var` directive noting that variable names that don't begin with underscores are
      reserved.
* Section "4.5. Included files": added a syntax specification for the `@symfile` directive, since symfiles no longer
  follow the syntax of the `@sym` directive.
* Section "5.1. Basic action structure": changed the wording referring to addresses, since actions may now contain
  more than one address or range.
* Section "5.3. Expressions":
    * Created the "Constant expressions" and "Address expressions" subsections.
    * Memory accesses: updated to point to the new address expression subsection.
* Section "5.5. The address subfield":
    * Updated the address specification explanation to point to the new address expression subsection.
    * Added the possibility of giving two or more address specifications separated by commas.
    * Reorganized the whole section for better clarity of exposition.
    * Resolved a parsing ambiguity related to `++` and `--` delimiters for implementations that allow consecutive
      operators.
* Section "5.6. Flags":
    * Reworded the description of attribute flags to remove a capitalized "OPTIONAL" keyword that shouldn't have been
      capitalized.
    * Added a more thorough description of the behavior of the `xx` flag.
    * Noted that the `s` and `ss` flags affect all expressions in the action, including addresses and expressions
      contained in commands.
* Section "6.2. Action-related commands": uncapitalized some "MAY" keywords that really represent user-facing options,
  not implementation alternatives.
* Section "6.3. State modification commands", `set` command:
    * Noted that writing to the `af` register is subject to the same constraints as writing to `f`.
    * Fixed a reference to CPU registers to correctly mention CPU flags instead.
    * Relaxed the restriction about detecting invalid variable names at read-in from MUST to SHOULD.
* Section "7.3. Expression escape sequences": inverted the order of the formatting character and the number indicating
  the precision.
* Section "7.4. Conditional escape sequences": noted that the included strings must be parsed for escape sequences.
* Section "8. Example file":
    * Updated the file according to the changes made to the specification.
    * Added an action line with two address specifications to exemplify the use of commas to delimit them.
    * Added conditional escape sequences, illustrating their use.
    * Added the `bb` flag to the action that fires on execution of $0100.
* Fixed various spelling and grammatical errors.
* Added some additional links between sections to improve navigation.

Version 0.3 (20 September 2019):

* Section "4.3. Declarations":
    * Removed the requirement that variables declared with `@var` must not collide with pre-existing symbols.
    * Added the `@alias` directive.
* Section "5.3. Expressions":
    * Noted that parentheses don't count for the "first token rule" used to determine whether a memory access is
      banked or unbanked.
    * Added a note stating that symbols may also come from an `@alias` directive, not just a `@sym` directive.
    * Required symbol names to be valid (by declaration rules), noting that aliases are necessary for invalid ones.
    * Modified the "bank of" (`&`) operator, making it return 0 for unbanked symbols instead of -1.
* Section "5.6. Flags":
    * Added a note requiring CPU-stopping instructions (`halt`, `stop`) to be handled as single-byte instructions,
      undoing part of the change introduced in the previous version.
    * Added the `b` and `bb` flags, defining behavior regarding the boot ROM.
* Section "7.2. Escape sequences":
    * Explained the purpose of escape sequences.
    * Switched the escape delimiters from percentage signs to braces in order to avoid a parsing ambiguity.
    * Added character replacement escape sequences to the list of possible escape sequences.
* Section "7.3. Expression escape sequences": adjusted the text to reflect the new escape delimiters.
* Section "7.4. Conditional escape sequences": adjusted the text to reflect the new escape delimiters.
* Section "7.5. Character replacement escape sequences": new section.
* Section "8. Example file":
    * Updated the version number in the example.
    * Adjusted the sample strings in order to reflect the new escape delimiters.
    * Added a `@sym` and an `@alias` directive to show usage examples.
* Fixed some minor grammatical and related issues.

Version 0.2 (16 April 2019):

* Section "1. Objective and scope": added a note stating that compilers, not just assemblers, can generate debugfiles.
* Section "3.3. Error handling": allowed implementations to error-check the main file if a referenced file is
  unavailable.
* Section "4.1. Identification": added requirements for this specification's version number, to be used in the
  `@debugfile` directive.
* Section "5.3. Expressions":
    * Added a requirement stating that undefined symbols/variables in an expression are an error.
    * Explained the difference between the `@` and `pc` variables.
    * Required memory accesses not to trigger any actions.
    * Defined properly a banked memory area.
    * Recommended that symbols in unbanked memory areas with a bank number of 0 be treated as unbanked symbols.
* Section "5.5. The address subfield": referenced the definition of banked memory area given above.
* Section "5.6. Flags":
    * Required actions to fire before the read, write, execution or jump.
    * Specified that flags can be given in any order.
    * Explained the interaction of combined read/write operations and actions that specify the `r` and `ww` flags.
    * Explained the interaction of multi-byte instructions that partially lie within an action's address range and the
      `x` and `xx` flags.
    * Added a requirement about the way undefined instructions and corrupted stops are handled.
    * Noted that instruction execution doesn't cause actions that only specify the `r` flag to fire.
* Section "6. Commands": noted that the command name can be separated from the arguments by any number of spaces.
* Section "6.3. State modification commands": for the `set` command, required simulated memory writes to not trigger
  any actions.
* Section "6.4. Control flow commands": for the `skip` command, added a note stating that the expression must not be
  negative if it is signed, since unsigned expressions cannot ever be negative.
* Section "7. Strings": noted that the `@error` directive also uses a string argument.
* Section "8. Example file": added additional examples and reformatted the file to keep it consistent.
* Capitalized some RFC2119 keywords to give them keyword strength.
* Fixed various minor writing and formatting issues.
* Added some additional links between sections to improve navigation.

Version 0.1 (15 April 2019):

* Initial draft version.

* * *

This document is hereby released to the public domain; no copyright is claimed on it. Attribution is appreciated. The
latest version of this document lives in [the `gb-debugfiles` repository][repo].

[repo]: https://github.com/aaaaaa123456789/gb-debugfiles
