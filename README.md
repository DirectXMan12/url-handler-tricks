# URL Handler Shortcuts

a nice pattern for when i'm working with a service or subject a lot is to create custom short links for it that i can open from my terminal.  for example, i've got `github:owner/repo#ISSUE-OR-PULL` (and the non-issue variant), or `rfc:RFC#SECTION` and the second number.

## How this works

if you register a custom [xdg mime handler](https://specifications.freedesktop.org/mime-apps-spec/latest/) for the scheme, and then point that at a custom script, it'll enable `xdg-open` to work with those links.

then, you can call `xdg-open` by hand, or, if your terminal/editor supports clikable links (and/or the link escape codes), you'll be able to launch those links quickly while working.

for instance, i'll leave comments like `// addresses bug XYZ, see also github:directxman12/sso-polycule-tech#8675` in source code, or `// per rfc:2743#1.1, we *must* blab blah blah`.

# Examples & Template

See the [examples](examples) directory for some examples.

i use [nushell](http://nushell.sh) to write the opener scripts, since it has built-in URL parsing libraries.

generally, this looks like

```nushell
def main [raw: string] {
    let parsed = $raw |
        url parse |
        select path fragment |
        rename thing1 thing2 |
        do-other-processing

    if parsed.thing2 == null {
        xdg-open $"https://the.actual.site/something/($parsed.thing1)"
    } else {
        xdg-open $"https://the.actual.site/something/($parsed.thing1)/other/($parsed.thing2)"
    }
}
```

then, you'll want an the relevant xdg-desktop file (for example, a `foobar:` scheme), in `$XDG_DATA_DIR/applications` (e.g. `~/.local/share/applications` (named `foobar-scheme-handler.desktop` or anything similar):

```ini
[Desktop Entry]
Exec=/path/to/the/script.nu %u
MimeType=x-scheme-handler/foobar
Name=foobar Scheme Handler
StartupNotify=false
Terminal=false
Type=Application
Version=1.4
```

and add it to `mimeapps.list`, in the same directory:

```ini
...

[Default Applications]
...
x-scheme-handler/foobar=foobar-scheme-handler.desktop

...
```

(i usuall do this with [home-manager](https://github.com/nix-community/home-manager))
