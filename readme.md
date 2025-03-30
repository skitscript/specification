# Skitscript Specification [![License](https://img.shields.io/github/license/skitscript/specification.svg)](https://github.com/skitscript/specification/blob/main/license) [![Renovate enabled](https://img.shields.io/badge/renovate-enabled-brightgreen.svg)](https://renovatebot.com/)

A Skitscript file (typically using the `*.skitscript` file extension) is a file
which reads similarly to a stage play, but describes a simple visual-novel-style
game.

## Encoding

Parsers handle Skitscript files as a stream of bytes which happen to be largely
compatible with UTF-8.

The following byte values are interpreted as follows:

| Name                      | Byte   | Description      |
| ------------------------- | ------ | ---------------- |
| Whitespace                | `0x32` | Space            |
|                           | `0x09` | Tab              |
| Excluded from Identifiers | `0x21` | Exclamation Mark |
|                           | `0x3F` | Question Mark    |
|                           | `0x27` | Single Quote     |
|                           | `0x22` | Double Quote     |
|                           | `0x7B` | Opening Bracket  |
|                           | `0x7D` | Closing Bracket  |
|                           | `0x40` | At Sign          |
|                           | `0x2F` | Forward Slash    |
|                           | `0x23` | Hash/pound       |
|                           | `0x25` | Percent          |
|                           | `0x60` | Backtick         |
|                           | `0x2B` | Plus             |
|                           | `0x3C` | Less Than        |
|                           | `0x3D` | Equal            |
|                           | `0x3E` | Greater Than     |
|                           | `0x7C` | Pipe             |
|                           | `0x24` | Dollar           |
|                           | `0x2E` | Period           |
|                           | `0x2D` | Dash             |
|                           | `0x26` | Ampersand        |
| Letter "A" in Keywords    | `0x41` | Upper Case       |
|                           | `0x61` | Lower Case       |
| Letter "C" in Keywords    | `0x43` | Upper Case       |
|                           | `0x63` | Lower Case       |
| Letter "D" in Keywords    | `0x44` | Upper Case       |
|                           | `0x64` | Lower Case       |
| Letter "E" in Keywords    | `0x45` | Upper Case       |
|                           | `0x65` | Lower Case       |
| Letter "H" in Keywords    | `0x48` | Upper Case       |
|                           | `0x68` | Lower Case       |
| Letter "I" in Keywords    | `0x49` | Upper Case       |
|                           | `0x69` | Lower Case       |
| Letter "L" in Keywords    | `0x4C` | Upper Case       |
|                           | `0x6C` | Lower Case       |
| Letter "N" in Keywords    | `0x4E` | Upper Case       |
|                           | `0x6E` | Lower Case       |
| Letter "O" in Keywords    | `0x4F` | Upper Case       |
|                           | `0x6F` | Lower Case       |
| Letter "R" in Keywords    | `0x52` | Upper Case       |
|                           | `0x72` | Lower Case       |
| Letter "S" in Keywords    | `0x53` | Upper Case       |
|                           | `0x73` | Lower Case       |
| Letter "T" in Keywords    | `0x54` | Upper Case       |
|                           | `0x74` | Lower Case       |
| Letter "W" in Keywords    | `0x57` | Upper Case       |
|                           | `0x77` | Lower Case       |
| Letter "X" in Keywords    | `0x58` | Upper Case       |
|                           | `0x78` | Lower Case       |
| Tilde                     | `0x7E` |                  |
| Opening Parenthesis       | `0x28` |                  |
| Closing Parenthesis       | `0x29` |                  |
| Period                    | `0x2E` |                  |
| Comma                     | `0x2C` |                  |
| Asterisk                  | `0x2A` |                  |
| Backslash                 | `0x5C` |                  |
| Colon                     | `0x3A` |                  |
| Carriage Return           | `0x0D` |                  |
| Line Feed                 | `0x0A` |                  |

## Reusable segments

### Identifier

An identifier is a sequence of characters which identifies an entity.

#### Types

The following types of identifiers can exist:

- Character
- Emote
- Entry animation
- Exit animation
- Label
- Flag
- Location

#### Normalization

Identifiers are to be normalized before use, either within the document (e.g.
when searching for a label referenced by a jump) or outside it (e.g. when
searching the local filesystem for a location image).

##### Processes

###### Casing

The following byte ranges are replaced to perform a conversion to lower case:

| Input Start (inclusive) | Input End (inclusive) | Output Start |
| ----------------------- | --------------------- | ------------ |
| `0x41`                  | `0x5A`                | `0x61`       |

For example:

