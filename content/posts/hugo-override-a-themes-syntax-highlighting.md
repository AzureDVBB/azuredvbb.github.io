---
title: "Hugo - Override a Theme's Syntax Highlighting"
date: 2023-02-09T14:59:43+01:00
draft: false
---

In the theme that I use on this site [Poison](https://github.com/lukeorth/poison), the syntax highlighting for codeblocks was not working by default.

Hugo gives us the ability to override theme files by the `static/` folder in our site's root directory (where `config.toml` lives).

More specifically, in the case of Poison, the file in question was under:
```
themes/poison/static/css/syntax.css
```

## Generate the desired syntax highlighting stylesheet

To override it we need to generate a `syntax.css` file for our desired syntax highlighting style.
Hugo has a guide to do so [available in their documentation](https://gohugo.io/content-management/syntax-highlighting/), but in short use the following command in the site's root directory:

```
hugo gen chromastyles --style=monokai > syntax.css
```

This generates the `monokai` style, changing the `--style=` parameter will allow you to use other supported styles, check the [list of availablable styles in their documentation](https://gohugo.io/content-management/syntax-highlighting/#list-of-chroma-highlighting-languages).

## Move the file into the correct subfolder

In our case, we need to put the file here: `static/css/syntax.css`

This will override the theme's file inside `/themes/THEMENAME/static/css/syntax.css`

## Setup the `config.toml` file for our site

Now we need to change the configuration (or ensure that it is setup) so that syntax highlighting will be supported.

[As per the default from their documentation available here](https://gohugo.io/getting-started/configuration-markup/#highlight), this was added to my `config.toml` file to make sure everything works fine:

```toml
[markup]
  [markup.highlight]
    anchorLineNos = false
    codeFences = true
    guessSyntax = false
    hl_Lines = ''
    hl_inline = false
    lineAnchors = ''
    lineNoStart = 1
    lineNos = false
    lineNumbersInTable = true
    noClasses = true
    noHl = false
    style = 'monokai'
    tabWidth = 4
```

