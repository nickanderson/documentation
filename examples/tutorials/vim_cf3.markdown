---
layout: default
title: Installing configuring and using the vim_cf3 plugin
sorting: 15
categories: [Examples, Tutorials]
published: true
alias: examples-tutorials-vim.html
tags: [Examples, Tutorials, vim]
---

Configuring your editor is one of the easiest things you can do to increase
your productivity for both learning and writing CFEngine policy. Syntax
highlighting significatly helps pattern recognition and can help you identify
possible syntax errors as you type.

In this tutorial you will learn how to install and configure vim for use with
CFEngine 3 using [Neil Watson's][http://ca.linkedin.com/in/neilhwatson]
`vim_cf3` plugin.

## Overview 
This tutorial provides instructions for the following:

* [Install the vim plugin][Installing configuring and using the vim_cf3 plugin#Installing the vim plugin]

* [Configure vim to use the plugins][Installing configuring and using the vim_cf3 plugin#Configure vim to use the plugins]

* [Work with vim_cf3][Installing configuring and using the vim_cf3 plugin#Work with vim_cf3]


This tutorial assumes that you have vim installed and have a working knowledge of it.

## Installing the vim plugin

There are several ways to install vim plugins. [Tim
Pope's][www.linkedin.com/in/tpope0]
[pathogen.vim][https://github.com/tpope/vim-pathogen] plugin provides an easy
way to manage vim plugins and will be used in this tutorial.

Note: The commands shows are relevant for UNIX like systems. Windows users will
need to adjust paths accordingly. And may need to install curl or manually
download the files and place them in the appropriate locations. On windows
~/.vimrc is %HOMEPATH%\_vimrc ~/.vim/ is %HOMEPATH%\vimfiles. Also note that
tar.gz files downloaded from github are also available as .zip files.

Download and install pathogen:
```
mkdir -p ~/.vim/autoload ~/.vim/bundle
curl -Sso ~/.vim/autoload/pathogen.vim \
    https://raw.github.com/tpope/vim-pathogen/master/autoload/pathogen.vim
```

Download and install vim_cf3:
```
curl -Sso ~/.vim/bundle/vim_cf3.tar.gz \
    https://github.com/neilhwatson/vim_cf3/archive/master.tar.gz
tar --ungzip --extract --file ~/.vim/bundle/vim_cf3.tar.gz --directory ~/.vim/bundle
rm -f ~/.vim/bundle/vim_cf3.tar.gz
```

[Back to top of page.][Installing configuring and using the vim_cf3 plugin#Overview]

## Configure vim to use the plugins

Once the plugins have been installed your vimrc needs to be configured for vim
to use the plugins.

Enable pathogen.vim by adding the following linesto your vimrc:
```
execute pathogen#infect()
syntax on
filetype plugin indent on
```

Enable vim_cf3 for files with .cf extension by adding the following lines to your vimrc:
```
" Set the filetype appropriately
au BufRead,BufNewFile *.cf set filetype=cf3
```

Optionally add these additonal settings to improve your experience:
```
" Adjust settings for cf3 file type
autocmd FileType cf3 setlocal tabstop=2 smarttab expandtab shiftwidth=2 softtabstop=2 number nofoldenable

" Turn off some vim_cf3 feautures that can cause confusion
let g:DisableCFE3KeywordAbbreviations=0
```

The optional settings turn on line numbers, adjust tabs and shifting in
accordance with the CFEngine `Policy Style Guide`. It also disables folding and
special keyword abbreviations which can cause confusion unless your familiar
with how to use them. 

Now when you open a .cf file with vim, you should have syntax highlighted policy similar to this example:

![Syntax Highlighted Policy](vim_cf3_screenshot.jpg)

Note: Colors used will vary depending on your terminal and other vim settings.

[Back to top of page.][Installing configuring and using the vim_cf3 plugin#Overview]

## Work with vim_cf3

Now that you have vim configured for working with CFEngine policy lets cover
some of the `vim_cf3` features.

### Tidying your policy

The vim_cf3 plugin has some automatic re-alignement features that help you keep
your policy looking nice.

```cf3
bundle agent example
{
  vars:
    "cat" string => "hat";
    "redfish" string => "bluefish";
    "wocket" string => "pocket";
}
```

By placing your cursor on any of the lines containing "string" and pressing
*<ESC>=* the variables will be re-aligned nicely and look like the following example.

```cf3
bundle agent example
{
  vars:
    "cat"     string => "hat";
    "redfish" string => "bluefish";
    "wocket"  string => "pocket";
}
```

A similar function is useful for larger promises with multiple attributes.

```cf3
bundle agent example
{
  files:
    "/etc/motd"
      edit_defaults => empty,
      edit_line => append_if_no_line("Managed by CFEngine"),
      perms => mog("600", "root", "root"),
      handle => "example_files_motd",
      comment => "Advertising that the host is managed by cfengine lets people know as soon as they log in that manual changes may be reverted";
}
```

By placing your cursor on any of the attribute lines and pressing *,=* the
promise attributes will be re-aligned as follows.

```cf3
bundle agent example
{
  files:
    "/etc/motd"
      edit_defaults  => empty,
      edit_line      => append_if_no_line("Managed by CFEngine"),
      perms          => mog("600", "root", "root"),
      handle         => "example_files_motd",
      comment        => "Advertising that the host is managed by cfengine lets people know as soon as they log in that manual changes may be reverted";
}
```

### Quickly Quoting lists

CFEngines implicit list iteration is very powerful. Building lists with the vim cf3 plugin is easy.

Start off by constructing your list as shown below.

```cf3
bundle agent example
{
  vars:
    "mylist" slist => {
onefish
twofish
redfish
bluefish
};
}
```

To properly quote and instert seperating commas between each list element
simply use vims visual select mode to select all of the list elements and then
press *,q*.

```cf3
bundle agent example
{
  vars:
    "mylist" slist => {
"onefish",
"twofish",
"redfish",
"bluefish",
};
}
```

Now you can simply indent your list elements using vims visual block mode.
Place your cursor at the beginning of the first list elements line press
*<ctrl>v* and press *down arrow* or *j* until the leading quote of each list
element is highlighted and the lists closing curley brace is highlighted. Now
simply press *<shift>I* and tab or space until the first element is properly
aligned and press *<ESC>*. All of your list elements will be shifted over the
same number of spaces.

```cf3
bundle agent example
{
  vars:
    "mylist" slist => {
                       "onefish",
                       "twofish",
                       "redfish",
                       "bluefish",
                      };
}
```

[Back to top of page.][Installing configuring and using the vim_cf3 plugin#Overview]
