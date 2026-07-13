
>Vim uses Unix principle

In contrast to Nano, `Vim` is a modal editor that can distinguish between text and command input. Vim offers a total of six fundamental modes that make our work easier and make this editor so powerful:

|**Mode**|**Description**|
|---|---|
|`Normal`|In normal mode, all inputs are considered as editor commands. So there is no insertion of the entered characters into the editor buffer, as is the case with most other editors. After starting the editor, we are usually in the normal mode.|
|`Insert`|With a few exceptions, all entered characters are inserted into the buffer.|
|`Visual`|The visual mode is used to mark a contiguous part of the text, which will be visually highlighted. By positioning the cursor, we change the selected area. The highlighted area can then be edited in various ways, such as deleting, copying, or replacing it.|
|`Command`|It allows us to enter single-line commands at the bottom of the editor. This can be used for sorting, replacing text sections, or deleting them, for example.|
|`Replace`|In replace mode, the newly entered text will overwrite existing text characters unless there are no more old characters at the current cursor position. Then the newly entered text will be added.|
|`Ex`|Emulates the behavior of the text editor [Ex](https://man7.org/linux/man-pages/man1/ex.1p.html), one of the predecessors of `Vim`. Provides a mode where we can execute multiple commands sequentially without returning to Normal mode after each command.
> Note: Entering the tutor mode in `vim` editor can be done using the `Command` mode `:Tutor` or by using the `vimtutor` command in the shell.