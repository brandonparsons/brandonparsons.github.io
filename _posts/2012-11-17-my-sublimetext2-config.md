---
layout: post
title: My SublimeText2 Configuration
tags:
  - tools
  - web development 
description: "ST2 is a great text editor, and is highly configurable. Here I share my config files that provide a comfortable workflow."
---

[SublimeText2](http://www.sublimetext.com/2) is great for editing code and even basic text documents.  For the most part is has completely replaced Microsoft Word or Pages in my daily workflow.

I've really been appreciating the ability to customize the layout of the editor, split-pane editing, and a ton of simple (and extendable) keyboard shortcuts.

*My SublimeText2 Editor Layout*

![My SublimeText2 Editor Layout](/assets/article_images/2012/my-sublimetext-editor.png)

## Theming ##

ST2 is extremely customizable in its visual appearance. Play around with it to find something you find comfortable. You'll want text to readable, with a comfortable background so avoid eye strain.  It also helps to get syntax highlighting for coding.

### Basic Program Theme ###

First you'll want to pick up a theme for the editor itself.  Think sidebars, tabs etc.  I've used [the Soda Theme][soda-theme] since I started with ST2 and it has worked out great.

*Soda Theme*

![Soda Theme for SublimeText2](/assets/article_images/2012/soda-theme.png)

[soda-theme]: https://github.com/buymeasoda/soda-theme

### Colour Schemes ###

Obviously this one will be personal preference, but it helps to use a legible set of colours that are comfortable for you. I had been using the [Railscasts Theme][] for some time, and just recently swapped over to a [GitHub Theme][] to shake things up.

*GitHub Colour Scheme*

![GitHub Colour Scheme for SublimeText2](/assets/article_images/2012/github-colors.png)

*RailsCasts Colour Scheme*

![RailsCasts colour scheme for SublimeText2](/assets/article_images/2012/railscasts-colors.png)

[Railscasts Theme]: https://github.com/talltroym/sublime-theme-railscasts
[GitHub Theme]: https://github.com/sbecker/github_textmate_theme

## Configuration ##

Although it is an incredibly powerful editor, it is important to get your configuration settings right to get the most out of it. ST2 has built-in functionality to format your text as you go so you don't have to worry about it.

You can also set up syntax-specific configuration files in order to provide different behvaiour between Ruby files and Markdown files for example.

Here are my configuration files, use what you will, but be sure to play around and see what works for you!

### General Config ###

```json
{
  "font_face": "Ubuntu Mono",
  "font_options":
  [
      "subpixel_antialias"
  ],
  "font_size": 16.0,

  "theme": "Soda Light.sublime-theme",
  "color_scheme": "Packages/GitHub Colour Scheme/GitHub.tmTheme",
  // "color_scheme": "Packages/RailsCasts Colour Scheme/RailsCastsColorScheme.tmTheme",

  "bold_folder_labels": true,
  "margin": 0,
  "caret_style": "phase",
  "rulers": [80],
  "soda_classic_tabs": true,
  "tab_size": 2,

  "trim_trailing_white_space_on_save": true,
  "ensure_newline_at_eof_on_save": true,
  "translate_tabs_to_spaces": true,

  "auto_complete_commit_on_tab": true,
  "shift_tab_unindent": true,
  "highlight_line": true,
  "scroll_past_end": true,
  "move_to_limit_on_up_down": true,
  "highlight_modified_tabs": true,
  "detect_indentation": true,

  "ignored_packages":
  [
    "Vintage"
  ]
}

```

### Custom Key Mapping ###

```json
[
  // to see output:
  // sublime.log_commands(True)

  // open this keybindings file
  {
    "keys": ["super+ctrl+,"],
    "command": "open_file",
    "args": {"file": "${packages}/User/Default ($platform).sublime-keymap"}
  },

  // go to line
  {
    "keys": ["ctrl+super+l"],
    "command": "show_overlay",
    "args": {"overlay": "goto", "text": ":"}
  },

  // copy file name
  {
    "keys": ["super+shift+c"],
    "command": "copy_path_to_clipboard"
  },

  // ctags
  {
    "keys": ["ctrl+]"],
    "command": "navigate_to_definition"
  },

  {
    "keys": ["super+alt+p"],
    "command": "rebuild_tags"
  },

  // go to alternate file
  // super+ctrl+. will show spec and implementation in split view
  {
    "keys": ["ctrl+shift+down"],
    "command": "open_rspec_file"
  },

  // Duplicate selection
  { "keys": ["ctrl+shift+d"], "command": "duplicate_line" },

  // show in project drawer
  { "keys": ["ctrl+super+r"], "command": "reveal_in_side_bar"},

  // select coffeescript twin
  {
    "keys": ["ctrl+shift+down"],
    "command": "open_coffee_twin",
    "context": [ {"key": "selector", "operator": "equal", "operand": "source.coffee"} ]
  },

  // compile and display coffeescript
  {
    "keys": ["super+b"],
    "command": "compile_and_display_js",
    "context": [ {"key": "selector", "operator": "equal", "operand": "source.coffee"} ]
  },

  // close all other tabs
  {
    "keys": ["super+alt+w"],
    "command": "close_other_tabs"
  },

  // focus on side bar
  {
    "keys": ["ctrl+shift+tab"],
    "command": "focus_side_bar"
  },

  // focus back in buffer
  {
    "keys": ["ctrl+tab"],
    "command": "focus_group",
    "args": {"group": 0}
  },

  // TEST STUFF

  // single test
  { "keys": ["super+shift+r"], "command": "run_single_ruby_test" },
  // test file
  { "keys": ["super+shift+t"], "command": "run_all_ruby_test" },
  // test file
  { "keys": ["super+shift+e"], "command": "run_last_ruby_test" },

  // show test panel
  { "keys": ["super+shift+x"], "command": "show_test_panel" },

  // verify ruby syntax
  { "keys": ["alt+shift+v"], "command": "verify_ruby_file" },

  // switch between code and test in single mode
  { "keys": ["super+period"], "command": "switch_between_code_and_test", "args": {"split_view": false}, "context" : [
    { "key": "selector", "operator": "equal", "operand": "source.ruby", "match_all": true }
  ]},

  // switch between code and test in split view
  { "keys": ["super+ctrl+period"], "command": "switch_between_code_and_test", "args": {"split_view": true}, "context" : [
    { "key": "selector", "operator": "equal", "operand": "source.ruby", "match_all": true }
  ]} ,

  // go to symbol
  {
    "keys": ["super+t"],
    "command": "show_overlay",
    "args": {"overlay": "goto", "text": "@"}
  },

  { "keys": ["ctrl+super+k"], "command": "toggle_side_bar" },

  // ["super+shift+v"] is default the command for paste and indent. Some folks may want this
  // to be the default behavior - simply add this to the Key Bindings - User file to swap the keybindings.
  { "keys": ["super+v"], "command": "paste_and_indent" },
  { "keys": ["super+shift+v"], "command": "paste" }

]
```

### Markdown & MultiMarkdown Settings ###

```json
{
  // Which file extensions go with this file type?
  "extensions":
  [
      "md",
      "mdown",
      "mdwn",
      "mmd",
      "txt"
  ],

  // Set to true to turn spell checking on by default
  "spell_check": true,
  "trim_trailing_white_space_on_save": false
}
```

### RubyTest Configuration ###

```json
{
  "erb_verify_command": "erb -xT - {file_name} | ruby -c",
  "ruby_verify_command": "ruby -c {file_name}",

  "run_ruby_unit_command": "ruby -Itest {relative_path}",
  "run_single_ruby_unit_command": "ruby -Itest {relative_path} -n '{test_name}'",

  "run_cucumber_command": "cucumber {relative_path}",
  "run_single_cucumber_command": "cucumber {relative_path} -l{line_number}",

  "run_rspec_command": "bundle exec rspec {relative_path}",
  "run_single_rspec_command": "bundle exec rspec {relative_path} -l{line_number}",

  "ruby_unit_folder": "test",
  "ruby_cucumber_folder": "features",
  "ruby_rspec_folder": "spec",

  "ruby_use_scratch" : false,
  "save_on_run": false,
  "ignored_directories": [".git", "vendor", "tmp"]
}
```

## Packages ##

The extensibility of SublimeText2 is one of its killer features. There are a pile of useful plugins out there - most of which I haven't had a chance to use yet.  In any case, here's a list of plugins which I use frequently and I've found quite helpful:

- HAML
- HTML
- JavaScript
- Markdown
- **MarkdownEditing** *(adds a nice comfortable colour scheme, and a ton of useful keyboard shortcuts)*
- Rails
- RSpec
- Ruby
- **RubyTest** *(lets you run tests from inside of the ST2 editor)*
- SideBarEnhancements

## Wrap-up ##

I'm no expert with ST2, but have been using it enough to get comfortable with setup and configuration.  Happy to answer any questions.
