# Debugfile format specification

Version 0.2 — 16 April 2019

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
        * [`@ifemu`][directive-ifemu]
        * [`@ifnotemu`][directive-ifnotemu]
        * [`@else`][directive-else]
    * [4.3. Declarations][section4.3]
        * [`@sym`][directive-sym]
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
        * [Numeric constants][expressions-constants]
        * [Symbols][expressions-symbols]
        * [Variables][expressions-variables]
        * [Memory accesses][expressions-memory]
        * [Operators][expressions-operators]
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
    * [7.4. Conditional escape sequences][section7.4]
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
[section7.4]: #74-conditional-escape-sequences
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
[command-message]: #message-command
[command-nop]: #nop-command
[command-reset]: #reset-command
[command-set]: #set-command
[command-skip]: #skip-command
[command-toggle]: #toggle-command
[directive-always]: #always
[directive-debugfile]: #debugfile
[directive-else]: #else
[directive-endgroup]: #endgroup
[directive-error]: #error
[directive-group]: #group
[directive-ifemu]: #ifemu
[directive-ifnotemu]: #ifnotemu
[directive-include]: #include
[directive-radix]: #radix
[directive-signedness]: #signedness
[directive-str]: #str
[directive-sym]: #sym
[directive-symfile]: #symfile
[directive-var]: #var
[directive-warning]: #warning
[expressions-constants]: #numeric-constants
[expressions-memory]: #memory-accesses
[expressions-operators]: #operators
[expressions-symbols]: #symbols
[expressions-variables]: #variables

* * *

## 1. Objective and scope

The present document describes a textual file format that contains information regarding debugging actions to be taken
when executing a Game Boy or Game Boy Color program. The main intended consumer of this information is the emulator
executing the program; while hardware debuggers aren't explicitly ruled out, they are not the intended audience for
this file format.

This specification does not intend to provide an origin for those debugging actions; they may be manually typed up or
generated by some code generation tool (e.g., an assembler or a compiler).

## 2. Introduction

In order to debug Game Boy programs, developers may need to specify a list of debugging actions that the emulator must
take on their behalf. These actions are simple commands, such as halting execution or printing information to a debug
console, that are taken when a certain condition is true, such as reading from or executing a certain memory address.

It therefore becomes necessary to have a common way to specify and persist these actions, so that they can be supplied
to the emulator on start-up (instead of having to manually enter them every time the program is debugged); this also
allows for tools (such as assemblers) to generate these lists of actions, and it enables an easy way of sharing them.

In order to enable the easy generation and editing of these lists of actions, a textual format was chosen for them,
which enables simple modifications in a text editor. Therefore, this document describes such a textual format, a
_debugfile_, to be used alongside existing metadata files (such as symfiles) as a debugging aid.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",
"NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119][rfc2119]
when, and only when, they appear in all capitals, as shown here.

[rfc2119]: https://tools.ietf.org/html/rfc2119

## 3. Format basics

This section describes the basic rules and constraints that make up the debugfile format. The actual contents are
defined in a later section.

### 3.1. Overall formatting

Debugfiles MUST be plain text files, encoded in UTF-8, without BOM. (Note that US ASCII, being a subset of UTF-8, is a
valid encoding.) There must be no C0 control characters (that is, characters with codepoints below 32) other than line
feeds (codepoint 10), horizontal tabulations ("tabs", codepoint 9) and carriage returns (codepoint 13).

The file is broken down into lines, which are delimited by line feed characters. The last line of the file MAY end
with a line feed character; this MUST NOT cause any differences in interpretation of the file.

Carriage return characters MAY appear before line feed characters; if they do, they MUST be considered to be part of
the line delimiter. A carriage return character not immediately followed by a line feed character is an error.

### 3.2. Normalization and whitespace

Normalization describes a process that eliminates superfluous changes to a debugfile. While emulators interpreting
such files are not required to actually perform normalization, they MUST interpret the file "as if" normalization had
happened — for instance, by ignoring trailing whitespace as if it had been eliminated by normalization.

If normalization is actually performed, the file provided by the user MUST NOT be overwritten with a normalized
version unless the user explicitly requests this.

Normalization is applied to each line of the file individually, and it involves the following steps:

1. Replace all tab characters by an implementation-defined number of space characters per tab. This number MUST be the
   same for all tab characters in the file. Consecutive tab characters MUST be replaced individually: for instance,
   if an implementation replaces each tab with 5 spaces, 3 tab characters in a row must be replaced with 15 spaces.
2. Delete all leading and trailing spaces from the line.
3. If the line is now empty, or if its first character is a semicolon, discard it. (A line whose first non-whitespace
   character is a semicolon is considered a comment.) Discarded lines are not considered at all by any further
   processing of the file: for instance, any reference to the "next line" ignores discarded lines, and the requirement
   for a `@debugfile` directive in the first line of the file (explained in [section 3.4][section3.4]) applies to the
   first line that hasn't been discarded.

### 3.3. Error handling

Several different kinds of errors can be detected while handling a debugfile.

The most obvious kind of error is a syntax error, which consists of some content in the file that is not conformant to
this specification. If such an error is detected, the implementation MUST report an error to the user and MUST NOT
use the file other than for further error detection; the error SHOULD tell the user the location of the invalid
content and the reason why it was considered invalid.

If the file references other files (for instance, through a `@symfile` directive), and those files are not available,
the implementation MUST report this to the user; it SHOULD NOT use the contents of the main file in that case other
than for further error detection. Syntax errors in referenced files MUST be treated like syntax errors in the main
file.

It is also possible that some emulator will not implement the full range of functionality described in this
specification. If this is detected when the file is initially read in, the emulator SHOULD inform the user that some
functionality specified in the file is not supported; the emulator MAY ignore those actions or directives it doesn't
implement and only follow the ones it does. The emulator MAY also treat this as a hard error, in which case it MUST
report the error to the user on initial read-in and disregard the data in the file. Any errors reported due to missing
functionality SHOULD indicate which functionality was requested by the file that the emulator does not implement.

If a portion of a debugfile is excluded by a conditional inclusion directive, implementations MUST NOT report any
errors for unsupported functionality within the excluded portion. Implementations MAY choose to report syntax errors
within those portions.

### 3.4. Basic structure

Lines that begin with a `@` character are called "directives". They are intended to be processed immediately (i.e.,
during the initial read-in of the file), and they perform several utility functions, such as defining additional
symbols or setting the default base for numeric constants. Each directive has a name, which MUST immediately follow
the `@` character (without any whitespace); unrecognized directive names are a syntax error.

Lines that begin with multiple `@` characters (at least two) are reserved for private use, for emulators that need to
define their own extensions to the format. These lines SHOULD always be conditionally included, in order to avoid
errors. Implementations MUST NOT issue syntax errors for these lines if they are excluded by a conditional inclusion
directive, regardless of content. The semantics of private use lines are implementation-defined.

Any other lines are called "actions". An action line specifies an action that the emulator must take when a certain
event occurs and some conditions are satisfied. Actions are usually specified entirely in one line, but they may be
broken down into several; if an action line ends in a colon or semicolon, it continues on the next line. If an action
line is continued into the next line, the next line MUST be an action line as well; if it isn't (i.e., if the next
line is a directive or a private use line, or if the continued line is the last line of the file), a syntax error
occurs.

The very first line of the file MUST be [a `@debugfile` directive][directive-debugfile]; this serves both as a
versioning system for the format and as a way of telling debugfiles apart. If this directive is missing, a syntax
error occurs.

