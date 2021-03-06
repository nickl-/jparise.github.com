---
layout: post
title: Vim Color Schemes
category: vim
---

The [Vim][] text editor supports highly-configurable color schemes which build
upon the editor's rich syntax highlighting system.  The stock Vim distribution
includes a number of color schemes, and many more are available from the [Vim
Scripts repository][scripts].

Color scheme definitions are simply normal Vim scripts that live in the
`colors/` directory of the Vim runtime hierarchy (see `:help runtimepath`).

Color schemes are loaded using the `:colorscheme` command.  The scheme's name
is determined by the filename of its script file (minus the `.vim` extension).
For example, to load the stock *blue* color scheme (which is defined by the
`colors/blue.vim` script):

{% highlight vim %}

    :colorscheme blue

{% endhighlight %}

## Creating Color Schemes

Creating a custom color scheme is quite easy.  Start by creating a new Vim
script file in the `colors/` directory based on the name of the new scheme.
Start the script with the following commands:

{% highlight vim %}

    set background=dark "or light
    highlight clear
    if exists("syntax_on")
        syntax reset
    endif
    let g:colors_name = "example"

{% endhighlight %}

These commands will reset the syntax highlighting system to its default state.
Note that some color scheme scripts might prefer a light background, so that
first line should be changed accordingly.  (`highlight clear` uses the
*background* value, so *background* must be set first.)

The final line sets the global `colors_name` variable to the scheme's name
(*example*, in this example).

The rest of the script defines the color scheme itself.  This is accomplished
primarily through the `highlight` (or `hi`) command.  Each `highlight` command
sets the colors for a single syntax group.  Setting the colors for the
*Comments* group might look like this:

{% highlight vim %}

    hi Comment ctermbg=black ctermfg=darkgrey guibg=#000000 guifg=#777777

{% endhighlight %}

To see the full list of Vim's syntax groups (along with their current
highlight settings), run the following command from within the editor:

{% highlight vim %}

    :source $VIMRUNTIME/syntax/hitest.vim

{% endhighlight %}

## Testing Runtime Features

Because the color scheme is simply a Vim script, you can conditionalize the
definitions based on various runtime values.  The presence of the
`gui_running` feature indicates that the Vim GUI is running, for example:

{% highlight vim %}

    if has('gui_running')
        " GUI colors
    else
        " Non-GUI (terminal) colors
    endif

{% endhighlight %}

And the terminal's color range -- the number of available colors -- can be
queried via the `&t_Co` variable:

{% highlight vim %}

    if &t_Co > 255
        " More than 256 colors are available
    endif

{% endhighlight %}

## Additional Configuration

Color scheme scripts can support basic configuration using global variables.

{% highlight vim %}

    if exists("g:example_force_dark")
        set background=dark
    endif

{% endhighlight %}

The user should set this variable in his `.vimrc` file before loading the
color scheme script.

{% highlight vim %}

    let g:example_force_dark = 1
    colorscheme example

{% endhighlight %}

(Global variables can be "unset" using the `unlet` command.)

[vim]: http://www.vim.org/
[scripts]: http://www.vim.org/scripts/script_search_results.php?keywords=&script_type=color+scheme&order_by=rating&direction=descending&search=search
