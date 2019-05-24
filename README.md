ipython-cell
============

Seamlessly run Python code from Vim in IPython, including executing individual
code cells similar to Jupyter notebooks and MATLAB.

This plugin is especially suited for data exploration and visualization using
Python. You could for example define a code cell that loads your input data,
and another code cell to visualize the data. This plugin then allows you to
change and re-run the visualization part of your code without having to reload
the data each time.


Demo
----

![Demo animation](../assets/ipython-cell-demo.gif?raw=true)


Requirements
------------

ipython-cell requires Vim or Neovim to be compiled with Python 2 or Python 3
support (`+python` or `+python3` when running `vim --version`). If both Python
versions are found, the plugin will prefer Python 3.

Additionally, the cell execution feature requires [xclip] or [xsel] to be
installed (preferring the former).

ipython-cell depends on [vim-slime] to send the code to IPython (see
[Installation](#installation) instructions below).

[xclip]: https://github.com/astrand/xclip
[xsel]: https://github.com/kfish/xsel
[vim-slime]: https://github.com/jpalardy/vim-slime


Installation
------------

It is easiest to install ipython-cell using a plugin manager. I personally
recommend [vim-plug]. See respective plugin manager's documentation for more
information about how to install plugins.


### [vim-plug]

~~~vim
Plug 'jpalardy/vim-slime', { 'for': 'python' }
Plug 'hanschen/vim-ipython-cell', { 'for': 'python' }
~~~


### [Vundle]

~~~vim
Plugin 'jpalardy/vim-slime'
Plugin 'hanschen/vim-ipython-cell'
~~~


### [NeoBundle]

~~~vim
NeoBundle 'jpalardy/vim-slime', { 'on_ft': 'python' }
NeoBundle 'hanschen/vim-ipython-cell', { 'on_ft': 'python' }
~~~


### [Dein]

~~~vim
call dein#add('jpalardy/vim-slime', { 'on_ft': 'python' })
call dein#add('hanschen/vim-ipython-cell', { 'on_ft': 'python' })
~~~


### [Pathogen]

~~~sh
cd ~/.vim/bundle
git clone https://github.com/hanschen/vim-ipython-cell.git
~~~

[vim-plug]: https://github.com/junegunn/vim-plug
[Vundle]: https://github.com/VundleVim/Vundle.vim
[NeoBundle]: https://github.com/Shougo/neobundle.vim
[Dein]: https://github.com/Shougo/dein.vim
[Pathogen]: https://github.com/tpope/vim-pathogen


Usage
-----

ipython-cell sends code from Vim to IPython running in a GNU screen / tmux /
whimrepl / ConEmu session / NeoVim Terminal / Vim Terminal using [vim-slime].
It is recommended that you familiarize yourself with [vim-slime] first before
using ipython-cell. Once you grasp the basics of vim-slime, using ipython-cell
will be a breeze.

(I personally prefer tmux, but you will find screen installed on most *nix
systems.)

The plugin does not define any key mappings by default, but comes with the
following commands, which I recommend that you bind to key combinations that
you like (see e.g.  [Example Vim Configuration](#example-vim-configuration)
below).


### Commands

    :IPythonCellExecuteCell

Execute the current code cell in IPython.

    :IPythonCellExecuteCellJump

Execute the current code cell in IPython and jump to the next cell.

    :IPythonCellRun

Run the whole script in IPython.

    :IPythonCellRunTime

Run the whole script in IPython and time the execution.

    :IPythonCellClear

Clear IPython screen.

    :IPythonCellClose

Close all figure windows.

    :IPythonCellPrevCell

Jump to the previous cell header.

    :IPythonCellNextCell

Jump to the next cell header.

[vim-slime]: https://github.com/jpalardy/vim-slime


Defining code cells
-------------------

Code cells are defined by either Vim marks or special text in the code,
depending on if `g:ipython_cell_delimit_cells_by` is set to `'marks'` or
`'tags'`, respectively. The default is to use Vim marks (see `:help mark` in
Vim).

The examples below show how code cell boundaries work.


### Code cells defined using marks

Marks are depicted as letters in the left-most column.

~~~
                                   _
  | import numpy as np              | cell 1
  |                                _| 
a | numbers = np.arange(10)         | cell 2
  |                                 |
  |                                _|
b | for n in numbers:               | cell 3
  |     print(n)                   _|
c |     if n % 2 == 0:              | cell 4
  |         print("Even")           |
  |     else:                       |
  |         print("Odd")            |
  |                                _|
d | total = numbers.sum()           | cell 5
  | print("Sum: {}".format(total)) _|

~~~

Note that code cells can be defined inside statements such as `for` loops.
IPython's `%paste` will automatically dedent the code before execution.
However, if the code cell is defined inside e.g. a `for` loop, the code cell
*will not* iterate over the loop.

In the example above, executing cell 4 after cell 3 will print `Odd` only once
because IPython will execute the following code:

~~~python
for n in numbers:
    print(n)

~~~

for cell 3, followed by

~~~python
if n % 2 == 0:
    print("Even")
else:
    print("Odd")

~~~

for cell 4. The `for` statement is no longer included for cell 4.

You must therefore be careful when defining code cells inside statements.


### Code cells defined using tags

Using `##` to define cell boundaries.

~~~
                                   _
import numpy as np                  | cell 1
                                   _|
## Setup                            | cell 2
                                    |
numbers = np.arange(10)             |
                                   _|
## Print numbers                    | cell 3
                                    |
for n in numbers:                   |
    print(n)                        |
                                   _|
    ## Odd or even                  | cell 4
                                    |
    if n % 2 == 0:                  |
        print("Even")               |
    else:                           |
        print("Odd")                |
                                   _|
## Print sum                        | cell 5
                                    |
sum = numbers.sum()                 |
print("Sum: {}".format(sum))        |
print("Done.")                     _|

~~~

See note in the previous section about defining code cells inside statements
(such as cell 4 inside the `for` loop in the example above).


Configuration
-------------

    g:ipython_cell_delimit_cells_by

Specify if cells should be delimited by `marks` or `tags`. Default: `marks`

    g:ipython_cell_tag

If cells are delimited by tags, specify the format of the tags. Default: `##`

    g:ipython_cell_valid_marks

If cells are delimited by marks, specify which marks to use.
Default: `abcdefghijklmnopqrstuvqxyzABCDEFGHIJKLMNOPQRSTUVWXYZ`


Example Vim configuration
-------------------------

Here's an example of how to configure your `.vimrc` to use this plugin. Adapt
it to suit your needs.

~~~vim
if has('autocmd')
    filetype plugin indent on
endif

" Load plugins using vim-plug
call plug#begin('~/.vim/plugged')
Plug 'jpalardy/vim-slime' { 'for': 'python' }
Plug 'hanschen/vim-ipython-cell' { 'for': 'python' }
call plug#end()

"------------------------------------------------------------------------------
" slime configuration 
"------------------------------------------------------------------------------
" always use tmux
let g:slime_target = "tmux"

" fix paste issues in ipython
let g:slime_python_ipython = 1

" always send text to the top-right pane in the current tmux tab without asking
let g:slime_default_config = {
            \ "socket_name": get(split($TMUX, ","), 0),
            \ "target_pane": "{top-right}" }
let g:slime_dont_ask_default = 1

"------------------------------------------------------------------------------
" ipython-cell configuration
"------------------------------------------------------------------------------
" Use '# <cell>' to define cells instead of using marks
g:ipython_cell_delimit_cells_by = 'tags'
g:ipython_cell_tag = '# <cell>'

" Keyboard mappings. <Leader> is \ (backslash) by default

" map <Leader>r to run script
autocmd FileType python nmap <buffer> <Leader>r :IPythonCellRun<CR>

" map <Leader>R to run script and time the execution
autocmd FileType python nmap <buffer> <Leader>R :IPythonCellRunTime<CR>

" map <Leader>c to execute the current cell
autocmd FileType python nmap <buffer> <Leader>c :IPythonCellExecuteCell<CR>

" map <Leader>C to execute the current cell and jump to the next cell
autocmd FileType python nmap <buffer> <Leader>C :IPythonCellExecuteCellJump<CR>

" map <Leader>l to clear IPython screen
autocmd FileType python nmap <buffer> <Leader>l :IPythonCellClear<CR>

" map <Leader>x to close all Matplotlib figure windows
autocmd FileType python nmap <buffer> <Leader>x :IPythonCellClose<CR>

" map [c and ]c to jump to the previous and next cell header
autocmd FileType python nmap <buffer> [c :IPythonCellPrevCell<CR>
autocmd FileType python nmap <buffer> ]c :IPythonCellNextCell<CR>

" map <Leader>h to send the current line or current selection to IPython
autocmd FileType python nmap <buffer> <Leader>h <Plug>SlimeLineSend
autocmd FileType python xmap <buffer> <Leader>h <Plug>SlimeRegionSend

~~~

Note that the mappings as defined here work only in normal mode (see
`:help mapping` in Vim for more information). The extra
`autocmd FileType python` and `<buffer>` parts are there just to ensure that
the mapping are defined only for Python files. You can also move these
mappings to `~/.vim/ftplugin/python.vim` and drop `autocmd FileType python`.

If you come from the MATLAB world, you may want e.g. F5 to save and run the
script regardless if you are in insert or normal mode:

~~~vim
" map <F5> to save and run script
autocmd FileType python nmap <buffer> <F5> :w<CR>:IPythonCellRun<CR>
autocmd FileType python imap <buffer> <F5> <C-o>:w<CR><C-o>:IPythonCellRun<CR>
~~~


FAQ
---

> Should I use `'marks'` or `'tags'` to define cells?

This depends on personal preference. I used to use `'tags'` because they are
similar to MATLAB's `%%` to define cells. `'tags'` are nice if you want the
cells to be saved together with your files, and may be easier to start with.
I switched to `'marks'` after discovering that I can show the marks in the
left-most column. I find `'marks'` to be more flexible because you add and
change cells without changing the code, which is nice when your code is under
version control.

> How do I show the marks in the left-most column?

Use the vim-signature plugin: https://github.com/kshenoy/vim-signature

> How to send only the current line or selected lines to IPython?

Use the features provided by vim-slime, see the
[Example Vim Configuration](#example-vim-configuration) for an example.
The default mapping `C-c C-c` (hold down Ctrl and tap the C key twice) will
send the current paragraph or the selected lines to IPython. See `:help slime`
for more information, in particular the documentation about
`<Plug>SlimeRegionSend` and `<Plug>SlimeLineSend`.

> Why do I get "name 'plt' is not defined" when I try to close figures?

ipython-cell assumes that you have imported `matplotlib.pyplot` as `plt` in
IPython. If you prefer to import `matplotlib.pyplot` differently, you can
achieve the same thing using vim-slime, for example by adding the following to
your .vimrc:

    autocmd FileType python nmap <buffer> ,x :SlimeSend1 matplotlib.pyplot.close('all')<CR>

> How can I send other commands to IPython, e.g. '%who'?

You can easily send arbitary commands to IPython using the `:SlimeSend1`
command provided by vim-slime, e.g. `:SlimeSend1 %who`, and map these commands
to key combinations.

> Why isn't this plugin specific to Python by default? In other words, why do
> I have to add all this extra stuff to make this plugin Python-specific?

This plugin was created with Python and IPython in mind, but I don't want to
restrict the plugin to Python by design. Instead, I have included examples of
how to use plugin managers to specify that the plugin should be loaded only
for Python files and how to create Python-specific mappings. If someone wants
to use this plugin for other filetypes, they can easily do so.

> Why is this plugin written in Python instead of pure Vimscript?

Because I feel more comfortable with Python and don't have the motivation to
learn Vimscript for this plugin. If someone implements a pure Vimscript
version, I would be happy to consider to merge it.


Related plugins
---------------

* [tslime\_ipython] - Similar to ipython-cell but with some small differences.
  For example, tslime\_ipython pastes the whole code that's sent to IPython
  to the input line, while ipython-cell uses IPython's `%paste -q` command to
  make the execution less verbose.
* [vim-ipython] - Advanced two-way integration between Vim and IPython. I never
  got it to work as I want, i.e., don't show the code that's executed but show
  the output from the code, which is why I created this simpler plugin.
* [vim-tmux-navigator] - Seamless navigation between Vim splits and tmux panes.
* [vim-signature] - Display marks in the left-hand column.

[tslime\_ipython]: https://github.com/eldridgejm/tslime_ipython
[vim-tmux-navigator]: https://github.com/christoomey/vim-tmux-navigator
[vim-ipython]: https://github.com/ivanov/vim-ipython
[vim-signature]: https://github.com/kshenoy/vim-signature


Thanks
------

ipython-cell was heavily inspired by [tslime\_ipython].
The code logic to determine which Python version to use was taken from
[YouCompleteMe].

[tslime\_ipython]: https://github.com/eldridgejm/tslime_ipython
[YouCompleteMe]: https://github.com/Valloric/YouCompleteMe