### 3.5. Conditional inclusion

Some directives are used to control conditional inclusion. These directives specify a condition; the following lines
MUST be ignored if the condition is not met. All conditions are evaluated on initial file read-in. Conditions remain
active until a new conditional inclusion directive appears or until the end of the file is reached.

Since there is no "end of condition" directive, conditional inclusion directives cannot be nested. This is
intentional. Conditional inclusion directives themselves are never conditional.

Emulators must define a (hopefully unique) name and version for themselves. Conditional inclusion directives can refer
to these values in order to tailor debugfiles to specific emulators. Emulator names and versions MUST be between 1 and
50 characters long, and they MUST contain only ASCII letters, digits, and the characters `!#$%&*+-.?@_`; the name MUST
begin with a letter and the version MUST begin with a digit. Emulators MUST also be able to compare their own version
numbers in order to determine which one is greater. Subject to these restrictions, emulator names and versions are
arbitrary and chosen by their authors; emulator names SHOULD be chosen making a reasonable effort to keep them unique.

All implementations MUST fully support all conditional inclusion directives, as these are the only mechanism that can
be used to prevent the use of unsupported features in partially-compliant emulators.

## 4. Directives

Directives perform various utility functions within a debugfile. They are processed immediately during initial file
read-in, and they can affect how the remainder of the file is processed. Directives are meant to have a support role
within this specification; they are described before actions because some directives can affect the actions that come
after them.

All valid directives are listed in this section, indicating their syntax and semantics. Any directives not listed here
MUST NOT be used; they are syntax errors.

### 4.1. Identification

This directive identifies the file as a debugfile.

#### `@debugfile`

Syntax: `@debugfile <version>`

This directive identifies the file as a debugfile, and indicates which version of the specification is being followed.
The version number for this specification is given at the top of the document; valid version numbers always begin and
end with a digit, contain only digits and period characters (`.`), and are no longer than 20 characters.

This directive MUST appear as the first line of the file. If it appears more than once in the file, all instances of
it MUST indicate the same version number.

### 4.2. Conditional inclusion

These directives allow [conditional inclusion][section3.5] of parts of the file. They remain in effect until the end
of the file or until a new conditional inclusion directive is found; conditional inclusion directives are never
conditional themselves, and thus nesting them is explicitly not possible.

As long as a conditional inclusion directive is in effect, implementations MUST ignore any lines (other than other
conditional inclusion directives) if the condition given by the directive is false.

These directives MUST be supported by all implementations, since they are the only mechanism available to exclude
features not supported by some particular implementation.

#### `@always`

Syntax: `@always`

This is a conditional inclusion directive whose condition is always true. In other words, this directive cancels all
previous conditions for conditional inclusion, indicating that the following lines (up to the following conditional
inclusion directive or the end of the file, whichever comes first) MUST be processed.

#### `@ifemu`

Syntax: `@ifemu <emulator spec> [, <emulator spec> [, <emulator spec> ...]]`

This is a conditional inclusion directive that is true if the emulator matches the given emulator specification.
Several specifications can be given, separated by commas; if that is the case, the condition is true if the emulator
matches any of the given specifications. Emulator specifications are matched against the name and version of the
emulator; valid values for names and versions are detailed in [section 3.5][section3.5].

An emulator specification can have one of four formats:

* Just the emulator name. The specification matches if the specified name is equal to the emulator's name. There are
  no wildcards or regular expressions — the name must match exactly. Name matching is case-insensitive.
* Emulator name and version, separated by one or more spaces; for example, `fooemu 3.8`. The specification matches if
  the specified name is equal to the emulator's name and the specified version is considered equal to the emulator's
  version.
* Emulator name, comparison operator and version; for example, `fooemu > 3.6`. There MUST be at least one space
  between the name and the operator; there MAY be spaces between the operator and the version. Valid operators are
  `<` (earlier than), `>` (later than), `=` (equal), `>=` (later or equal), `<=` (earlier or equal) and `<>` (not
  equal). The specification matches if the specified name is equal to the emulator's name and the comparison between
  the specified version and the emulator's version, as given by the comparison operator, is true.
* Emulator name and two version numbers; for example, `fooemu 3.5 3.8`. There MUST be at least one space between each
  pair of components of the specification. The specification matches if the specified name is equal to the emulator's
  name and the emulator's version is strictly later than the first specified version number and strictly earlier than
  the second one; in other words, this is a version range comparison. The endpoints are intentionally excluded from
  the range; they can be included by naming them in an additional specification (for example,
  `fooemu 3.5, fooemu 3.5 3.8, fooemu 3.8` includes both endpoints).

Since versions are only checked when the names match, version comparisons can be handled by each emulator in any way
they deem suitable. Each emulator MUST be able to determine which one of two valid version numbers is later than the
other, in order to be able to resolve the comparison; if the specified version is not a valid version number (i.e., it
doesn't match the formatting rules for that emulator), the comparison MUST be considered to fail. In particular,
invalid version numbers MUST NOT raise any errors.

If version numbers contain alphabetical characters, version comparisons SHOULD be handled in a case-insensitive manner
(for example, by case-folding the version number string).

Note that the characters `<`, `>` and `=` are not allowed in either emulator names or version numbers. Additionally,
version numbers are required to begin with a digit. Therefore, it is always possible to split the version number from
the comparison operator in an unambiguous way.

If more than one emulator specification is given, there MAY be spaces around the commas used to separate them. These
spaces MUST be ignored when processing the emulator specifications.

#### `@ifnotemu`

Syntax: `@ifnotemu <emulator spec> [, <emulator spec> [, <emulator spec> ...]]`

This is the negated form of the `@ifemu` directive. The condition is true if the emulator matches none of the given
emulator specifications.

All considerations given for the `@ifemu` directive are valid for `@ifnotemu`.

#### `@else`

Syntax: `@else`

This is a conditional inclusion directive whose condition is true if the previous conditional inclusion directive's
condition was false; in other words, it negates the previous conditional inclusion directive in the file.

The `@else` directive MUST NOT be the first conditional inclusion directive in the file, and it MUST NOT appear twice
in a row. For the purpose of this rule, "twice in a row" implies that there are no other conditional inclusion
directives between both instances.

### 4.3. Declarations

These directives declare elements so that further action lines can use them. Declarations MUST appear before the first
use of the declared elements.

Every declared element has an identifier. Identifiers MUST be purely ASCII strings, and they MUST only contain
letters, numbers, and the following characters: `$.@_`. Additionally, their first character MUST be a letter or an
underscore. Identifiers are case-sensitive; each declared identifier MUST be unique.

#### `@sym`

Syntax: `@sym <name> <address>`

This directive declares a symbol; any further reference to the declared symbol MUST use the address declared by this
directive. Additionally, if the emulator keeps a list of symbols for the loaded program (e.g., loaded from a symfile),
it SHOULD add the declared symbol to that internal list.

If the declared symbol collides with a symbol loaded from an implicit source such as a symfile, the address given in
this directive takes precedence. However, the symbol MUST NOT collide with any other symbol declared in the debugfile.

The address can be either a single 16-bit value, or a pair of values separated by a colon; in the latter case, the
first value indicates the bank, and the second one indicates the address. The address given MUST fit into 16 bits; if
a bank is given, the bank number MUST be valid for that address.

The values given for the bank and address are hexadecimal, regardless of the default base in force for numeric
constants. Banks and addresses that begin with the characters `A` through `F` (or their lowercase equivalents) may be
freely specified without the requirement for a leading digit used for numeric constants elsewhere.

If a bank number is given for the symbol, the symbol is considered banked; otherwise, it is an unbanked symbol.

#### `@var`

Syntax: `@var <name> <value>`

This directive declares a variable to be used in expressions. The value MUST be a numeric constant and it MUST fit
into 32 bits; it MAY be preceded by a `+` or `-` sign. The name MUST begin with an underscore and it MUST NOT collide
with any existing symbol or variable.

The value is specified in the default base in force, as explained in [the `@radix` directive][directive-radix]. If a
different base is desired, the value can be prefixed by the `%`, `#` or `$` characters to respectively indicate base
2, 10 or 16. If a both a sign and a base indicator character are present, the base indicator MUST come after the sign.

If the value doesn't contain a base indicator character, it MUST begin with a digit. (The sign character, if present,
is not taken into account for this requirement.) If the default base in force is 16, hexadecimal constants that begin
with the characters `A` through `F` can be given by prepending a zero — for instance, a variable with the highest
possible 32-bit value can be declared like so: `@var _uintmax 0FFFFFFFF` (assuming that the default base in force is
16).

