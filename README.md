# Google's Vimscript Style Guide Snippet


## Whitespace

Similar to Python: 

* Use two spaces for indents
* Do not use tabs
* Use spaces around operators

This does not apply to arguments to commands.

```vim
let s:variable = "concatenated " . "strings"
command -range=% MyCommand
```
* Do not introduce trailing whitespace

You need not go out of your way to remove it. Trailing whitespace is allowed in mappings which prep commands for user input, such as ```noremap <leader>gf :grep -f ```.

* Restrict lines to 80 columns wide
* Indent continued lines by four spaces
* Do not align arguments of commands
  
Without Google styleguide:

```vim
command -bang MyCommand  call myplugin#foo()
command       MyCommand2 call myplugin#bar()
```
With Google styleguide: 

```vim
command -bang MyCommand call myplugin#foo()
command MyCommand2 call myplugin#bar()
```

## Dangerous commands 

Avoid commands with unintended side effects.

Avoid using `:s[ubstitute]` as it moves the cursor and prints error messages. Prefer functions (such as `search())` better suited to scripts.

For many vim commands, functions exist that do the same thing with fewer side effects.

## Type checking 

Use strict and explicit checks where possible.

Vimscript has unsafe, unintuitive behavior when dealing with some types. For instance, `0 == 'foo'` evaluates to true.

Use strict comparison operators where possible. When comparing against a string literal, use the `is#` operator. Otherwise, prefer `maktaba#value#IsEqual` or check `type()` explicitly.

Check variable types explicitly before using them. Use functions from `maktaba#ensure`, or check `maktaba#value` or `type()` and throw your own errors.

Use `:unlet` for variables that may change types, particularly those assigned inside loops.

## Functions 

In the autoload/ directory, defined with `[!]` and `[abort]`.

Autoloading allows functions to be loaded on demand, which makes startuptime faster and enforces function namespacing.

`Script-local` functions are welcome, but should also live in autoload/ and be called by autoloaded functions.

Non-library plugins should expose commands instead of functions. Command logic should be extracted into functions and autoloaded.

`[!]` allows developers to reload their functions without complaint.

`[abort]` forces the function to halt when it encounters an error.



