# Text Editor Integration

This section assumes you already have a working binary of *boxes*. Binaries for some platforms can be obtained through the [download page]({{ site.baseurl}}/download.html). Should your platform be missing from the list, you can still download the source distribution and compile your own binary. This sounds harder than it is.

Although *boxes* can be useful when used on the command line, the more frequent use case will be as a filter tied to your editor. So, how can *boxes* be tied to your editor?

Example config file entries are featured so far for [Vim](http://www.vim.org/), [Jed](http://www.jedsoft.org/jed/), and [Emacs](http://www.gnu.org/software/emacs/). If you know how to to this in other editors, please drop me a line!

## Integration with Vim

To call filters from vim, you need to press `!` in visual mode or `!!` in normal mode. So the easiest way to tie in *boxes* with vim is by adding the following four lines to your *.vimrc*:

    vmap ,mc !boxes -d c-cmt<CR>
    nmap ,mc !!boxes -d c-cmt<CR>
    vmap ,xc !boxes -d c-cmt -r<CR>
    nmap ,xc !!boxes -d c-cmt -r<CR>

`<CR>` should be there literally; just paste the lines directly from your browser window. This would comment out the current line or the lines you have marked when you press `,mc` (for *make comment*). Comments can be removed in the same way by pressing `,xc`. Should you feel that `,mc` is too long a combination to type, feel free to choose a shorter one. The above example assumes you are using the standard boxes config file, which features the *c-cmt* design. Of course, the same technique works for any other designs.

While the above example is nice, it does not offer much convenience when you are editing different languages a lot, because you need to remember the hotkey for each different box design. Fortunately, vim has a feature called *autocommands*. They can be used to automatically change the meaning of a key combination depending on what file you edit (any many other things too, of course). Autocommand syntax is

    au[tocmd] [group] {event} {pat} [nested] {cmd}

We can leave out the group. For `{event}`, we choose `BufEnter`, which is generated every time you enter a new buffer, e.g. when starting vim or when switching between open files. `{pat}` is a file glob, and `{cmd}` is our call to *boxes*.

The lines below are from the author's *.vimrc*. They can be pasted directly from your browser window. Their effect is that `,mc` and `,xc` always generate the correct comments for many languages, including C, C++, HTML, Java, lex, yacc, shell scripts, Perl, etc. The default key binding is to generate shell comments using a pound sign (file glob of `*` at the start).

    autocmd BufEnter * nmap ,mc !!boxes -d pound-cmt<CR>
    autocmd BufEnter * vmap ,mc !boxes -d pound-cmt<CR>
    autocmd BufEnter * nmap ,xc !!boxes -d pound-cmt -r<CR>
    autocmd BufEnter * vmap ,xc !boxes -d pound-cmt -r<CR>
    autocmd BufEnter *.html nmap ,mc !!boxes -d html-cmt<CR>
    autocmd BufEnter *.html vmap ,mc !boxes -d html-cmt<CR>
    autocmd BufEnter *.html nmap ,xc !!boxes -d html-cmt -r<CR>
    autocmd BufEnter *.html vmap ,xc !boxes -d html-cmt -r<CR>
    autocmd BufEnter *.[chly],*.[pc]c nmap ,mc !!boxes -d c-cmt<CR>
    autocmd BufEnter *.[chly],*.[pc]c vmap ,mc !boxes -d c-cmt<CR>
    autocmd BufEnter *.[chly],*.[pc]c nmap ,xc !!boxes -d c-cmt -r<CR>
    autocmd BufEnter *.[chly],*.[pc]c vmap ,xc !boxes -d c-cmt -r<CR>
    autocmd BufEnter *.C,*.cpp,*.java nmap ,mc !!boxes -d java-cmt<CR>
    autocmd BufEnter *.C,*.cpp,*.java vmap ,mc !boxes -d java-cmt<CR>
    autocmd BufEnter *.C,*.cpp,*.java nmap ,xc !!boxes -d java-cmt -r<CR>
    autocmd BufEnter *.C,*.cpp,*.java vmap ,xc !boxes -d java-cmt -r<CR>
    autocmd BufEnter .vimrc*,.exrc nmap ,mc !!boxes -d vim-cmt<CR>
    autocmd BufEnter .vimrc*,.exrc vmap ,mc !boxes -d vim-cmt<CR>
    autocmd BufEnter .vimrc*,.exrc nmap ,xc !!boxes -d vim-cmt -r<CR>
    autocmd BufEnter .vimrc*,.exrc vmap ,xc !boxes -d vim-cmt -r<CR>

<a name="jed">&nbsp;</a>

## Integration with Jed

*Andreas Heiduk* (@asheiduk) kindly provided the following excerpt from his *.jedrc*:

    %!% Ripped from "pipe.sl"
    
    variable Last_Process_Command = Null_String;
    
    define do_process_region(cmd) {
       variable tmp;
       tmp = make_tmp_file ("/tmp/jedpipe");
       cmd = strncat (cmd, " > ", tmp, " 2>&1", 4);
    
       !if (dupmark ()) error ("Mark not set.");
    
       if (pipe_region (cmd))
       {
          error ("Process returned a non-zero exit status.");
       }
       del_region ();
       () = insert_file (tmp);
       () = delete_file (tmp);
    }
    
    
    define process_region ()
    {
       variable cmd;
       cmd = read_mini ("Pipe to command:", Last_Process_Command, "");
       !if (strlen (cmd)) return;
    
       Last_Process_Command = cmd;
       do_process_region(cmd);
    }
    
    
    %-----------------------------------------------------------------------
    
    if( BATCH == 0 ){
    
       setkey("process_region",	"\e|");		% ESC-Pipe :-)
       add_completion("process_region");
       
       % define some often used filters
       setkey("do_process_region(\"tal\")",	"\et")	% tal on esc-t
    }

I think it calls [tal](http://www.thomasjensen.com/software/tal/) when you press `ESC-t` (second but last line). Thus, you would have to add a similar line to call *boxes*.
<a name="emacs">&nbsp;</a>


## Integration with Emacs

[Jason L. Shiffer](mailto:jshiffer@zerotao.com) kindly submitted the following information on integrating *boxes* with Emacs:

The simple interface (only a single box style, but easy):

    (defun boxes-create ()
        (interactive)
        (shell-command-on-region (region-beginning) (region-end) "boxes -d c-cmt2" nil 1 nil))
    
    (defun boxes-remove ()
        (interactive)
        (shell-command-on-region (region-beginning) (region-end) "boxes -r -d c-cmt2" nil 1 nil))

Jason also wrote a [*boxes* mode for Emacs](boxes.el). Remember to update its design list when you add new designs to your config file.