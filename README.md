# Unofficial TailwindCSS Obsidian Plugin

This plugin uses [TailwindCSS](https://tailwindcss.com/) (via
[PostCSS](https://postcss.org/)) to produce a CSS snippet in your Obsidian
vault.

## How it works

This plugin implements
[Tailwind's PostCSS installation guide](https://tailwindcss.com/docs/installation/using-postcss)
with the [`postcss`](https://postcss.org/api/) JavaScript API. This produces a
CSS snippet which **you must manually enable** once it is created. This snippet
will automatically update based on changes to your files and settings.

This plugin makes use of the
[FileSystemAdapter](https://docs.obsidian.md/Reference/TypeScript+API/FileSystemAdapter/FileSystemAdapter)
API to read and write files in your vault's configuration directory.

## Installation

At the time of writing, this plugin has not yet been submitted as a community
plugin. Therefore, you must manually install this plugin, as you might do so for
the [`hot-reload` plugin](https://github.com/pjeby/hot-reload).

In the Releases section for this repository, you can find the zip archive that
contains all of the individual files. Unzip that archive so that its contents
are in a new directory called `unofficial-tailwindcss-plugin/`. Then you should
be good to go after restarting Obsidian and enabling the plugin.

Don't forget to enable the generated CSS snippet after it appears.

## Settings

### Preflight

[Preflight](https://tailwindcss.com/docs/preflight) is a set of base styles that
Tailwind injects into its `@tailwind base` directive. In a more literal sense,
[`preflight`](https://github.com/tailwindlabs/tailwindcss/blob/master/src/corePlugins.js#L494)
is one of Tailwind's core plugins, and it parses a static stylesheet named
[`preflight.css`](https://github.com/tailwindlabs/tailwindcss/blob/master/src/css/preflight.css),
feeding the styles into its `addBase` utility function.

However, the `preflight` plugin uses `__dirname` and `path.join` to locate the
stylesheet, which results in an error in the context of the Obsidian application
built with [Electron](https://www.electronjs.org/). Additionally, Preflight
styles conflict with Obsidian's base styles such that:

1. the Obsidian UI is affected by Preflight styles, and
2. some Preflight styles are overshadowed by Obsidian styles.

Therefore, this plugin (the Unofficial TailwindCSS Obsidian Plugin) does not
apply Preflight styles by default. You can enable Preflight in the plugin
settings, which will direct the plugin to source a packaged copy of
`preflight.css` using the Obsidian API.

### Prefix selector

You may also control what is affected by Tailwind's CSS rules by enabling this
plugin's prefix selector option. This will add a prefix (default `.tailwind`) to
all CSS selectors using a
[descendant combinator](https://developer.mozilla.org/en-US/docs/Web/CSS/Descendant_combinator).
so that `.a, #foo.bar` becomes `.tailwind .a, .tailwind #foo.bar`.

You can configure this so that only certain notes (or even certain sections of
notes) are affected by Tailwind.

#### Combining with Preflight setting

You may have mixed success with the Preflight setting described above. If
Obsidian's CSS rules are taking precedence over Preflight's styles, then setting
a prefix selector may help Preflight take control.

### PostCSS entrypoint

By default, this plugin will use Tailwind's default input template, which
combines three of its directives:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

You can specify a custom entrypoint using a file path that is relative to your
vault's configuration directory (`.obsidian/`. This is useful if you want to use
Tailwind's `@layer` directive for your own purposes.

See
[Using CSS and @layer](https://tailwindcss.com/docs/adding-custom-styles#using-css-and-layer)
from Tailwind's documentation for more information.

### Tailwind theme

You can specify a custom Tailwind theme using a file path that is relative to
your vault's configuration directory. The file contents should be a JSON object.
This is useful if you want to replace or extend Tailwind's default theme.

Two caveats to this setting:

-   This is different from the entire Tailwind configuration object. You may
    **not** use a `tailwind.config.js` file that exports a JavaScipt module.
-   You may **not**
    [dynamically reference other values](https://tailwindcss.com/docs/theme#referencing-other-values)
    or the
    [default theme](https://tailwindcss.com/docs/theme#referencing-other-values).

For now, you are limited to the themes you can define with a static JSON file.

### Tailwind content

You can specify file globs relative to your vault's configuration directory that
will be included in Tailwind's process. This is helpful if other files contain
Tailwind CSS class names and use them to dynamically generate Markdown content.

## Developer Notes

This is a fork of Obsidian's sample plugin repository. Changes other than
implementing this plugin include:

-   Adding `predev` and `prebuild` NPM scripts to automatically copy Tailwind's
    `preflight.css` file from `node_modules` into the project root.
-   Taking some extra notes about GitHub workflows from Obsidian docs and
    [pjeby's `hot-reload` plugin](https://github.com/pjeby/hot-reload).
-   Adding a custom `esbuild` plugin to copy this plugin's files into a test
    vault.

### Test Vault

This repository contains an example Obsidian vault to showcase and test the
plugin's functionality. You will need to enable this plugin after initially
opening the folder in Obsidian, and then you must enable the generated CSS
snippet after that to see the effects.

This repository also declares `hot-reload` as a submodule within the test
vault's `.obsidian/plugins/` directory. In order to actually download
`hot-reload`, you must run the following commands after cloning this repository:

```bash
git submodule init
git submodule update
```

After that, you should be able to run `npm run dev` and then open the vault.
