# Debugfile format specification - version history

This document is a companion for the [debugfile format specification](debugfile.md), listing all changes since its
first published version.

#### Copyright

This document, being a companion to the main specification, is also released to the public domain under the
[Creative Commons Zero][cc0] dedication.
No copyright is claimed on this document; attribution is appreciated.

[cc0]: https://creativecommons.org/publicdomain/zero/1.0/legalcode

* * *

### Version 1 (27 April 2023)

* Section "3.3. Error handling": added an explicit allowance for implementations to issue warnings.
* Section "3.5. Conditional inclusion":
    * Mentioned `@ifemu` and `@ifnotemu` explicitly in the requirements about emulator names and version numbers.
    * Reworded the valid character restriction for emulator names and version numbers from a MUST to a MUST NOT.
* Section "4.1. Identification":
    * Indicated that relations between version numbers (compatibility and recency) are transitive.
    * Reworded the requirement for the first line to contain a `@debugfile` directive to avoid implying that it cannot
      be used elsewhere.
    * Clarified that included files need not contain any `@debugfile` directives.
* Section "4.2. Conditional inclusion": clarified the meaning of "invalid version numbers".
* Section "4.3. Declarations":
    * Reworded the valid character restriction for identifiers from a MUST to a MUST NOT.
    * Required double-underscore symbols referenced by `@alias` to be real symbols, not implementation-defined ones.
    * Explicitly noted that local symbols cannot be aliased by `@alias`.
    * Restricted the prohibition on overriding symbols aliased by `@alias` to further `@sym` declarations.
    * Added a remark stating that variables declared by `@var` are 32 bits wide.
    * Marked `@str` string names beginning with two underscores as reserved for the implementation, matching the rule
      for symbol names.
* Section "4.5. Included files": noted that relative paths are relative to the including file.
* Section "5.3. Expressions":
    * Referenced the new Annex B in the introduction.
    * Made division by zero requirements explicit.
    * Noted that division is affected by signedness.
    * Required the `&` unary operator to return the effective bank number when it differs from the one selected.
    * Strengthened the option to accept banked address expressions with a bank of zero for unbanked regions from a MAY
      to a SHOULD.
    * Renamed the `@` variable to `pc` and the (old) `pc` variable to `next`.
    * Required the (new) `pc` variable to point to the current instruction, even if it wouldn't in hardware.
    * Rewrote the "Memory accesses" section and added the `^` suffix.
    * Required memory accesses to not cause side effects.
    * Required constant expressions to not contain unary `&` operators.
    * Required address expressions not to truncate their bank numbers when defining a symbol via `@sym` or `@local`.
    * Relaxed the requirement for address expressions to be unbanked when referring to unbanked regions to only apply
      when not declaring a symbol.
* Section "5.6. Flags":
    * Required flags to be case-insensitive.
    * Required unknown flags to be considered a syntax error.
    * Specified that the `op` variable is set to 2 when an action with the `xx` flag is executed.
    * Updated the description of the `xx` flag to refer to the `pc` variable instead of `@`.
* Section "6.3. State modification commands":
    * Defined the behavior of `set` on memory accesses with the `^` suffix.
    * Noted that writing to the `pc` variable with `set` causes a jump.
    * Noted that the `next` variable is read-only.
    * Required `set` to treat the `sram` variable as read-only if SRAM cannot be disabled in the current hardware
      configuration.
    * Indicated that `jump` doesn't cause actions with the `x` flag (which would execute later as those instructions
      are executed) not to execute.
* Section "7. Strings": noted that tabs are the only C0 control character allowed within a string (although they will
  be replaced by spaces after normalization).
* Section "7.1. String formatting": noted that strings cannot contain literal quote characters.
* Section "7.2. Escape sequences": allowed emulators to treat invalid escape sequences in strings used with `message`,
  `alert` and `@str` as syntax errors.
* Section "7.4. Selection escape sequences": indicated that the rules for default base and signedness that apply for
  expression escape sequences also apply for selection escape sequences.
* Section "8. Specification compliance": renumbered (formerly section 9).
* Section "9. Closing remarks": renumbered (formerly section 10).
* Annex A — "Example file":
    * Moved into an annex (formerly section "8. Example file").
    * Marked the annex as not normative.
    * Updated the version number in the example.
    * Replaced references to the removed variable `@` with `pc`.
    * Added some extra checks to show current instruction detection.
    * Shortened and restructured some lines for better rendering of the example on narrower screens.
* Annex B — "Expression evaluation examples": created.
* Version history (formerly section "11. Version history"): moved to a separate document.
* Changed the terms of the public domain dedication from the Unlicense to Creative Commons Zero, and added an explicit
  section at the beginning of the document to reflect this.
* Removed the table of contents at the beginning of the document.
* Minor editorial changes.

