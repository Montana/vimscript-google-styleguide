# Google's Vimscript Style Guide Snippet

## Maktaba

Maktaba is a vimscript plugin library. It is designed for plugin authors. Features include:

* Plugin objects (for manipulating plugins in vimscript)
* Plugin flags (used to configure plugins without global settings)
* Universal logger interface
* Dependency management tools
* Real closures

Maktaba advocates a plugin structure that, when adhered to, gives the plugin access to many powerful tools such as configuration flags. Within Google, these conventions standardize behavior across a wide variety of plugins.

Also contained are many utility functions that ease the pain of working with vimscript. This includes, among other things:

* Exception handling
* Variable type enforcement
* Filepath manipulation

Maktaba relies heavily on dependency management, so I'd recommend use something like VAM.

Installation of a few plugins using VAM looks something like:

```vim
set runtimepath+=~/.vim/bundle/vim-addon-manager/
" Loads glaive, vtd, and their maktaba dependency.
call vam#ActivateAddons(['glaive', 'vtd'])
" Initializes all maktaba plugins.
call maktaba#plugin#Detect()
```

We can generate help tags for a file by by installing maktaba and `fakeplugins/myplugin`:

  ```vim
  :set nocompatible
  :let g:maktabadir = fnamemodify($VROOMFILE, ':p:h:h')
  :let g:bootstrapfile = g:maktabadir . '/bootstrap.vim'
  :execute 'source' g:bootstrapfile

We need write access to the plugin in order to generate help files, so let's
copy it to a tmp directory:

  :let g:thisdir = fnamemodify($VROOMFILE, ':p:h')
  :let g:srcpath = maktaba#path#Join([g:thisdir, 'fakeplugins', 'myplugin'])
  :let g:tmpdir = fnamemodify(tempname(), ':h')
  :call maktaba#system#Call(['cp', '-R', g:srcpath, g:tmpdir])
  :let g:pluginpath = maktaba#path#Join([g:tmpdir, 'myplugin'])
  :let g:myplugin = maktaba#plugin#Install(g:pluginpath)

Observe: There is no bunny in the hat:

  :let g:tagfile = maktaba#path#Join([g:pluginpath, 'doc', 'tags'])
  :call maktaba#ensure#IsFalse(maktaba#path#Exists(g:tagfile))

Maktaba makes them for us:

  :call g:myplugin.GenerateHelpTags()

And now the helpfiles exist!

  :call maktaba#ensure#IsTrue(maktaba#path#Exists(g:tagfile))

Not only that, but the helpfiles are useful.

  :help myhelp
                                                                       *myhelp*
  These are my help files.

Well... somewhat.

Now let's reset the test for next time:

  :call maktaba#system#Call(['rm', '-R', g:pluginpath])
```

Several plugins are [already using maktaba](https://github.com/google/maktaba/wiki/Plugins-Using-Maktaba). As a user, you can generally expect these plugins to be configurable using [Glaive](https://github.com/google/glaive) and be more well-behaved in terms of things like defining unwanted global mappings and variables and avoiding annoying side-effects like moving your cursor.



## Whitespace

Similar to Python: 

* Use two spaces for indents
* Do not use tabs
* Use spaces around operators

This does not apply to arguments to commands.

```vim
let s:variable = "concatenated " . "strings"
command -range=% MyCommand

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


## Prefix all variables with their scope

`g:`, `s:`, and `a:` must always be used.

`b:` changes the variable semantics; use it when you want buffer-local semantics.

`l:` and `v:` should be used for consistency, future proofing, and to avoid subtle bugs. They are not strictly required. Add them in new code but don’t go out of your way to add them elsewhere.

## Regex 

Prefix all regexes with `\m\C`.

In addition to the case sensitivity settings, regex behavior depends upon the user's nomagic setting. To make regexes act like nomagic and noignorecase are set, prepend all regexes with `\m\C`.

You are welcome to use other magic levels `(\v)` and case sensitivities `(\c)` so long as they are intentional and explicit.