`AnExampleIdentifier`

Normalizes to:

`anexampleidentifier`

###### Excluded Characters

The following characters are considered "excluded":

- Whitespace
- Backslash
- Backtick
- Period
- Asterisk
- All other "Excluded from Identifiers".

Where they occur at the start and/or end of an identifier, they are to be
removed.

Where they occur within an identifier, they are to be collapsed down to single
dashes.

For example:

`   @*an    example<>   !-   =identifier!!   `

Normalizes to:

`an-example-identifier`

#### Validation

##### Disallowed Words

The following words are not permitted in identifiers as they may otherwise be
ambiguous in some contexts:

- `and`
- `or`
- `when`
- `not`
- `is`
- `are`
- `enters`
- `enter`
- `exits`
- `exit`
- `leads`
- `to`
- `set`
- `clear`
- `jump`

It is an error to include them in an identifier.

Note that they can occur within words; for example, while the following are
valid identifiers:

- `Andy`
- `Command`
- `Underhanded`

The following is not:

`Red and green`

##### Disallowed Characters

The following characters are not permitted in identifiers as they may otherwise
be ambiguous in some contexts:

- Colon
- Comma
- Opening parenthesis
- Closing parenthesis
- Tilde

##### Elimination

It is an error to give an identifier which is completely eliminated by the above
processes, for example:

`  !$<>   `

##### Consistency

A warning is to be produced when two identifiers of the same type normalize to
the same value but are not written as the exact same code point sequence in the
document.

Only one warning is to be produced per unique type/normalized identifier
combination per document, pointing to the first and second inconsistent
occurrences.

### Condition

Some events include an optional condition which must be met for the event to
execute.

#### Types

##### Flag set

The specified flag must be set for the condition to pass.  For example:

`when Example Flag Name`

##### Flag cleared

The specified flag must **not** be set (cleared) for the condition to pass.  For
example:

`when not Example Flag Name`

##### At least one flag set

At least one of the listed flags must be set for the condition to pass.  For
example:

- `when Example Flag A Name or Example Flag B Name`
- `when Example Flag A Name, Example Flag B Name or Example Flag C Name`
- `when Example Flag A Name, Example Flag B Name, Example Flag C Name or Example Flag D Name`

##### At least one flag cleared

At least one of the listed flags must **not** be set (cleared) for the condition
to pass.  For example:

- `when not Example Flag A Name and Example Flag B Name`
- `when not Example Flag A Name, Example Flag B Name and Example Flag C Name`
- `when not Example Flag A Name, Example Flag B Name, Example Flag C Name and Example Flag D Name`

##### Every flag set

Each of the listed flags must be set for the condition to pass.  For example:

- `when Example Flag A Name and Example Flag B Name`
- `when Example Flag A Name, Example Flag B Name and Example Flag C Name`
- `when Example Flag A Name, Example Flag B Name, Example Flag C Name and Example Flag D Name`

##### Every flag cleared

Each of the listed flags must **not** be set (cleared) for the condition to
pass.  For example:

- `when not Example Flag A Name or Example Flag B Name`
- `when not Example Flag A Name, Example Flag B Name or Example Flag C Name`
- `when not Example Flag A Name, Example Flag B Name, Example Flag C Name or Example Flag D Name`

#### Additional validation

##### Flag Uniqueness

A warning is to be raised should any condition list two flag identifiers which
normalize to the same value.

##### Flag never set

A warning is to be raised should any condition list a flag which is never set.

## Statements

Each line (separated by carriage return, line feed or carriage return then line
feed) which contains non-white-space characters is to be parsed as an individual
statement.

### Types

An error is to be raised should a statement not parse as any of the following:

#### Label

This specifies a point to which it is possible to jump.  For example:

`~ Example Label Name ~`

##### Validation

###### Uniqueness

It is an error to include two or more labels within the same document with names
which normalize to the same value.

###### References

A warning is to be raised should a label be defined, but never referenced by a
jump or option.

#### Jump

This specifies that the interpreter is to jump ("go to") the label of the
specified name.  For example:

`Jump to Example Label Name.`

##### Conditional

A condition may also be included, for example:

`Jump to Example Label Name when Example Flag Name.`

The jump will only be executed when the condition passes, which, in this case,
requires that `Example Flag Name` be set.  If it is cleared, the interpreter
will instead skip to the subsequent instruction.

##### Validation

###### Label presence

An error is to be raised should the referenced label not exist within the
document.  It is, however, valid for the label to follow the jump instruction,
for example:

```
Jump to Example Label Name.
(...omitted script...)
~ Example Label Name ~
```