### Version 0.7 (11 February 2023)

* Section "1. Objective and scope": defined "user" as the originator of the debugfile.
* Section "3.1. Overall formatting":
    * Explained what a byte order mark is and enforced a MUST NOT requirement.
    * Strengthened the requirement to not use C0 control characters to a capitalized MUST.
* Section "3.3. Error handling": required implementations to treat missing referenced files as an error.
* Section "3.4. Basic structure": required directive names to be case-insensitive.
* Section "4. Directives": required directive names to be case-insensitive.
* Section "4.3. Declarations":
    * Marked symbols beginning with two underscores as reserved for the implementation.
    * Added `@local` directive.
    * Updated `@alias` directive to declare local symbols.
* Section "4.5. Included files": required implementations to treat unacceptable file paths as non-existent files.
* Section "4.6. Configuration":
    * Noted that `@signedness` applies to directives, not just actions, and that it affects more than just divisions
      and shifts.
    * Required `@signedness` to handle its settings in a case-insensitive manner.
* Section "5.3. Expressions": noted that directives can also use expressions.
* Section "5.1. Basic action structure": required actions to return to their initial state (enabled or disabled) when
  the emulator is reset.
* Section "5.3. Expressions":
    * Renamed the `z` and `cy` variables to `zf` and `cf`, and added `nf` and `hf`.
    * Noted that symbols defined by `@local` are also symbols.
    * Indicated that the current signedness in force also affects the signedness of expressions.
    * Required the `&` and `&&` unary operators to zero-extend.
    * Strengthened the requirement that defines constant expressions to a MUST.
* Section "6. Commands": required command names to be case-insensitive.
* Section "6.1. Basic emulator commands": required all escape sequences to apply for `message` and `alert`, not just
  expression escape sequences.
* Section "6.2. Action-related commands": required `enable`, `disable` and `toggle` to only take effect starting at
  the next event.
* Section "6.3. State modification commands": updated the `set` command's effect on variables to refer to the new
  variables for CPU flags.
* Section "7.3. Expression escape sequences": required expression escape sequences to be evaluated in the context of
  their containing action.
* Section "7.4. Selection escape sequences": explained unconditional named string replacements.
* Section "8. Example file": updated the version number in the example.
* Section "9. Specification compliance": weakened the statement about no emulator implementing all required features
  to "most emulators".
* Minor rewording and other editorial changes.

### Version 0.6 (3 February 2023)

* Section "3.2. Normalization and whitespace": noted that the number of spaces that will replace a tab is only
  relevant inside quoted strings.
* Section "3.5. Conditional inclusion":
    * Required implementations to ignore syntax errors other than invalid directives in conditionally excluded
      portions of the file.
    * Capitalized the MUST requirement for emulators to define a name for themselves.
* Section "4.1. Identification":
    * Noted that a zero by itself is a valid version number component.
    * Noted that empty and comment lines don't count towards the first line requirement.
* Section "4.3. Declarations": made explicit that symbols referenced by `@alias` can use all of UTF-8.
* Section "4.5. Included files": replaced inline symfile definition (in the description of the `@symfile` directive)
  with a reference to the RGBDS specification.
* Section "4.7. Miscellaneous": required `@error` to cause processing to be immediately aborted.
* Section "5.3. Expressions":
    * Fixed value of sample past-the-end symbol.
    * Noted that past-the-end symbols are valid due to their potential use in calculations.
    * Added new "bank at address" (`&`) operator.
    * Updated "bank of symbol" operator to use `&&` instead of `&`.
    * Removed the `bank`, `rombank` and `srambank` variables from list of mandatory variables.
    * Renamed `sramenable` variable to `sram` and updated its effective size to 32 bits.
    * Updated the effective size of the `z`, `cy` and `ime` variables to 32 bits.
    * Removed information about variables with a width of "any" (as none remain).
* Section "5.6. Flags": required emulators not to fire actions on operations not caused by execution of instructions.
* Section "6. Commands": added definitions about private-use commands.
* Section "6.3. State modification commands:
    * Added modification of a bank number (via the unary `&` operator) for the `set` command.
    * Rewrote truncation requirements for expressions in the `set` command to account for all cases more clearly.
* Section "7. Strings": added `@alias` to the list of directives using strings.
* Section "8. Example file":
    * Updated the version number in the example.
    * Replaced old bank-related variables in the example with the new `&` operator.
* Reworded some things for clarification.

### Version 0.5 (26 January 2023)

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

### Version 0.4.1 (4 October 2020)

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

### Version 0.4 (2 October 2020)

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

### Version 0.3 (20 September 2019)

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

### Version 0.2 (16 April 2019)

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

### Version 0.1 (15 April 2019)

* Initial draft version.