#### `@str`

Syntax: `@str <name> <value>`

This directive declares a string to be used in contexts that require one, such as debug messages. Strings live in
their own namespace; the given name MUST NOT collide with any other string declared in the debugfile, but it MAY be
equal to some symbol or variable.

The value MUST be a constant string, between quotes. The format for the value is identical to the format used in
commands that take a string as an argument; refer to [section 7][section7] for further details.

### 4.4. Action groups

These directives allow grouping actions into groups, in order to be able to execute actions upon those named groups
(such as disabling all actions within a group). Actions that come before a `@group` directive are not considered to
belong to any group.

#### `@group`

Syntax: `@group <name> [<display name>]`

This directive declares that the actions that follow it belong to the specified group. If there was already a `@group`
directive in effect, this directive replaces it; it is not necessary to explicitly close groups with `@endgroup`
before opening a new group.

The name of the group is an identifier that must follow the same rules given in [section 4.3][section4.3]. If the same
group name is given multiple times in a debugfile, all of the actions covered by the corresponding `@group` directives
are considered to belong to the same group; in other words, reusing a group name appends actions to that group.

The display name is an OPTIONAL constant quoted string; no escape characters are allowed within it. If this string is
given, and the emulator allows grouping debugging actions, it SHOULD use the given display name as the name for the
group.

If multiple `@group` directives refer to the same group name, and at least two of them specify a display name for the
group, they MUST all specify the same display name. If different display names are specified for the same group, the
emulator MAY, at its discretion, raise an error or use any of the display names given.

#### `@endgroup`

Syntax: `@endgroup`

This directive closes a `@group` directive; actions that follow will not be considered to belong to any group.

### 4.5. Included files

These directives allow a debugfile to refer to other files. The specified file paths may be relative or absolute.
Implementations MAY impose restrictions on allowable file paths for security reasons.

File paths MUST be given as quoted strings; no escape sequences are allowed within them.

#### `@include`

Syntax: `@include <filepath>`

This directive includes a separate debugfile as part of the one where it appears. Circular inclusions (that is, chains
of inclusions that result in a file directly or indirectly including itself) are not allowed; it is RECOMMENDED that
implementations check for this.

#### `@symfile`

Syntax: `@symfile <filepath>`

This directive includes a symfile into the debugging information for the loaded program. If the included symfile has
already been automatically loaded by the implementation, it MAY reload the symbols.

A symfile is a text file where each nonempty line contains either a comment or a symbol declaration. Comments begin
with zero or more whitespace characters followed by a semicolon; they are ignored. Symbol declarations contain an
address and a symbol name separated by one or more whitespace characters; the address is hexadecimal and follows the
same syntax given for [the `@sym` directive][directive-sym].

Implementations MAY automatically load symfiles even in the absence of any `@symfile` directives; this is part of the
normal behavior of debuggers.

### 4.6. Configuration

These directives allow configuring some internal parameters of the debugfile, such as the default base in force for
numeric constants. They take effect immediately for the directives and actions that follow them; they may be given
several times to change the configuration at different points in the file.

Included files MUST have a configuration separate from their parent files; in other words, they neither inherit nor
modify the configuration from the file that includes them.

The defaults for each configuration option are stated in the description of the directives themselves.

#### `@radix`

Syntax: `@radix 2|10|16`

Default: 10

This directive specifies the default base in force for numeric constants. Numeric constants that don't contain a
prefix character indicating their base will use this value as their base. Valid values are 2, 10 and 16.

#### `@signedness`

Syntax: `@signedness 1|0`

Default: 0

This directive specifies whether arithmetic expressions in actions will use signed or unsigned arithmetic; this
affects divisions and right shifts. Valid values are 0 (off, i.e., use unsigned arithmetic) and 1 (on, i.e., use
signed arithmetic).

### 4.7. Miscellaneous

These directives don't fit in any of the above categories.

#### `@warning`

Syntax: `@warning <message>`

This directive contains warning text for the user; this is intended to be used within a conditionally included section
to warn the user about missing features or actions. The emulator SHOULD display the warning message to the user during
initial read-in of the file, and it SHOULD allow the user to abort the load of the debugfile.

The message MUST be a quoted string; no escape sequences are allowed within it.

#### `@error`

Syntax: `@error <message>`

This directive is equivalent to the `@warning` directive, but it generates a hard error. If this directive is
encountered and not excluded by a conditionally inclusion directive, the error text MUST be shown to the user and
the debugfile MUST NOT be loaded.

## 5. Actions

Actions are a series of responses that the user requests the emulator to automatically take when a certain condition
is met, such as stopping emulation when a particular memory address is written to. Actions are the main focus of this
specification; the goal of a debugfile is to define these actions for a particular program.

### 5.1. Basic action structure

An action is comprised of two components: a _condition_ and one or more _commands_. The condition indicates when the
action will execute, and the commands indicate what the action will do when it executes.

The condition is bound to an address or range of addresses, and it triggers when the addresses in question are read,
executed or written to. The least restrictive condition is one that is true for all reads, writes and executions; such
a condition would trigger at least once per instruction executed.

Commands are the actual steps that the emulator must take when the condition becomes true. Commands are simple
debugging steps, such as stopping execution and breaking into the debugger, or printing a debug message to some debug
console. An action can have any number of commands, which MUST be executed in sequence.

It is possible for many actions' conditions to be true simultaneously. If this is the case, the emulator MUST queue
all of them for execution in any order it wishes. The commands for each action MUST be executed in sequence, and they
MUST NOT be interleaved with other actions' commands.

An action can be enabled or disabled. Actions are enabled by default, unless their flags set them as disabled. A
disabled action behaves as if its condition was always false, i.e., it never triggers.

Emulators may offer the ability to edit some or all of the properties of an action. For instance, they may allow
enabling or disabling actions, as well as deleting them. An edited action SHOULD behave as if it had been entered into
the debugfile in whatever form it results from the edits; in other words, such edits SHOULD take precedence over the
originally-entered data in the debugfile.

### 5.2. Syntax overview

The overall syntax of an action line is as follows:

```
<condition>: <command> [; <command> [; <command> ...]]
```