###### Unreachable statements

It is possible to write script immediately following an unconditional jump which
is impossible to reach; for example:

```
Jump to Example Label Name.
Example Character Name is Example Emote Name.
```

Here, it is impossible to reach the emote assignment as the interpreter will
jump away under all circumstances.  This is to raise a warning unless the jump
is immediately followed by a label or the end of the file.

#### Speaker(s)

Specify which character(s) speak the next line:

- `Example Character Name:`
- `Example Character A Name and Example Character B Name:`
- `Example Character A Name, Example Character B Name and Example Character C Name:`
- `Example Character A Name, Example Character B Name, Example Character C Name and Example Character D Name:`

##### Emote

Optionally specify an emote for the character(s) to display while speaking the line:

- `Example Character Name (Example Emote Name):`
- `Example Character A Name and Example Character B Name (Example Emote Name):`
- `Example Character A Name, Example Character B Name and Example Character C Name (Example Emote Name):`
- `Example Character A Name, Example Character B Name, Example Character C Name and Example Character D Name (Example Emote Name):`

The default emote for all characters is `neutral`.

##### Validation

###### Character uniqueness

A warning is to be raised should any speaker statement list two character
identifiers which normalize to the same value.

#### Entry animation

Specify that one or more characters are play an entry animation; for example:

- `Example Character Name enters Example Animation Name.`
- `Example Character A Name and Example Character B Name enter Example Animation Name.`
- `Example Character A Name, Example Character B Name and Example Character C Name enter Example Animation Name.`
- `Example Character A Name, Example Character B Name, Example Character C Name and Example Character D Name enter Example Animation Name.`

All characters referenced within the document exist at all times, but default to
being hidden until an animation plays to bring them into view.

##### Emote

An emote can optionally be specified at the same time as specifying an
animation:

- `Example Character Name enters Example Animation Name, Example Emote Name.`
- `Example Character A Name and Example Character B Name enter Example Animation Name, Example Emote Name.`
- `Example Character A Name, Example Character B Name and Example Character C Name enter Example Animation Name, Example Emote Name.`
- `Example Character A Name, Example Character B Name, Example Character C Name and Example Character D Name enter Example Animation Name, Example Emote Name.`

##### Validation

###### Character uniqueness

A warning is to be raised should any entry animation statement list two
character identifiers which normalize to the same value.

#### Exit animation

Specify that one or more characters are play an exit animation; for example:

- `Example Character Name exits Example Animation Name.`
- `Example Character A Name and Example Character B Name exit Example Animation Name.`
- `Example Character A Name, Example Character B Name and Example Character C Name exit Example Animation Name.`
- `Example Character A Name, Example Character B Name, Example Character C Name and Example Character D Name exit Example Animation Name.`

##### Emote

An emote can optionally be specified at the same time as specifying an
animation:

- `Example Character Name exits Example Animation Name, Example Emote Name.`
- `Example Character A Name and Example Character B Name exit Example Animation Name, Example Emote Name.`
- `Example Character A Name, Example Character B Name and Example Character C Name exit Example Animation Name, Example Emote Name.`
- `Example Character A Name, Example Character B Name, Example Character C Name and Example Character D Name exit Example Animation Name, Example Emote Name.`

##### Validation

###### Character uniqueness

A warning is to be raised should any exit animation statement list two character
identifiers which normalize to the same value.

#### Emote

Specify an emote for one or more character(s):

- `Example Character Name is Example Emote Name.`
- `Example Character A Name and Example Character B Name are Example Emote Name.`
- `Example Character A Name, Example Character B Name and Example Character C Name are Example Emote Name.`
- `Example Character A Name, Example Character B Name, Example Character C Name and Example Character D Name are Example Emote Name.`

##### Validation

###### Character uniqueness

A warning is to be raised should any emote statement list two character
identifiers which normalize to the same value.

#### Set Flag(s)

Specify that one or more flag(s) are to be set; has no effect on flags which are
already set at the time of execution:

- `Set Example Flag Name.`
- `Set Example Flag A Name and Example Flag B Name.`
- `Set Example Flag A Name, Example Flag B Name and Example Flag C Name.`
- `Set Example Flag A Name, Example Flag B Name, Example Flag C Name and Example Flag D Name.`

All flags default to cleared/unset until they are set at least once.

##### Validation

###### Flag uniqueness

A warning is to be raised should any set statement list two flag identifiers
which normalize to the same value.

###### Usage

A warning is to be raised should a flag be set but never referenced by a
condition.

#### Clear Flag(s)

