# keycut-data
Keyboard shortcuts data stored in YAML files.

* [Motivation](#motivation)
* [Contributing](#contributing-or-using-data-to-build-a-keycut-tool)
  * [Shortcut structure](#structure-of-a-shortcut)
  * [Directory structure](#directory-structure-for-keycut-tools)
  * [File names](#file-names)
  * [File contents](#file-contents)
  * [Syntax of sequences](#syntax-of-sequences)
  * [Case sensitivity](#case-sensitivity)
  * [Finer control](#finer-control-over-sequences-attributes)
* [Todo-list](#todo)
* [License](#license)


## Motivation
You can find several databases on the web that aim at storing shortcuts
for a big variety of applications.

**So why store them again as YAML in a git repository?**

Well, **first**, because it is simple to manipulate and simple to maintain.

**Second**, because each of these databases lacks this or that application,
because the way they provide shortcuts is not standardize, and because they
do not have APIs to let us build tools on it (or do they? If you do know
such a website or database, please let me know, post an issue on this repo!).

And **third**, for me. As a user of many command-line utils, graphical
interfaces and websites (like most of you), I want a way to easily search
through all the available shortcuts to improve my use of these apps.

I often have been scared about starting to use such or such app because
I didn't want to spend time reading and writing and learning (and binding?)
AND FAILING TO REMEMBER and going back in the man page or on a web cheat-sheet
to find again all the numerous shortcuts and hotkeys that were available or
configurable.

## Contributing or using data to build a keycut tool
### Structure of a shortcut
A shortcut is described by an action. This action is a string without newlines.  
A shortcut is obviously composed of sequences of keys, written as an array
(or list). Each sequence is equivalent to an other in terms of behavior.  
A shortcut has three attributes:
* `binding`, which tells if its related action is "bindable"
  (i.e configurable to work with another sequence of keys).
* `case_sensitive`, which tells if keys sequences are case sensitive or not.
* `overwrite`, which, in case of inheritance, overwrites the inherited
  shortcut sequences and attributes if set to true, and concatenates new
  sequences to previous ones if false.

#### Deprecated attributes
- [hotkey](https://en.wikipedia.org/wiki/Keyboard_shortcut):
  no real interest.

- [sacred](https://en.wikipedia.org/wiki/Keyboard_shortcut#.22Sacred.22_keybindings):
  no real interest, opinionated value, never 100% sacred.

### Directory structure for keycut tools
The following is just a proposition.
You can of course organize directories as you want.

The `default` directory contains all community-or-authors-validated shortcuts.
Every other directories are "profile" directories. They contain user-defined
shortcuts that inherit (or not) from the default shortcuts, overwriting some
of them (or none).

### File names
`<APP>[<VERSION>].yml` where `<APP>` and `<VERSION>` can contain any character.
Avoid spaces and other annoying characters.

### File contents
First of all, you should know that *mapping keys* order is not preserved
when loading a YAML file. *Sequences* keep order.

To improve readability, put your inheritance instructions on top of the file,
if any.
 
```yaml
# Explicit inheritance
inherits: app.version
# Implicit inheritance from previous version using semantic versioning
inherits: true
# Default
inherits: false
```

Then, add your global variables, if any.  
The following values are the defaults.

```yaml
binding: false
overwrite: false
case_sensitive: auto
# These global variables are handy if you don't want to add
# these attributes in each of the following shortcuts
```

Then, add your shortcuts.

```yaml
# If you don't need categories, just use ONE category named 'shortcuts'
Some category:
- action: some action related to this category
  keys: [<SEQUENCE_1>, ..., <SEQUENCE_K>]
# The next section details syntax of SEQUENCES
- action: >
    This action is long enough to take several lines in the YAML file.
    I think we should have a line size limit of 79 characters, what do you 
    think ?
  keys: [Ctrl+E]
- action: you may want to use the short syntax if you have many short actions
  keys: [y]
- {action: Go left, keys: [←]}
- {action: Go right, keys: [→]}
- {action: Go up, keys: [↑]}
- {action: Go down, keys: [↓]}

Here is another category:
- action: I am a binding, even if 'binding' is globally set to false :D
  keys: [C]
  binding: true
```

### Syntax of sequences
* To disambiguate, each control key should use CapFirst format, and each
  letter in a sequence part **with control keys** should be lower:  
  `Ctrl-a`, `Ctrl-Shift-b` or `Ctrl-Alt-Del`, but not
  `ctrl-a`, `ctrl-shift-B` or `CTRL-ALT-DEL`.  

* When keys have to be entered ALL TOGETHER, use the `-` symbol:  
  `Shift-kct` means Shift+k then c then t, **while keeping Shift down**.  
  It is equivalent to writing `Shift-k Shift-c Shift-t`, but more concise
  (in this case you could even write `KCT` but that's not the point).  
  If Shift should have been released before typing c then t, then you would
  have written `Shift-k ct` or `Shift-k c t` (only the first space is important).  
  Sometimes, sequences with or without release of the control key are equals,
  like `Ctrl-w Ctrl-w` in vim. In the vim shortcut you should then write the two:
  `keys: [Ctrl-ww, Ctrl-w w]`.
  > Remember that control keys like Ctrl, Alt, Shift and others have to
    be pressed BEFORE any other key in order for the sequence to be recognized.
  
* For sequences using control keys and a hyphen `-`,
  you might use the terms `Minus`, `Hyphen` or `Dash`, because `Ctrl--`
  is not very easy to read, even if it is tolerated. If the sequence uses
  a hyphen but no control keys, then just use `-`.

### Case sensitivity
Typically, if a sequence (an or just a part of it) requires usage of a control
key (Ctrl, Alt, Shift, ...), then it is assumed to be case-insensitive.  
Thus, each sequence part **without any control key** is assumed to be
case-sensitive: `g` is not equal to `G`, which is equal to `Shift-g`.
You can override this behavior using the `case_sensitive` attribute:
* `true`: case sensitive for all
* `false`: case insensitive for all
* `auto`: case insensitive for sequences with control keys,
  case sensitive otherwise.

### Finer control over sequences attributes
Sometimes you can have a key sequence that does not make use of control keys
and is case-insensitive. It is also possible than some of equivalent sequences
of a shortcut are bindings whereas others are not (hardcoded default).  
You can use a `binding` and `case_sensitive` attribute for each item in
a list of sequences, like this:

```yaml
binding: false
case_sensitive: true

shortcuts:
- action: finer control
  keys:
    - seq: a
      binding: true
      case_sensitive: false
# or even like this
- action: global < shortcut < sequence
  case_sensitive: false
  binding: true
  keys:
    - seq: t
      case_sensitive: true
    - seq: a
      
```

## Todo
- [ ] Find a standard to notate CTRL, ALT, SHIFT, DEL, ESC, ... keys
- [ ] Find a standard for sequences that take parameters
- [ ] Use Unicode arrows and others characters ('apple' key), or literals?
- [x] Case-sensitive attribute
- [ ] Note attribute for details about shortcut use?
- [x] Overwrite attribute for inheritant files.
      False = concatenation of sequences.
- [ ] System for i18n and l10n: copy all strings (actions)
      into translation files?
- [ ] Expand the concept of inheritance to profiles, with
      this configuration written in a .keycutrc file?
- [ ] Do shortcuts requiring for several non-control keys to be entered
      together exist? Something like `k+e`? Never saw one like that.

## License
Copyright (c) 2015 Timothée Mazzucotelli

This Source Code is subject to the terms of the Mozilla Public License, v. 2.0. See the LICENSE file for more details.