There MAY be zero or more whitespace characters at either side of the colon and semicolons; such whitespace characters
are meaningless and ignored. Also, an action MAY be broken down into more than one line right after the colon and/or
any of the semicolons shown above, as explained in [section 3.4][section3.4]; action lines MUST NOT be broken down
anywhere else. The trailing colon or semicolon at the end of a line indicates that the action has been broken down
into several lines and that the next line continues the same action.

If an action is broken down into several lines, these lines MUST be consecutive; in other words, after an action line
ending with a colon or semicolon, the action MUST be continued on the immediately following line. (Note that blank
lines and comment lines are stripped by the normalization process described in [section 3.2][section3.2], and thus
they are not considered for this rule; if the second line is separated from the first by blank and/or comment lines
only, it is still considered to immediately follow the first line.) An action line ending with a colon or semicolon
MUST NOT be the last line of the debugfile and it MUST NOT be followed by a directive or a private use line.

Semicolons are used as command delimiters, not command terminators. In particular, this means that the last command of
an action MUST NOT be followed by a semicolon. Additionally, all actions MUST have at least one command. There is no
explicit upper bound on the number of commands per action, but implementations MAY impose a reasonable limit. Such a
limit SHOULD NOT interfere with intended uses of debugfiles.

### 5.3. Expressions

Many components within an action use expressions to specify values. Expressions have similar syntax regardless of
where they appear, and thus they are described here.

Expressions use a single data type: 32-bit integers. All values in an expression are 32-bit integers, and operands
manipulate those values. Expressions can contain numeric constants, symbols, variables, memory accesses and operators.
All operators that result in a signed value use two's complement arithmetic.

Expressions may be signed or unsigned; this is determined by the action's flags. This only affects some operators and
variables; the exact cases where the signedness makes a difference are indicated below. For example, division is
signed or unsigned depending on the signedness of the expression where it appears.

An expression is said to be constant if it involves no variables and no memory accesses.

#### Numeric constants

Numeric constants are sequences of digits, optionally prefixed by a base indicator character, that evaluate to a
constant non-negative numerical value. The value of a numeric constant MUST fit in 32 bits.

Numeric constants can be prefixed with the characters `%`, `#` or `$` to indicate that they are respectively in base
2, 10 or 16; if no prefix is present, the constant is in the default base in force, as given by
[the `@radix` directive][directive-radix]. For instance, the constants `%10100`, `#20` and `$14` all specify the same
value, and so does the constant `20` if the default base in force is 10. The hexadecimal digits `A` through `F` can
be written in uppercase or lowercase.

Numeric constants that are not prefixed by a base indicator character MUST begin with a digit. This is only relevant
if the default base in force is 16; in that case, a numeric constant MUST NOT begin with the digits `A` through `F`.
This can be avoided by adding an additional `0` in front, or by explicitly prefixing the constant with a `$`
character: for example, the value `FF` can be written as `0FF` or `$FF`.

#### Symbols

Symbols are constants, specified in a symfile or explicitly in [a `@sym` directive][directive-sym], that evaluate to
an address. The address in question may be banked or unbanked; a banked address contains an explicit bank number in
addition to the address portion. Unless otherwise stated in a specific context, bank numbers are generally ignored;
banked symbols evaluate to their address portions when used in an expression.

Symbols always evaluate to an unsigned value, i.e., they are zero-extended into 32 bits.

A symbol's name can collide with a variable's name. If this is the case, using the name in an expression MUST evaluate
to the symbol, not the variable. The use of a name in an expression that is neither a valid symbol name nor a valid
variable name MUST be considered an error.

If a symbol's address portion falls within an unbanked memory area and its bank number is zero, it SHOULD be treated
as an unbanked symbol in order to facilitate its usage. (Banked memory areas are defined in the subsection regarding
[memory accesses][expressions-memory].)

#### Variables

Variables are values that represent some part of the program's state, such as a register. They come from two sources:
the emulator provides a fixed set of variables that the user can use, and the user can define additional variables
through [the `@var` directive][directive-var]. User-defined variables only change when the user explicitly changes
them (for instance, via a command); emulator variables can also change to reflect changes in the program state. All
user-defined variables MUST begin with an underscore; emulator variables MUST NOT begin with one.

A variable's name can collide with a symbol's name; in this case, using the name in an expression MUST evaluate to the
symbol, not the variable. In order to allow using the variable in this case, the name can be prefixed with a `@`
character to indicate that it SHALL evaluate to the variable. Variable names MAY be prefixed with `@` even if there
is no collision — for example, the expression `@bc` MUST evaluate to the `bc` variable, regardless of whether a `bc`
symbol exists. The variable `@` is an exception, and it MUST NOT be prefixed with an additional `@`.

The use of a name in an expression that is neither a valid variable name nor a valid symbol name, as well as the use
of a `@`-prefixed name that is not a valid variable name, MUST be considered an error.

Emulators MUST provide at least the variables indicated below. Emulators MAY provide any additional variables they
wish, including alternate casings for the following variables, provided that none of them begin with an underscore.

|Variables                        |Bits|Signedness|Description                                                       |
|:--------------------------------|---:|:--------:|:-----------------------------------------------------------------|
|`a`, `b`, `c`, `d`, `e`, `h`, `l`|   8|Context   |CPU single-byte registers.                                        |
|`f`                              |   8|Unsigned  |CPU flags register; it is always unsigned.                        |
|`af`, `bc`, `de`, `hl`           |  16|Context   |CPU register pairs.                                               |
|`sp`, `pc`                       |  16|Unsigned  |CPU two-byte registers; they are always unsigned.                 |
|`z`, `cy`                        |   1|Unsigned  |Zero and carry flags. Always 0 or 1.                              |
|`ime`                            |   1|Unsigned  |Interrupt master enable flag. Always 0 or 1.                      |
|`@`                              |  16|Unsigned  |Address of current instruction. MUST NOT be prefixed with `@`.    |
|`rombank`                        | Any|Unsigned  |Current ROM bank. MUST be 0 for ROMs without banking.             |
|`srambank`                       | Any|Unsigned  |Current SRAM bank. MUST be $FFFFFFFF for programs without SRAM.   |
|`sramenable`                     |   2|Signed    |1 if SRAM is enabled, 0 if disabled, -1 if no SRAM is present.    |
|`target`                         |  16|Unsigned  |Address that caused the action to fire.                           |
|`op`                             |   2|Unsigned  |Current operation (0: read, 1: write, 2: execute, 3: read+write). |
|`value`                          |   8|Context   |Value being read or written, or opcode byte for execution actions.|

The "signedness" column in the table above indicates how the variables' values are extended into 32 bits: they are
zero-extended if unsigned, or sign-extended if signed. If the signedness is "context", it means that they take the
signedness of the expression they appear in.

The "bits" column of the table above indicates the natural width of the value, i.e., the number of bits the value
actually has. The value will be extended to 32 bits when used in an expression, since all values are 32 bits wide in
expressions. The value "any" in that column indicates that the variable can have any natural width — the emulator must
select a reasonable width for the variable. For example, the `rombank` variable will typically be as wide as the
actual ROM bank selection registers in the banking hardware.

The `pc` variable contains the value of the `pc` register, which points to the _next_ instruction (i.e., to the end of
the current instruction); on the other hand, the `@` variable points to the instruction currently being executed.
Therefore, they will always differ by one, two or three. For example, if the currently executing instruction is a
`swap a` instruction (which is two bytes long) at address $1234, then `@` will be equal to $1234 and `pc` will be
equal to $1236.

