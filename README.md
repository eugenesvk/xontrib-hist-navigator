# xontrib-hist-navigator

Inline (within the same prompt line) dir history navigation with keyboard shortcuts for [xonsh](https://github.com/xonsh/xonsh/)

This fork allows for much more ergonomic configuration of keyboard shortcuts, e.g., `a-x` or `⎇x` insead of `['escape','x']`

# Install

```xsh
xpip install -U git+https://github.com/eugenesvk/xontrib-hist-navigator
```

# Configure

1. Add the following to your `.py` xontrib loading config and `import` it in your xonsh run control file (`~/.xonshrc` or `~/.config/rc.xsh`):
```py
from xonsh.xontribs 	import xontribs_load
from xonsh.built_ins	import XSH
envx = XSH.env

xontribs = [ "hist_navigator", # Initializes hist_navigator (fish-shell-like dir history navigation)
 # your other xontribs
]
# ↓ optional configuration variables (use `False` to disable a keybind)
if 'hist_navigator' in xontribs: # configure xontrib only if it's loaded
  # config var                 value  |default|alt_cmd¦ comment
  envx["XSH_HISTNAV_KEY_PREV"] = "⎇←"  #|['escape','left' ]|False¦ Move to the previous working directory
  envx["XSH_HISTNAV_KEY_NEXT"] = "⎇→"  #|['escape','right']|False¦ Move to the next working directory in the history (if 'prevd' was used)
  envx["XSH_HISTNAV_KEY_UP"]   = "⎇↑"  #|['escape','up'   ]|False¦ Move to the parent directory
  # run to see the allowed list for ↑: from prompt_toolkit.keys import ALL_KEYS; print(ALL_KEYS)
  # Alt is also supported as either of: a- alt- ⎇ ⌥ (converted to a prefix 'escape')
  # Control symbols are also supported as either of: ⎈ ⌃
  # Arrow key symbols are also supported as either of: ▼▲◀▶ ↓↑←→
  envx["XSH_HISTNAV_EMPTY_PROMPT"] = False #|True|False¦ Keybinds only work in an empty prompt

xontribs_load(xontribs) # actually load all xontribs in the list
```

2. Or just add this to your xonsh run control file
```xsh
xontrib load hist_navigator
# configure like in the example above, but replace envx['VAR'] with $VAR
$XSH_HISTNAV_KEY_PREV	= "⎇←" # ...
```

# Use

It keeps track of `cd` usage per session and adds 3 new commands while allowing keyboard shortcuts to update existing prompt instead of creating a new line

| command	| description                                                            	| default shortcut        	|
| -------	| -----------------------------------------------------------------------	| --------                	|
| `prevd`	| move to previous working directory in history                          	| <kbd>⎇</kbd><kbd>◀</kbd>	|
| `nextd`	| move to next working directory in history<br/>(if `prevd` is used)     	| <kbd>⎇</kbd><kbd>▶</kbd>	|
| `listd`	| list cd history                                                        	|                         	|
|        	| move to parent directory                                               	| <kbd>⎇</kbd><kbd>▲</kbd>	|


# Traversal behavior

By default, all cd history is kept. Let's look at an example. Let's open
a new shell:

```sh
❯
```

Now when we change the directory, it's added to the history along with
the previous directory where we were (the user's home directory):

```sh
❯ cd BASE
❯ listd
['~', '~/BASE']
```

Let's descend further down:

```sh
❯ cd sub
❯ listd
['~', '~/BASE', '~/BASE/sub']
```

So far, so obvious. Now, when we use `prevd`, you'll see that the history
stays intact:

```sh
❯ prevd
❯ pwd
~/BASE
❯ listd
['~', '~/BASE', '~/BASE/sub']
```

This allows you to use `nextd` to go back and forth:

```sh
❯ nextd
❯ pwd
~/BASE/sub
❯ listd
['~', '~/BASE', '~/BASE/sub']
❯ prevd
❯ pwd
~/BASE
❯ listd
['~', '~/BASE', '~/BASE/sub']
```

Now, if you change the directory entirely after a `prevd`, the entire
previous history is still kept:

```sh
❯ pwd
~/BASE
❯ listd
['~', '~/BASE', '~/BASE/sub']
❯ cd /tmp
❯ listd
['~', '~/BASE', '~/BASE/sub', '/tmp']
```

This means that issuing `prevd` now will actually return to `~/BASE/sub`
and not to `~/BASE` which was the last seen previous directory.
This is by design to allow you to quickly traverse previously visited
directories using keyboard shortcuts.

If you would rather have the history truncated, so that `prevd` always
takes you to the directory you were in just before, set the following
environment variable in your xonshrc:

```xsh
$XONTRIB_HIST_NAVIGATOR_TRUNCATE="true"
```

In this case the last example would behave differently:

```sh
❯ pwd
~/BASE
❯ listd
['~', '~/BASE', '~/BASE/sub']
❯ cd /tmp
❯ listd
['~', '~/BASE', '/tmp']
```

As you can see `~/BASE/sub` was dropped from history because it wasn't
the last visited directory at the time of changing the directory to
`/tmp`.

# Release

```sh
semantic-release version && semantic-release publish
```