Specify that one or more flag(s) are to be cleared; has no effect on flags which
are already cleared at the time of execution:

- `Clear Example Flag Name.`
- `Clear Example Flag A Name and Example Flag B Name.`
- `Clear Example Flag A Name, Example Flag B Name and Example Flag C Name.`
- `Clear Example Flag A Name, Example Flag B Name, Example Flag C Name and Example Flag D Name.`

##### Validation

###### Flag uniqueness

A warning is to be raised should any clear statement list two flag identifiers
which normalize to the same value.

###### Flag(s) set

A warning is to be raised should a flag be cleared but never be set.

#### Line

Specifies a line of dialog to be shown to the user.  It is spoken by the
subject(s) of the last executed speaker(s) statement.  For example:

`  Example line content.`

Note that all lines are indented by any amount of white space to distinguish
them from other statement types.

##### Formatting

Markdown-like formatting can be given within a line:

```
  Example line content with *italics*, **bold** and `code`.
```

###### Escaping

Any formatting character can be prefixed with a backslash `\\` to write that
formatting character directly instead.  Backslashes themselves can also be
escaped by doubling them up `\\\\`; for example:

`  Example line content \*with\* \*\*escaped\*\* \``formatting\`` \\characters\\.`

##### Validation

###### Invalid formatting

An error is to be raised should any formatting blocks either not be closed, or
be closed in an incorrect order.

###### Speaker

An error is to be raised at runtime should a line be reached without defining
speaker(s) at least once.

#### Menu Option

Specifies an option to be displayed in a menu next to the text of the preceding
line.  Normally, multiple of these would be given; for example:

```
- Example Option A leads to Example Label A Name.
- Example Option B leads to Example Label B Name.
- Example Option C leads to Example Label C Name.
```

The line cannot be dismissed other than selecting a menu option.

##### Formatting

Markdown-like formatting can be given for menu options' text; for example:

```
- Example Option *With Italics* A leads to Example Label A Name.
- Example Option **With Bold** B leads to Example Label B Name.
- Example Option `With Code` C leads to Example Label C Name.
```

###### Escaping

Any formatting character can be prefixed with a backslash `\\` to write that
formatting character directly instead.  Backslashes themselves can also be
escaped by doubling them up `\\\\`; for example:

`- Example Option \*With\* \*\*Escaped\*\* \``Formatting\`` \\Characters\\ D leads to Example Label D Name.`

##### Conditional

Each menu option may optionally have a condition which must be met for it to be
visible and selectable; for example:

```
- Example Option A leads to Example Label A Name.
- Example Option B leads to Example Label B Name when Example Flag Name.
- Example Option C leads to Example Label C Name.
```

Should `Example Flag Name` be cleared here, this menu is equivalent to the
following from the user's perspective.

```
- Example Option A leads to Example Label A Name.
- Example Option C leads to Example Label C Name.
```

###### Fall-through

Should all options be eliminated by conditions, no menu is shown and the line
can be dismissed as though no menu options were defined.

##### Validation

###### Label presence

An error is to be raised should the referenced label not exist within the
document.  It is, however, valid for the label to follow the menu option,
for example:

```
- Example Option leads to Example Label Name.
(...omitted script...)
~ Example Label Name ~
```

###### Invalid formatting

An error is to be raised should any formatting blocks either not be closed, or
be closed in an incorrect order.

###### Missing preceding line

An error is to be raised at runtime should a menu option be reached without the
preceding statement being another menu option or a line.

###### Unreachable statements

It is possible to write script immediately following a block of menu options
containing at least one which is unconditional, therefore making that script
impossible to reach; for example:

```
- Example Option leads to Example Label.
Example Character Name is Example Emote Name.
```

Here, it is impossible to reach the emote assignment as the interpreter will
jump away under all circumstances.  This is to raise a warning unless the block
is immediately followed by a label or the end of the file.

#### Location

Specifies the location that the dialog and menu options are presented within;
for example:

`Location: Example Location Name.`

This usually, in practice, specifies a background image.  This should be blank
white until a location is specified.

### End-of-file handling

Script interpreters are to return to the start of the file on reaching the end.
No state such as the active speaker(s), flag(s), etc. is to be reset when this
occurs.

### Infinite loop detection

Interpreters must include infinite loop detection.  They must keep a log of
which lines have been visited since the last time either user input was
requested or the start of the file, and the states of all flags at those times.

Should a line be visited more than one time with the exact same set of flags
set/cleared, this indicates that the script has entered an infinite loop which
will never interact with the user again, and execution is to stop with a runtime
error raised.