#### Memory accesses

A memory access indicates that a value MUST be retrieved from memory in order to evaluate the expression. Memory
accesses can retrieve values that are 8, 16 or 32 bits in width; if the value is not 32 bits wide, it MUST be extended
into 32 bits according to the signedness of the expression (zero-extended if unsigned, sign-extended if signed). The
retrieval of a value from memory MUST NOT be considered a memory read; in particular, it MUST NOT trigger any actions
that watch the address in question.

Memory accesses that are not 8 bits wide will read several bytes of memory. These bytes can be collected into a single
value using either little or big endianness; this is indicated for each access.

The syntax for a memory address is any of the following: `[<address>]`, `[<address>!]`, `[<address>!!]`,
`[<address>?]`, `[<address>??]`. These forms respectively indicate 8-bit, 16-bit little-endian, 32-bit little-endian,
16-bit big-endian and 32-bit big-endian access. For example, `[$4000]` is an 8-bit access, `[$5000?]` is a 16-bit
big-endian access, and `[@bc!!]` is a 32-bit little-endian access.

Memory accesses may be banked or unbanked. For banked memory areas, banked accesses use a specific bank, while
unbanked accesses use whichever bank is currently loaded. Accesses to unbanked memory areas or accesses that cross
memory areas (such as a 16-bit access at address $9FFF) MUST be unbanked. A memory area is considered banked if bank
switching is possible in that area, and unbanked otherwise; emulators MAY consider all of ROM, VRAM, SRAM and/or WRAM
to be banked even when no bank switching is possible in some or all of those areas.

The `<address>` part of the memory access can be written in one of three forms:

* `<bank>:<expression>`: indicates a banked access. Both `<bank>` and `<expression>` may be arbitrary expressions.
* `:<expression>`: indicates an unbanked access. `<expression>` may be any arbitrary expression.
* `<expression>` (i.e., just an expression): the access is banked or unbanked depending on the expression. If the
  first token of the expression is a banked symbol, the access is banked and takes the bank of that symbol; otherwise,
  the access is unbanked.

For the forms that contain a colon, the colon MAY be surrounded by arbitrary amounts of whitespace; this whitespace is
meaningless.

If either the bank or the address are too big to fit in their corresponding numbers of bits (e.g., the address doesn't
fit in 16 bits), the upper bits are ignored; overflowing the fields MUST NOT cause an error. A multi-byte unbanked
access MAY begin near the top of the addressing space; the addressing space MUST be considered to wrap around.

There MAY be arbitrary whitespace after the `[` character or before the `]` character; there MAY also be arbitrary
whitespace before the `!`, `!!`, `?` or `??` signs at the end of a multi-byte access. This whitespace is meaningless
and MUST be ignored.

#### Operators

Operators indicate a calculation that is carried out on their operands in order to produce a new result. All operators
take 32-bit values as their operands and generate a 32-bit result. Some operators behave differently depending on
whether they are signed or unsigned; operators always take the signedness of the expression they appear in.

Operators can be unary or binary. Binary operators go between their two operands, while unary operators go before
their single operand. Operators MAY be surrounded by arbitrary amounts of whitespace; this whitespace is meaningless.
Unary operators MUST appear at the beginning of an expression or subexpression; emulators MAY relax this rule, but
they are also allowed to issue an error instead if the rule is broken.

Parentheses can be used to group all or part of an expression into a subexpression; such a subexpression MUST be
evaluated into a single value before being used as an operand to some other operator. In other words, parentheses
indicate that the subexpression takes precedence above the regular precedence rules. Parentheses MAY be surrounded by
arbitrary amounts of whitespace; this whitespace is meaningless.

Unary operators MUST be evaluated before binary operators. If an implementation allows many of them to appear together
(violating the rule about unary operators appearing at the beginning of an expression or subexpression, since only the
first of such operators can be at the beginning), they MUST be evaluated right to left. Valid unary operators are as
follows, all having the same precedence:

|Operator|Description                                                                                                |
|:------:|:----------------------------------------------------------------------------------------------------------|
|`-`     |Two's complement of its operand.                                                                           |
|`+`     |Results in its operand unmodified. Added for completeness.                                                 |
|`~`     |One's complement of its operand, i.e., unary NOT.                                                          |
|`&`     |MUST be applied to a symbol; results in that symbol's bank. If the symbol is unbanked, the result is -1.   |
|`!`     |1 if the operand is zero, or 0 otherwise.                                                                  |
|`!!`    |0 if the operand is zero, or 1 otherwise.                                                                  |

If an implementation allows multiple unary operators to appear together, the `!!` operator MAY be implemented as two
consecutive instances of the `!` operator instead of being a valid operator by itself.

Binary operators MUST be evaluated in descending order of precedence; ties are broken in left-to-right order. Valid
binary operators are as follows:

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

<!-- A note to readers of the raw file: GitHub-flavored Markdown requires that the pipe character (`|`) is escaped
     within tables, even if it appears between backticks. The actual operators for bitwise and logical OR are `|`
     and `||`, without any backslashes.                                                                            -->

If there is any parsing ambiguity between two unary or two binary operators, it MUST be resolved in favour of the
longest one of the two. If there is any parsing ambiguity between a unary and a binary operator, it MUST be resolved
in favour of the binary operator.

### 5.4. The condition field

The condition field of an action indicates when the action will fire. This field is comprised of three subfields: the
address field, the flags field and the condition expression field. The first two are mandatory, but the condition
expression field MAY be missing; if it is, it is assumed to be the constant expression `1`. These subfields are
separated by arbitrary amounts of whitespace.

The address and flags subfields MUST NOT contain any inner whitespace, since whitespace is used to delimit them. The
condition expression subfield MAY contain arbitrary inner whitespace, as it is the last subfield of the condition
field. The condition field is terminated by a colon; while the address and condition expression subfields can contain
colons, the terminating colon is unambiguous because the flags subfield is mandatory and all colons within the
condition expression subfield will necessarily appear within brackets.

The address subfield indicates the addresses that can cause the action to fire; the flags subfield indicates which
operations over those addresses will trigger it, as well as some additional attributes for the action. The condition
expression subfield is an expression that must be non-zero for the action to fire; if the condition expression
evaluates to zero for any specific instance, the action is ignored. For example, the following condition field:

```
$04:$58AB rx @bc > $1234
```

indicates that the action will fire if the byte at bank 4, address $58AB is read or executed, but only if register
`bc` is greater than $1234.

The condition field will be followed by a colon and the command list; the action line MAY be broken after the colon by
inserting a newline. Following the example above, the full action line might look like this:

```
$04:$58AB rx @bc > $1234: break
```

The colon that terminates the condition field (the one after `$1234`) is unambiguous, because the colon in the address
subfield (`$04:$58AB`) comes before the flags subfield, and the flags subfield is mandatory.

Additional whitespace between the subfields MAY be inserted for readability, like so:

```
$04:$58AB       rx      @bc > $1234:            break
```

The condition expression subfield can be any arbitrary expression, as detailed in [section 5.3][section5.3]. The other
two subfields will be explained in the following sections.

### 5.5. The address subfield

The address subfield of the condition field specifies the addresses that will be watched by the action, i.e., which
addresses can cause the action to fire. It can specify either a single address or a range, and it can be banked or
unbanked.

If the address subfield specifies a range, the starting address MUST be lower than or equal to the ending address; no
wrap-around is assumed for this subfield. Also, if the address is banked, the address or range of addresses MUST be in
a banked area of memory and MUST NOT cross memory areas. The criteria for determining whether a memory area is banked
is the same as the one given for [memory access][expressions-memory].

Address ranges always include both endpoints. If an address or range is unbanked, but it covers a banked portion of
memory, the address or range refers to those addresses in all banks.

The address subfield contains one or more constant expressions. Whitespace is usually allowed (but meaningless) within
expressions; however, the address subfield MUST NOT contain any whitespace.

A single address can be specified in any of the ways allowed for [memory accesses][expressions-memory], but all
expressions involved MUST be constant expressions (i.e., expressions that don't contain variables or memory accesses).
The address subfield will be banked or unbanked depending on the format used for the address; this is determined in
the same way as memory accesses. The address is not surrounded by brackets, and it cannot contain any size/endianness
suffixes; only the address part of the memory access syntax is used.

Since addresses are constant expressions, numerical constants are handled in the manner explained in
[the corresponding part of the Expressions section][expressions-constants]. In particular, this means that numerical
constants don't default to hexadecimal (unless there is a `@radix 16` directive in effect), and that they MUST begin
with a digit or a base indicator character.

If either the bank or the address are too big to fit in their corresponding numbers of bits (e.g., the address doesn't
fit in 16 bits), the upper bits are ignored; overflowing the fields MUST NOT cause an error.

Ranges can be specified in one of two ways:

* `<address>--<expression>`: the start and end are given explicitly. The `--` delimiter represents a hyphen
  (indicating a range), not the subtraction operator; for instance, the range `$C000--$DFFF` covers all of WRAM.
* `<address>++<expression>`: the second expression indicates the length of the range. For instance, `$C000++$2000`
  covers all of WRAM as well.

In both cases, `<address>` is a banked or unbanked address as explained above, and `<expression>` is any constant
expression. The result of `<expression>` MUST be truncated to 16 bits before computing the range; in particular, this
means that using a negative value of `<expression>` for the `++` form will not work as expected. The `--` and `++`
symbols are delimiters, not operators; there is no need to parenthesize the two expressions that they delimit, and
they don't have a precedence because they are not involved in any calculation. For the `++` form, after truncating to
16 bits, the length expression MUST NOT evaluate to zero.

Ranges MUST NOT wrap around. This means that, for the `--` form, the ending address MUST be greater than or equal to
the starting address; for the `++` form, after truncating both the address and the length to 16 bits, their sum MUST
NOT exceed $10000. (It is acceptable for the sum to be _exactly_ $10000, as this occurs for ranges that extend to the
very end of the addressing space, such as `$FF80++$80`.)

The address subfield can also be a single `*` character. This is shorthand for the range `$0000--$FFFF`.

### 5.6. Flags

An action's flags indicate which operations will trigger the action, as well as some attributes for it. The address
subfield is used to specify the addresses that trigger the action; the flags refine this by stating which operations
on those addresses will cause it to fire. Actions MUST always fire _before_ the operation is executed.

Flags come in two kinds: _operation flags_ and _attribute flags_. Operation flags are the flags that represent
operations over the memory addresses that are being watched; every action MUST have at least one operation flag.
Attribute flags indicate an action's attributes, and they are OPTIONAL.

Flags consist of one or two characters; multiple flags can be given by concatenating them. Flags MUST NOT be separated
by whitespace. Two-character flags always consist of identical characters, and they are modified versions of the
corresponding one-character versions; the corresponding one-character and two-character flags MUST NOT be given
together. Flags MUST NOT be duplicated; any particular flag may only be given once per action. Flags MAY be given in
any order; the order is immaterial.

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

For actions that fire on writes, if an expression contains a memory access that retrieves the value of the address
being written to, the value _before_ the write MUST be used for that expression. The `ww` flag indicates that the
action fires when the value being written to is different from the value already present at that address; in other
words, the expression `[target] != value` evaluates to 1.

Executing an instruction MUST NOT be considered a memory read for the purpose of firing actions. Actions that fire on
reads are only triggered by explicit memory reads.

If an instruction triggers an action multiple times (for example, by writing two bytes at once, or by reading and
writing to the same address at once), the action MUST fire only once unless the `m` flag is given. With the `m` flag,
the action MUST fire once per operation. For example, given the following actions:

```
$C100--$C101 w: message "action 1"
$C100--$C101 wm: message "action 2"
```

the instruction `ld [$C100], sp` would cause the first action to fire once but the second action to fire twice (once
per byte written). If an action with the flags `xm` was created for the address range containing that instruction,
that action would fire three times, since three bytes are being executed.

If a multi-byte instruction is executed, actions that fire on execution MUST fire if any part of the instruction lies
within the addresses watched by the action. If the action does not include the `m` flag, it fires only once; the
`target` variable MUST be set to point to the first byte of the instruction that lies within the watched addresses. On
the other hand, actions that fire on jump targets (i.e., actions that contain the `xx` flag) MUST only fire if the
jump target matches exactly a watched address, regardless of the size of the instruction at the receiving end of the
jump.

For the purpose of firing actions that trigger on instruction execution, undefined instructions and corrupted stops
MUST be considered to be one and two bytes long respectively. Any actions that fire on execution of such instructions
MUST fire before the emulator handles the error condition that results from attempting to execute them.

For a combined read/write operation (such as `inc [hl]`), if an action fires on both reads and writes and the `m` flag
is given, the read operation MUST fire before the write. If the action does not contain the `m` flag, the combined
operation fires the action once, with the `op` variable taking the value 3 and the `value` variable taking the value
of the written byte.

If an action fires on reads and modifying writes (i.e., it contains the `r` and `ww` flags), the above behavior only
applies for bytes that are modified. Reads followed by non-modifying writes (e.g., the execution of a `set 0, [hl]`
instruction when the value at `[hl]` is already odd) MUST be treated as pure reads for the purpose of firing the
action.

For an instruction that accesses multiple bytes at once, if it causes an action watching several of those addresses
that does not contain the `m` flag to fire, the `target` and `value` variables for it MUST be set for the highest
address in common between the action and the operation. If the action contains the `ww` flag and the operation that
fires the action is a write that only modifies some of the affected addresses, addresses that are not modified by the
write MUST NOT be taken into account when determining the highest address in common.

## 6. Commands

Commands indicate the actual steps taken by the emulator when an action fires. Actions may cause the emulator to halt
execution, print a debug message, and so on. Every action has a command list containing one or more commands; these
commands MUST be executed in order when the action fires. Command lists MAY have any number of commands, but emulators
MAY impose reasonable limits in order to prevent bugs or security issues. Such a limit SHOULD be high enough to not
interfere with the normal, intended usage of debugfiles.

Commands in an action's command list are separated by semicolons. Semicolons are separators, not terminators; this
means that the last action in the command list MUST NOT be followed by a semicolon. (In fact, the line ending without
a colon or semicolon at the end is what marks the end of the action.) These semicolons MAY be surrounded by arbitrary
amounts of whitespace, which is ignored.

An action line MAY be broken after any of these separating semicolons, inserting a newline. This means that the
command list continues in the next line.

Each command has its own syntax and semantics. Commands always begin with a keyword, which identifies the command;
commands can have additional arguments after the keyword, separated by one or more spaces. For example, the `message`
command will be followed by a string, as in `message "Hello world!"`, but the `break` command always stands by itself.

### 6.1. Basic emulator commands

These commands cause the emulator to perform basic debugging functions.

#### `break` command

Syntax: `break`

This command causes a breakpoint, i.e., it instructs the emulator to stop execution and open the debugger.

This command is idempotent: if several of these commands are executed at once (because a command list contains
multiple `break` commands or because several actions containing a `break` command fire at once), they have the same
effect as a single `break` command. This MUST NOT cause an error.

#### `reset` command

Syntax: `reset`

This command instructs the emulator to reset itself, as if the Game Boy was power-cycled.

This command MAY be implemented in an idempotent way: several `reset` commands executed at once MAY reset the emulator
any number of times between one and the number of `reset` commands executed. The exact number of resets executed in
this case is implementation-defined and it MAY vary between executions.

#### `message` command

Syntax: `message <string>`

This command causes a debug message to be printed. The emulator MAY output debug messages in any way it wishes; it
SHOULD NOT stop emulation for that purpose.

The `<string>` argument can be either a named string declared with [the `@str` directive][directive-str] or a quoted
constant string as explained in [section 7][section7]. Expression substitutions apply regardless of the chosen format.

#### `alert` command

Syntax: `alert <string>`

This command is equivalent to the `message` command, but it is intended to bring immediate attention to the message
printed. The emulator SHOULD display the message in a way that brings the user's attention to it, such as a message
box; emulation MUST be stopped until the user acknowledges the message.

If several `alert` commands are executed at once (because a command list contains many of them or because several
actions containing an `alert` command fire at once), the emulator SHOULD combine them into a single alert, in order to
avoid stopping emulation repeatedly.

### 6.2. Action-related commands

These commands enable or disable actions.

#### `enable` command

Syntax: `enable [<group>]`

Enables all actions within a group (as declared by [the `@group` directive][directive-group]). If some or all actions
within that group are already enabled, this command does nothing to those actions; enabling already-enabled actions
MUST NOT cause an error.

The group MAY be omitted from the command; if it is, it enables the action where it appears. (This can be useful if
the action has been disabled by another command from the same command list, or by a command from an action that fired
simultaneously.)

#### `disable` command

Syntax: `disable [<group>]`

Disables all actions within a group; disabling already-disabled actions MUST NOT cause an error. This is the opposite
of the `enable` command.

The group MAY be omitted from the command, in which case it disables the action where it appears.

#### `toggle` command

Syntax: `toggle [<group>]`

Toggles the enabled/disabled status of all actions within a group, enabling the ones that are disabled and disabling
the ones that are enabled.

The group MAY be omitted from the command, in which case it toggles the enabled/disabled status of the action where it
appears.

### 6.3. State modification commands

This command modifies the program state, by altering either a variable or a memory location.

#### `set` command

Syntax: `set <lvalue> := <expression>`

This command modifies a value in the program state. The `<lvalue>` part MUST be a variable name or a memory access
expression; the `<expression>` part can be any arbitrary expression. The `:=` symbol used as a delimiter MAY be
surrounded by arbitrary amounts of whitespace, which is ignored. If a variable name is used for the `<lvalue>` part,
it MAY be prefixed with an `@` character; this prefix MUST be used if the variable name collides with an existing
symbol name.

The behavior of this command depends on the kind of `<lvalue>` used:

* If the `<lvalue>` is a memory access, a write to that address MUST be simulated; if it is a multi-byte access, the
  bytes MUST be written in ascending order of addresses. This may or may not cause the address to be modified; for
  example, an address in WRAM will be modified by the write, while an address in ROM will probably cause some banking
  command to be executed. This write MUST NOT cause any actions to fire.
* If the `<lvalue>` is a user-defined variable, that variable MUST be set to the full value of the expression;
  user-defined variables are always 32 bits wide.
* If the `<lvalue>` is an emulator variable, the semantics of writing to it are defined by the emulator; in
  particular, an emulator variable MAY be treated as read-only, ignoring the writes. For the minimal required
  variables listed in [the "Variables" heading of the Expressions section][expressions-variables], they MUST behave as
  follows:
    * Writes to register variables MUST update the corresponding registers (writes to the `f` register will ignore the
      lower four bits);
    * Writes to variables representing CPU flags (`z`, `cy`, `ime`) MUST set the corresponding CPU register to 0 if
      the expression is zero, or to 1 otherwise;
    * Writes to the `rombank` variable MUST cause a ROM bank switch, unless the program does not use ROM banking;
    * If the program uses SRAM, writes to the `srambank` variable MUST cause a SRAM bank switch;
    * If the program uses SRAM, writes to the `sramenable` variable MUST enable SRAM if the expression is non-zero or
      disable SRAM if it is zero;
    * The remaining required variables (`@`, `target`, `op`, `value`, and the `srambank` and `sramenable` variables
      for programs without SRAM) MUST be read-only.

If the `<lvalue>` is a memory access or an emulator variable that are narrower than 32 bits, the result of evaluating
the expression MUST be truncated to fit into the available bits; overflows MUST NOT cause an error.

Undefined variables (that is, variables that aren't provided by the emulator or already defined by the user in a
[`@var` directive][directive-var]) MUST NOT be defined by a `set` command. In other words, attempting to set an
undefined variable MUST cause an error. Since the names of existing variables are already known as the debugfile is
loaded, this error MUST be detected during initial read-in of the file.

### 6.4. Control flow commands

These commands alter the execution of other commands. They can be used to conditionally control the execution of the
command list in order to provide multiple behaviors for an action.

While these commands can be used to skip some commands in the list, there is no way to move backwards in it. This is
an intentional design choice, which ensures that pending commands can be safely discarded by the emulator once they
have been executed or skipped.

#### `nop` command

Syntax: `nop`

This command does absolutely nothing; it can be used as a placeholder.

#### `done` command

Syntax: `done`

This command terminates the execution of the command list; any further commands within the list MUST NOT be executed
after a `done` command is executed. This MUST NOT affect commands executed by other actions that happen to fire
simultaneously.

#### `skip` command

Syntax: `skip <expression>`

This command causes a certain number of subsequent commands to be skipped. The number of skipped commands is
determined by the given expression, which MUST be a constant expression; this expression MUST NOT evaluate to a
negative value (if it is a signed expression) or to a value greater than the number of remaining commands in the list.

#### `if` command

Syntax: `if [<expression>]`

This command causes the following command to be skipped if the given expression evaluates to zero; it MUST NOT be the
last command in the command list. The given expression can be any arbitrary expression.

If the expression is omitted, the `if` command behaves identically to the previous `if` command; in other words, it
skips the following command if and only if the previous `if` command did as well. This MUST NOT cause the expression
of the previous `if` command to be evaluated again.

If an `if` command without an expression is executed before any other `if` command has been executed in the same
command list, it skips the following command, as if the condition had been false.

#### `else` command

Syntax: `else`

This command causes the following command to be skipped if the last `if` command executed did not skip the command
that followed it; in other words, it inverts the condition for the last `if` command. This MUST NOT cause the last
`if` command's expression to be evaluated again. This command MUST NOT be the last command in the command list.

If an `else` command is executed before any `if` commands are executed in the same command list, it doesn't skip the
following command, as if the last `if` command's condition had been false.

## 7. Strings

Strings are used in several contexts within a debugfile. They appear in several directives ([`@str`][directive-str],
[`@group`][directive-group], [`@include`][directive-include], [`@symfile`][directive-symfile],
[`@warning`][directive-warning] and [`@error`][directive-error]) as well as in some commands
([`message`][command-message] and [`alert`][command-alert]). Strings are also the only part of a debugfile (other than
comments) that can contain non-ASCII characters; the full range of UTF-8 (except C0 control characters) is allowed
within a string.

### 7.1. String formatting

Strings MUST be written within quotes (`"`), and they MUST NOT contain any embedded newlines. Simple strings are
always literal, and they contain no escape sequences; these are the strings used in the various directives that use
them.

On the other hand, the [`message`][command-message] and [`alert`][command-alert] commands allow strings that contain
escape sequences, in order to incorporate the result of evaluating an expression into those strings.
[The `@str` directive][directive-str] doesn't parse the strings passed to it, but it also accepts those escape
sequences, since those strings are evaluated when used in the commands mentioned before. Escape sequences are
described in the following section.

### 7.2. Escape sequences

Escape sequences always occur between percent signs (`%`); these signs MUST NOT be mismatched when using a string that
allows escape sequences. The simplest escape sequence is the empty sequence, `%%`, which is simply replaced by a
single percent sign.

Beyond this trivial example, there are two kinds of valid escape sequences:

* Expression escape sequences. These are of the form `%<expression>%` or `%<expression>:<format>%`.
* Conditional escape sequences. These are of the form `%<expression>?<name>%` or `%<expression>?<name>:<name>%`.

Arbitrary whitespace is allowed (and ignored) between the tokens inside the percent signs. Of course, whitespace
outside the percent signs is taken literally as part of the string.

### 7.3. Expression escape sequences

Expression escape sequences are escape sequences that contain an expression and are replaced by the expression's value
when printed. They take the forms `%<expression>%` or `%<expression>:<format>%`; the expression can be any arbitrary
expression.

The `<format>` part of the escape sequence determines how the value will be formatted when printed. It can contain a
formatting character, a number, or both; if it contains both, the formatting character MUST precede the number. The
numbers used as part of the format specifier are always decimal, regardless of the default base in force; the format
character is _not_ a base prefix in this context. The number MUST NOT contain more than two digits.

Valid formatting characters are as follows:

|Character|Description                                                                                               |
|:-------:|:---------------------------------------------------------------------------------------------------------|
|`#`      |Unsigned decimal number.                                                                                  |
|`$`      |Hexadecimal number. Digits `A` through `F` MUST be printed in uppercase.                                  |
|`%`      |Binary number.                                                                                            |
|`-`      |Signed decimal number, prefixed with a `-` if negative.                                                   |
|`+`      |Signed decimal number prefixed with a `-` or `+` depending on its sign. The `+` sign is used for zero.    |

The number indicates how many digits will be used to print the value. If the number is 0, the minimal number of digits
needed to represent the value MUST be used; the value will never begin with a 0 digit unless it is zero. Otherwise,
the exact number of digits indicated MUST be used to print the value; the value MUST be prefixed with 0 digits if it
is shorter than that length, and if it is longer, only the last digits are used. For signed decimal numbers, the sign
is not taken into account when calculating the length.

If the number is omitted, it defaults to 0. If the format character is omitted, it defaults to the base indicated by
the current base in force (as given by [the `@radix` directive][directive-radix]); if that base is 10, the default is
`#` for unsigned expressions and `-` for signed ones.

The `<format>` part of the escape sequence MUST NOT be empty, but it MAY be left out entirely. If it is, both defaults
(format character and number) are used.

### 7.4. Conditional escape sequences

Conditional escape sequences print one of two strings depending on a condition. They take one of the forms
`%<expression>?<name>%` or `%<expression>?<name>:<name>%`, where the expression is any arbitrary expression and the
names represent named strings declared with [the `@str` directive][directive-str].

The first form is replaced by the named string if the expression evaluates to a non-zero value, or by an empty string
if the expression is zero; the second form is replaced by the first named string if the expression evaluates to a
non-zero value, or by the second named string if the expression is zero.

The names used in the `<name>` part of the escape sequence MUST be names previously declared in a `@str` directive.

## 8. Example file

The following is a full example file, in order to show various parts of the format:

```
@debugfile 0.2

@ifemu fooemu < 2.0.3beta
    @warning "Your emulator is known to have issues with debugfiles; please consider updating."

@always

@str rstmessage "RST $%@target:$2% triggered from %@rombank:$2%:%[sp!]:$4%, resetting..."

$0000         x                                   : message rstmessage; reset
$0038         x                                   : message rstmessage; reset

$0000         rw                                  : message "Null pointer access at %@rombank:$2%:%@:$4%!";
                                                    if @op == 1;
                                                         break

; track iterations of this loop
@var _iter 0
@var _total 0
FuncFoo.loop  x                                   : set _iter := _iter + 1
FuncFoo.loop  x     _iter >= 5000                 :
    set _total := _total + _iter;
    message "Looped %_iter% times (accumulated: %_total%)";
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
$0100         x     sp = $FFFE                     : disable stackcheck
Init.done     x                                    : if sp = wStackTop;
                                                         enable stackcheck;
                                                     else;
                                                         alert "Stack error; sp initialized to $%sp:$4%"
```

## 9. Specification compliance

An implementation can be said to be compliant with this specification if it implements it in full, accepting any valid
debugfile and handling it like the specification describes. This can apply to any kind of tool (for example, it should
be possible to create debugfile editors that generate correct debugfiles based on user selections), but it is mostly
relevant for emulators.

However, at the time of writing, no emulator is known to implement all the debugging capabilities required by this
specification. In order to encourage adoption of the specification and with the purpose of achieving compliance
progressively as more features are used, this specification allows for an emulator to claim "partial compliance",
provided that it follows the criteria set in the following sections.

### 9.1. Partial compliance

Partial compliance is compliance with parts of the specification, applicable to implementations that only implement
some of its features. For instance, some directives or commands may not be recognized, or the implementation may
impose additional restrictions not allowed by the specification, such as requiring that conditional expression
subfields contain only simple expressions.

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

This specification is the result of several partially conflicting goals. The desire is to determine a format that
allows easy recording and sharing of debugging information; however, any features supported by such a format become a
burden for implementations to handle, and thus the desire for additional features must be balanced against the cost of
implementing such feaures.

Several considerations within the format, such as always allowing `@` prefixes for variables, come from the desire to
support automated or tool-assisted generation of debugfiles. Tools that generate debugfiles may not have the full
symbol list available at the time of writing debugging actions, and thus the `@` prefix helps them emit the right
expressions without having to assume that a symbol with some specific name will or will not be defined. The same
applies to the various features that allow overriding defaults, such as base prefixes or the `s` and `ss` flags; this
helps those tools not have to rely on their context.

Some restrictions intend to make implementing the format easier. For instance, unary operators are only allowed at the
beginning of an expression; this ensures that expressions are always alternating sequences of values and operators,
without having to detect consecutive operators. That is why emulators that _can_ handle the additional burden are
allowed to relax this restriction, which helps them deal with accidental breaches of this rule more gracefully.

Finally, it is explicitly not the intention to turn this format into a programming language; features like loops are
intentionally left out. User-defined variables are a concession to the utility of tracking some values while debugging
a program (such as the iteration counter in the example in [section 8][section8]). Moving forward, this specification
is not intended to incorporate additional features that would turn the debugfile format into a full programming
language, such as negative arguments to the `skip` command.

## 11. Version history

Version 0.2 (16 April 2019):

* Section "1. Objective and scope": added a note stating that compilers, not just assemblers can generate debugfiles.
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
