# Chapter 3. Advanced Editing

In this chapter, we are taking a step forward from basic zsh usage and diving into the more advanced features of the command line. We will be getting close and personal with the zsh line editor, understanding how it works and why zsh needs it's very own input editor. We will discover new ways of accessing and tapping into the shell's history and learn some new command line editing tricks in order to speed up most of our regular tasks and avoid repeating ourselves to boredom. Finally, we will discover that there's really no need to be limited to a single line of text while using zsh.

# Zsh line editor

In the previous chapter, we learned how to access the shell's history and how to use some special escape sequences in order to access its records. Nevertheless, we assumed that the only way for us to review previous history entries was by using the arrow-up and down keys on the keyboard and loop through them sequentially. Well, as you can imagine, it's time we got acquainted with another of zsh's great features: the zsh line editor.

Unlike other shells—I'm looking at you, Bash—zsh does not depend on GNU's `readline` library, rolling instead with its own version of a command line editor that boasts most of the bells and whistles you'd expect to find in a full-fledged application. The zsh line editor, or ZLE in short, allows you to define your own key bindings (a combination of key presses) and set of custom keymaps (collections of key bindings) in addition to extending predefined entries. ZLE is also a key module of zsh, and is present in any interactive shell you use. Luckily, zsh is smart enough to know when not to load ZLE, thus avoiding extra resources if ZLE is not required.

## Getting to know ZLE

By now, you have been using zsh long enough to notice that some things just seem odd; like when you press a key, say *PageUp*, you are bound to see some arcane glyphs, same as trying to use the *Ctrl* + left-arrow shortcut to move the cursor between words. As it stands, ZLE is the one in charge of knowing what these symbols mean and what behavior is linked to them, a task we need to set up via key bindings. We can even group our collection of keybinds under the same name and use different collections for altogether different purposes such as *Home* to move to the beginning of the line when editing commands or selecting the first entry when searching through history. But first, let's take advantage of what's already defined in a default installation of zsh and the vanilla ZLE.

## Working with keymaps

On its own, ZLE comes with some handy bindings in order to cater to Emacs and vi users, some of the most popular editors out there. ZLE supports both vi *insert* and *read* modes, but defaults to Emacs as this seems to be the most user-friendly mapping for new users.

You can access it at any time by typing `bindkey` `-e` in the command line. We will be using the Emacs keybinds throughout this book, but feel free to roll with the vi mode if you feel more comfortable with it. You can always go back to Emacs mode by typing `bindkey` `-e` into your terminal. Whatever you choose, keep in mind that ZLE works only in interactive shell sessions, and that you will need to add your different configuration entries and bindings to your `.zshrc` file as they will be needed to be set for each session.

### Note

Zsh relies on your environment variables `$EDITOR` and `$VISUAL` in order to guess—make an educated guess, that is—which keybind it will default ZLE to. However, note that names such as `vile`, which contain the string `vi`, will trigger the use of vi keymap. You can add your own safety net of sorts, simply by adding `bindkey -e` in your `.zshrc` file to avoid possible conflicts and explicitly setting the layout.

For example, in order to default each new session to the Emacs mode, open up your `.zshrc` and append the following line:

```
bindkey -e

```

Having a default set in your startup files does not mean you have to commit to it at all times though. You can switch between vi and Emacs modes respectively, simply by typing the following line:

```
% bindkey -e

```

or

```
% bindkey -v

```

By using the `e` or `v` options, you are telling `bindkey` to link the provided `emacs` or `viins` keymaps to the `main` alias, which in turn gets loaded by default during startup. If anything goes awry, ZLE will default to `.safe`, which is a very constrained mode that provides you with the bare essentials. In such cases, your best shot at jumping out of the frying pan is by typing things such as `bindkey -e` and pressing *return* in order to switch keybinds. As you might expect then, using `.safe` spells trouble with your configuration and thus, is a binding you really don't want to see that often.

### Note

As vi users might expect, zsh provides two keymaps for vi: `viins` and `vicmd`. Be careful when tinkering with those though, as defaulting to `vicmd` will leave you without the ability to insert any kind of text.

## Basic editing

Now that we have set our default key mapping to Emacs, we can start discussing some of its more interesting features such as keyboard shortcuts that speed up your tasks.

The following table contains some useful Emacs mappings:

| *Ctrl* + *A* | Moves the cursor to the beginning of the line |
| *Ctrl* + *E* | Moves the cursor to the end of the line |
| *Ctrl* + *W* | Deletes the whole word backwards from the cursor location |
| *Esc* + *B* | Moves the cursor backwards one word |
| *Esc* + *F* | Moves the cursor forward one word |
| *Ctrl* + *D* | Deletes a character (moves forward) / lists completions / logs out |
| *Ctrl* + *U* | Deletes the whole line |
| *Ctrl* + *K* | Kills (or deletes) until the end of the line |
| *Esc* + *D* | Deletes one word on the right of the cursor |
| *Esc* + *Backspace* | Deletes one word on the right of the cursor |
| *Ctrl* + *Y* | Yanks the last killed word |
| *Esc* + *Y* | Switches the last yanked word |
| *Ctrl* + *T* | Transposes two characters |
| *Esc* + *T* | Transposes two words |
| *Ctrl* + *R* | Incremental search backwards |
| *Ctrl* + *S* | Incremental search forwards (automatically enables `NO_FLOW_CONTROL` option) |

### Note

Depending on your keyboard and input configuration, you could replace the *Esc* + button sequences with what is commonly known as the Meta key. This is usually mapped to the *Alt* key; however, we'll refer to these kinds of mappings with the *Esc* + sequences throughout this text, since they sport the same behavior and are arguably more portable.

### Going back and forth with words

The *Esc* + *B* and *Esc* + *F* bindings are tightly related to the `WORDCHARS` shell variable. This is zsh's way of knowing where any given word begins, although the definition of "word" might be rather peculiar for those coming from other shells. Particularly, the `WORDCHARS` shell variable defaults to… well, see it for yourself:

```
% echo $WORDCHARS
> *?_-.[]~=/&;!#$%^(){}<>

```

See those symbols? These are also considered as part of any given word (besides alphanumeric characters, that is). What's important to keep in mind here is the rather bipolar behavior of the shell; a character is either part of a word, or it isn't. Keep this in mind when using sequences such as *Esc* + *B* or *Esc* + *F*, and remember you can always override the `WORDCHARS` definition in those rare occasions where it might be required.

### Yanking and transposing text

You might have noticed the terms *yanking* and *transposing* in the shortcuts table and immediately addressed your thoughts with a healthy dose of what? So let's expand a bit more on that.

Transposing (*Ctrl* + *T*) might be a fancy name, but rest assured its functionality is nowhere near as complicated to understand as it sounds. Put simply, transposing a character will swap its place with the one immediately following it on the right, making it march valiantly towards the end of the line, one place at a time. Once there, it'll only swap positions with the character immediately before it. This might be a bit confusing, so let's get going with an example as follows:

```
% echo bca
> bca

```

That's not right. Let's edit our previous history entry:

```
% echo bca

```

Now move your prompt on top of `a`—the more straightforward way of doing this is by hitting the end-of-the line shortcut, *Ctrl* + *E*—and hit the transpose shortcut, *Ctrl* + *T*.

```
% echo bac

```

`a` and `c` switched places. Progress! Now go back one char to the left, placing your cursor on top of `a` again and, again hit the transpose shortcut.

```
% echo abc

```

Success! As we'll see in [Chapter 5](ch05.html "Chapter 5. Completion"), *Completion*, automatic completion will amend most of these silly mistakes; however, transposing comes in really handy on those occasions when you mistype things like parameter flags or URLs.

```
% git psuh origin master

```

A mistyped `git push` sentence can be easily fixed by simply navigating to `u` in `psuh` and hitting transpose.

```
% git push origin master

```

The same rules apply to the word transposing mechanism (*Esc* + *T*). The only difference, as you might have guessed already, is that it works with whole words instead of just chars.

As the old saying goes, actions speak louder than words, so the following is another example, this time by transposing words:

```
% echo 'world hello,'

```

Whoops! Got that completely backwards, time for some *Esc* + *T*. Put your prompt's cursor right on top of `hello` and hit the transpose shortcut.

```
% echo 'hello, world'

```

Sure enough, this will give the *Backspace* key a much-deserved vacation.

Yanking seems a bit harder to explain, but basically boils down to inserting a word you previously deleted by any of the kill shortcuts (*Ctrl* + *W*, *Ctrl* + *U*, *Ctrl* + *K*, *Esc* + *D*, *Esc* + *Backspace*). It works as follows:

Start typing your command.

```
% echo world hello

```

Realize you made a mistake, and kill the offending part. In this example, we use *Esc* + *Backspace* to delete the `hello` string.

```
% echo world _

```

Now move the cursor one word backwards, using the *Esc* + *B* bind.

```
% echo _world

```

And yank the `hello` string into the line by pressing *Ctrl* + *Y* (note that in this particular case, you will need to add an extra space between both the words and the `_` character is there to show where the prompt cursor should be).

```
% echo hello_world

```

After using the *Ctrl* + *Y* shortcut for yanking, you can call the *Esc* + *Y* shortcut to swap between previously deleted words. The shell you see retains up to 10 deleted words in memory, in case you need to use them again. This sort of "deleted words clipboard" is popularly known as the kill ring due to its behavior—you will swap each of the killed words up to the last, and then start again from the very first by repeatedly pressing *Esc* + *Y*. However, note that pressing *Ctrl* + *Y* again will only insert a new previously yanked word.

## Revisiting history

As you might have noticed in the Emacs shortcuts table, there are quite a few shortcuts we can use to work with history. So let's put ZLE to better use and build on the *History expansion* section from [Chapter 2](ch02.html "Chapter 2. Alias and History"), *Alias and History*, with our newly learned bindings.

Turns out we can use *Esc* + *<* to go to the very beginning of our history file, that is, the first entry of our log. Likewise, pressing *Esc* + *>* will deliver us to the end of the history file. However, that's hardly convenient for larger history logs. What we really need is to perform an incremental search. *Ctrl* + *R* is the default provided mechanism in zsh, and this will show us a prompt in which we can type to use as an immediate search filter. The more you type, the more precise the match is.

```
% # press Ctrl + R
bck-i-search: _

```

Start typing and once you have found the history entry you were looking for, you can either go ahead and press *return* to execute it, or the left-arrow/right-arrow key to edit the selected entry. You can exit this mode at any time by pressing *Ctrl* + *G*.

### Tip

The incremental search mode has its own keymap, conveniently called `isearch`.

It's very likely that your terminal is set to use the *Ctrl* + *Q* and *Ctrl* + *S* combinations for flow control, respectively stopping and resuming any output to the terminal. In order to avoid overlapping the default `history-search-forward` binding (also *Ctrl* + *S*), zsh offers the `NO_FLOW_CONTROL` option, which can be set in your startup files.

```
setopt NO_FLOW_CONTROL

```

This will safely disable such behavior within the shell (other programs can depend on flow control normally) and thus, is the recommended way of using *Ctrl* + *S*.

# Advanced editing

So far we have discovered our way around the command line and started to get the hang of ZLE. It's time we kick it up a notch though, so we can see what the line editor is really capable of.

## ZLE-related options

This chapter wouldn't be complete without some options for us to tinker with now, would it? The following are some things to try if you are looking to modify ZLE's default behavior:

*   `NO_BEEP`: This option skips beeping on errors.
*   `OVERSTRIKE`: This defaults the editor to the insert mode. The way it works is that each new character replaces the one to the immediate right, instead of displacing it one position to the right as default.
*   `SINGLELINEZLE`: It turns off multiline editing. No, I'm not on drugs. This could be used as a reminder of darker times.

Sprinkle some of these on your startup files (namely, `.zshrc`) and you'll be all set.

## Defining your own keymaps

Besides the Emacs and vi mode-setting options, the `bindkey` built-in allows you to create your own keymaps and alias them by using a couple of simple options. Namely, the `-N` flag will let you define a new keymap on the fly.

```
% bindkey -N newmap # this creates a keybind named 'newmap'

```

Or even create one based on an existing one.

```
% bindkey -N mycoolmap emacs # this creates a new keymap based off the existing 'emacs'

```

You can then alias your new keymap with the `-A` option by simply issuing the following command:

```
% bindkey -A mycoolmap mymacs # this creates an alias 'mymacs' for 'mycoolmap'

```

Creating the alias `mymacs` for the existing `mycoolmap` keybind will allow you to eventually use `bindkey -D mycoolmap` to delete it without the fear of losing your settings. Turns out that both aliases are treated as separate keybinds; thus, deleting one does not affect the other. This proves useful during the time you are experimenting with bindings and wish to start from scratch, or just wish to have a backup of sorts for when things go awry. Be careful when naming your aliases though, as any existing keybind will be immediately replaced by the new alias if their names are the same!

### Note

You should avoid naming your own keymaps starting with the dot `.` character as future editions of zsh might eventually ship with conflicting namespaces.

The `bindkey` command also has quite a few other options at its disposal. Of particular interest when populating your startup files are the listing options. Namely, `l` and `L` allow you to list the available keymaps in different formats. By typing `bindkey -l`, you can quickly have a look at the currently available keymaps, while issuing `bindkey -lL` will format the output as a series of the `bindkey` commands.

```
% bindkey -lL
> bindkey -N command
 bindkey -N emacs
 bindkey -N isearch
 bindkey -N listscroll
 bindkey -A emacs main
 bindkey -N menuselect
 bindkey -N vicmd
 bindkey -N viins

```

You can also use this option in order to check if a particular keymap is a link:

```
% bindkey -lL mymacs
> bindkey -A mycoolmap mymacs

```

This tells you that, as expected, `mymacs` is an alias for the `mycoolmap` keymap we defined earlier on. By using the `-lL` option to check the `main` alias, you have a practical way of determining the keymap currently in use.

```
% bindkey -lL main
> bindkey -A emacs main

```

Finally, you can use the `-L` option to have a list of all your current bindings, including those of a built-in keymap, formatted in a way you can use within your scripts:

```
% bindkey -L
 bindkey "^@" set-mark-command
 bindkey "^A" beginning-of-line
 bindkey "^B" backward-char
 bindkey "^D" delete-char-or-list
 bindkey "^E" end-of-line
 bindkey "^F" forward-char
 bindkey "^G" send-break
 bindkey "^H" backward-delete-char
 # [...] large list of bindings omitted
 bindkey -R "\M-^@"-"\M-^?" self-insert

```

Just copy and paste the output into your startup files and you have the foundation for your custom keymap. It's just a matter of replacing the action or shortcut keys with something that better suits your needs, and you are done. Handy, isn't it?

### Tip

You can use the `read` utility in order to figure out the actual escape sequence your terminal emulator is sending to the shell; just call `read` and then input the sequence you want to try out. For example, the following is what *Ctrl* + back-arrow is sending on my system:

```
% read
> ^[[1;5D
```

Some keys such as *Backspace* might require you to use the `-k` option, which allows you to specify the number of characters to read. Used by itself, it'll default to one.

```
% read -k
```

Now (press *Backspace*.)

```
^?
% # and you are back to the prompt
```

Keep in mind that you can exit the `read` command at any time by pressing *Ctrl* + *C*.

Emacs users will find themselves at home with the *Esc* + *X* sequence. By pressing *Esc* followed by the key *X*, ZLE greets you with an `execute` prompt. You can then start typing in your commands and even use the *Tab* key for auto-completion help. For example:

```
# type in "hello" and navigate to the beginning of the line (Ctrl + A) followed by Esc + X
% _hello
execute:
# ZLE waits for your command, type `ca` and press Tab key:
% _hello
execute: ca

% _hello
execute: capitalize-word
# now press return and watch how the command is applied

% Hello

```

The reason we used *Ctrl* + *A* is for the prompt to be at the beginning of the line, just before the rest of the string.

### Tip

Remember that you can exit the `execute` prompt at any time by using the *Ctrl* + *G* sequence.

As the astute reader might have noticed, there are quite a few ways of achieving the same behavior, but that's partially missing the point of the `execute` sequence. It is there simply to allow you to do things you would normally not do (either because of an awkward shortcut or lack of muscle memory); execute it and its completion mechanism will make recalling commands a snap.

In the same vein as `execute`, `where-is`—which is unbound to any sequence by default—will show you how to perform any given command. Just call `execute`, type `where-is` (you can use Tab for completion just as before) and press *return*. This time you will be greeted with the `Where is:` prompt, where you can also use completion to list any command you are after. Press *return* for ZLE to show you the sequence bound to the said command. For example, we can use `where-is` to find an alternative shortcut to our `capitalize-word` example as follows:

```
% # enter where-is mode via Esc + X
> Where is: capitalize-word
> capitalize-word is on "^[C" "^[c"

```

Well, look at that. Turns out we can capitalize the word immediately after the prompt by using the *Esc* + *C* combination.

# Don't call them widgets

There comes a time in the life of any eager zsh learner to talk about widgets. It's time you and me had that talk already.

Ever wondered how all those keybindings and special actions are put together and work marvelously? Well, we have widgets to thank for that. See, zsh likes to delegate responsibilities whenever it can, and widgets are a prime example of that; instead of having to deal with handling every little action performed by key sequences (similar to those defined in your keymaps), zsh relies on widgets to do the actual work. Think of them as small functions designed to carry out a simple mission. I, on the other hand, like to think of them as those sneaky gnomes that make magic happen in the kitchen whenever I'm not around.

ZLE comes with quite a few built-in widgets, each boasting two names, a vanilla name and a hidden name, which is simply defined as the normal name and preceded by a dot `.` character. Hidden names are there just to signal that they can't be rebound to a different widget (thus creating a backup copy that's always available in case your keybind definitions go awry).

As you might have guessed, that's not the whole deal; widgets can be user-defined or defined by other modules (such as ZLE or the built-in FTP client, `zftp`).

## Defining your own widgets

Defining your own widgets doesn't get more complicated than calling `zle -N` with your widget's name on it.

```
autoload -Uz tetris
zle -N tetris
bindkey '\et' tetris
```

The previous example, a slight variation from one of the suggestions available at the zsh wiki site ([http://zshwiki.org](http://zshwiki.org)), binds the *Esc* + *T* combination to the built-in tetris module, so you can spend those idle times on the command line a bit more entertained.

Let's go over it, line by line:

```
autoload -Uz tetris
```

This is the good old `autoload` module, which handles the loading of different modules and functions across the shell. In this particular case, we're importing the `tetris` module for later use.

```
zle -N tetris
```

This is where the magic actually happens; we're defining the new widget by calling ZLE with the `-N` option and telling it that the name for our new widget is `tetris`.

### Note

Keep in mind that hidden names are special for widgets, so refrain from using names starting with a dot (`.)`.

We wrap up the definition simply by binding our newly defined widget to the *Esc* + *T* shortcut on the keyboard:

```
bindkey '\et' tetris

```

Notice that the bold **tetris** call there refers to the widget we defined and not the actual `tetris` module.

Now, to actually see this in action, you'll have to either add it to your `.zshrc` file or save it as a separate file and source it from `.zshrc`, just as we've done before. So go ahead and save this as `.zsh_tetris` in your `$HOME` folder, and source it from `.zshrc` by adding the following line:

```
source .zsh_tetris
```

Now go ahead and type *Esc* + *T* to enjoy your new widget.

![Defining your own widgets](img/2937OS_03_01.jpg)

Just some Tetris. Yes, I'm rusty.

### Special variables

Some special variables that are available in ZLE should come in handy when defining your own widgets for editing and/or manipulating the command line.

The following list contains some of the most commonly used references:

*   `CURSOR`: This is the current position of the cursor on the command line.
*   `BUFFER`: This contains the current editing buffer and can span multiple lines.
*   `LBUFFER`/`RBUFFER`: These are the contents to the left and right of the current cursor, respectively. They too can span multiple lines.
*   `PREBUFFER`: This contains the buffer already read when editing a continuation line.
*   `WIDGET`: This gives the name of the widget currently in use by the editor.

By using these variables you can, for example, know precisely which character is currently under the cursor by simply using the `${BUFFER[CURSOR]}` expression. This might as well read as "the value of the `BUFFER` array for the `CURSOR` position" (remember, `CURSOR` is just a number that tells which column the prompt is at).

## Your first function

You can achieve even more complex behavior by defining your own functions. Each time the widget is executed, it'll call the corresponding function. Let's kick it up a notch with our second widget.

For the following example, we'll work with a variant of the excellent `rationalize-dot` widget, as presented on the ZSH-LOVERS' manpage ([http://grml.org/zsh/zsh-lovers.html](http://grml.org/zsh/zsh-lovers.html)):

```
function rationalize-dot {
  if [[ $LBUFFER = *.. ]]; then
    LBUFFER+=/..
  else
    LBUFFER+=.
  fi
}
zle -N rationalize-dot
bindkey . rationalize-dot
```

And now let's go ahead and go over it line by line.

Firstly, we're defining our own function here, called `rationalize-dot`. The way we declare a function is simply a matter of giving it a special name, followed by parentheses as follows:

```
my_function() {
    my_code
}
```

The curly braces `{}` you see there are the delimiters of the function body; whatever lays between them is considered part of the function, just like the `my_code` stub in the preceding example.

Alternatively, you can also define functions using the reserved keyword `function` and using a slight variation of the previous syntax as follows:

```
    function my_function {
        my_code
    }
```

As you can see, we trade the parentheses for the preceding function keyword. Otherwise, both syntaxes represent the same thing and are interchangeable. So pick whatever floats your boat.

Likewise, calling a function doesn't get any more complicated than explicitly writing its name; `my_function`, in this particular case.

Back to the `rationalize-dot` example, the second line there is an `if` statement, the most basic control flow mechanism provided by the shell. When used in its full glory, an `if` statement will resemble the following:

```
if condition; then
    my_code
elif another_condition; then
    more_code
else
    even_more_code
fi

```

In its most basic form, `if` statements test for a Boolean condition, an expression or command that resolves as either true or false (or has an exit status to indicate this), and takes action accordingly. Whatever is not suitable for the first clause, the `else` part will gladly take care of as follows:

```
if condition; then
    do_a_barrel_roll
else
    echo "can't do it"
fi

```

### Tip

Notice the `fi` at the end? Think of the first `if` as an opening brace `{` character, and the `fi` as the closing one `}`.

The previous sample will test the condition `condition`, if it evaluates to something that is true, our mock function will call the `do_a_barrel_roll` code. If `condition` is not true (what is popularly known as false), then the `else` block gets called, and dutifully issues an `echo "can't do it"` command.

The `elif` statement simply means "else, if" and is used to evaluate further conditions. You can add as many `elif` clauses as the options you have, but be careful when traversing down that road; neat code becomes wild spaghetti in a matter of keystrokes if not properly tamed.

In the `rationalize-dot` example, the `if` statement tests whether the `LBUFFER` variable matches the expression `*..`, which actually means "has the user typed anything followed by two periods?". If that's the case, then append a `/..` expression to the buffer variable. Otherwise, just let the `else` statement handle it.

As per the `else` block, it'll just add an actual period to the buffer:

```
else
    LBUFFER+=.
fi
```

This might not seem a logical decision at first, until we move into the following lines:

```
zle -N rationalize-dot
bindkey . rationalize-dot
```

The first one is the standard widget declaration we've seen before, but the binding immediately after it is what makes the `rationalize-dot` function require the `else` statement to add a period. As it's called on each dot press (the keybinding it's being assigned), this requires you to behave as an actual period key if the user hasn't typed anything yet.

As before, you can go ahead and add this to your `.zshrc` (or any other module that gets sourced by it) and take it out for a spin; just type `...` and see what happens after that third dot gets pressed.

As we'll see later in [Chapter 5](ch05.html "Chapter 5. Completion"), *Completion*, you can also let the shell source functions automatically by extending or adding them to your `$fpath` variable.

This is particularly useful in combination with the `cd` command and an unhealthy dose of nested folders.

Want to go further? You'll find tons of predefined built-in widgets for customizing your keybindings in the `zshzle(1)` manpage's *STANDARD WIDGETS* section. Just type `man zshzle` to get started.

# Working with regions

Continuing the legacy of Emacs' inherited behavior, you can set regions in the command line by holding *Ctrl* and pressing the Space bar. This will trigger a region selection mechanism that you can expand with the arrow keys, which works just as if you were clicking and dragging your mouse to highlight text.

So, why bother with regions? You could, for example, mark a region via the *Ctrl* + Space bar sequence and then perform a command on top of it (similar to `capitalize-word` we saw earlier), or even mix-in the previously mentioned `execute-command` to call a function that has no keybind. Overall, these few niceties straight from Emacs give ZLE (and of course, zsh) the versatility to behave almost like a full-fledged editor.

## Multiline editing

At this point, it should be no surprise to learn that zsh is smart enough to notice when you aren't done with a line. Unlike most other shells though, zsh is also capable of suggesting what might be missing, or even allowing you to use multiple rows of lines for entering your commands. Unlike traditional continuation where you put a `\` character at the end of the line and press *return* to continue on the one immediately below, ZLE will greet you with the `$PS2` prompt and add more of context information.

On most flavors of Bourne-derived shells, you can use the following line:

```
% ls \

```

Press *return* (notice there is absolutely nothing after the `\` char).

```
> -a

```

Press *return* again, and it'll work just like the `ls -a` command. Zsh will give you a bit more context as follows:

```
% echo " # press return immediately after the double quotes
dquote> _

```

The `$PS2` prompt (the alternative/second prompt) is called in order to signal that the shell is waiting for the rest of the double-quote assignment. Go ahead and wrap it up as follows:

```
dquote> $HOME" # press return here
> /home/gfestari

```

There's more to multiline editing than alternative prompts though. You can use the *Esc* + *return* shortcut to add a new continuation line:

```
% echo hello world # press Esc + return
echo goodbye world

```

And press *return* to see both the lines execute sequentially, just as though it were a script. Keep in mind that you are not limited to just two lines and can add as many lines as you want.

This sorcery owes its powers to the `self-insert-unmeta` command, whose job consists simply of inserting a carriage return character into the line. So now you know that each time you press *Esc* + *return*, you are actually using a shortcut to the `self-insert-unmeta` command.

Besides the obvious "being different" feeling, what's really convenient about the *Esc* + *return* method is that you can move across lines as you please by using the arrow keys. To top it off, each multiline entry is treated as a whole line. Just press the up-arrow and you will see the block you previously typed come back to life for you to edit. While we are at it, I'd like you to meet the `push-line-or-edit` command, which allows you to convert a previously typed block of lines into a single block whenever you are on a continuation (otherwise it'll behave like a normal push-line command). It works more or less as follows:

Start entering your function in the command line, pressing *return* after the first `if` line:

```
% if [[ true = false ]]; then # press return here
then> echo _

```

And stop right there. Realize you have made a terrible mistake with the condition clause of the `if` statement (apart from the extremely simple logic… but hey, this is an example). Unfortunately, you can't scroll back to the previous line with the up-arrow button as you have already pressed *return* and that would trigger the history search behavior, so what's next? Well, `push-line-or-edit`, of course. Hit *Esc* + *X* in order to execute a command, and type `push-line-or-edit` (you can use the *Tab* key for completion) and press *return*.

The prompt will change to a traditional one (ditching the `then>` indicator from the continuation line), and you will have a new buffer filled with all your previously typed lines which, of course, you can edit at will as follows:

```
% if [[ true = false ]]; then
echo_

```

Seeing how much better a push-line `push-line-or-edit` is, it's of course advisable to bind it to the default `push-line` shortcut, either `^q` or `\eq`:

```
bindkey '^Q' push-line-or-edit
bindkey '\eQ' push-line-or-edit

```

And now you can either use the *Ctrl* + *Q* or *Esc* + *Q* shortcuts to edit a whole block as if it were a single line. As with the `history-search-forward` binding we saw earlier (which defaulted to *Ctrl* + *S*), *Ctrl* + *Q* will require the `NO_FLOW_CONTROL` option to be set so as not to conflict with the terminal driver's behavior.

This whole thing started with `push-line-or-edit`,so it seems fair we got to discuss the actual `push-line` bit. This will be the default behavior when you are not on a continuation line. Just type your commands as usual, but do not press *return*:

```
% ls -a

```

Realize you are in the wrong directory, call our newly bound `push-line-or-edit` command via *Ctrl* + *Q*, and the prompt will be cleared for you as follows:

```
# push-line-or-edit
% _

```

Now use `cd` to go to the folder you were trying to list, and watch the buffer come back to life:

```
% cd myfolder
myfolder % ls -a

```

As soon as you execute a line, the prompt gets populated with the line you were editing prior to calling `push-line`.

## Putting it all together

As we saw earlier, a peculiar aspect of ZLE is that it has access to the shell's history, which of course means we can use some of the niceties we have learned in order to further improve how we work with it.

A neat way of taking advantage of the up/down arrow keys is via the `history-beginning-search` commands. We could define our own mappings in order to add some extra kick to the default behavior as follows:

```
bindkey '\e[A' history-beginning-search-backward
bindkey '\e[B' history-beginning-search-forward

```

Note that the `\e` escape sequence could also be replaced by `^[`, thus leaving the bindings as `^[[A` and `^[[B` respectively.

Now, if you have an empty prompt and press the up-arrow key, it'll work by retrieving the most recent entry in the history as usual. However, as soon as you type something and press the up arrow key, it'll autocomplete with your most recent entry that matches with what's typed.

As an example, type the following pressing *return* after each line:

```
% echo hello world
% ls
% echo bye world

```

Now go ahead and press the up-arrow key. The natural backwards-scrolling sequence should be as follows:

```
> echo bye world
> ls
> echo hello world

```

Press *Ctrl* + *G* to exit the search mode. Now type `ec` and press the up arrow key:

```
% ec
> echo bye world

```

This comes in really handy during those times when you forget about a line mid-sentence and don't want to perform a search or discard the current line. Just remember to add your bindings into your startup files if you want to keep these kinds of changes between sessions!

# Summary

In this chapter we took a deep dive into what goes on between the prompt and the shell by the time you press *return*. We discovered some new tricks to work with history and tamed the default shortcuts by creating our own keymaps and bindings. As if this wasn't enough, you now know we are no longer limited to just working with one line, and that mistakes and distractions can easily be solved by a couple of keystrokes without us re-typing the whole line.

Okay, I'll admit it, we have been pretty busy in this chapter. So here's a chance to catch your breath while we go over everything we've covered in this sprint. What we have done is:

*   Learned that zsh is made out of various modules, and got acquainted with ZLE
*   Used key maps for editing text and learned about various shortcuts to improve our productivity in the command line
*   Defined our own custom keymaps and worked with various regions and multiline prompts
*   Learned about widgets, special functions that carry out every little task in the editor
*   Written our first sample widgets to further extend the functionality of the editor and improve our shell experience
*   Learned about functions and control flow via the `if` statements
*   Finally, we learned that both modules and functions have special access to different parts of the shell, and we can do things such as hooking up ZLE widgets to keybindings in order to search the history.

Not a bad day I'd say. Now, let's head into the next chapter, where we'll learn about Globbing and filename generation, another of those features where zsh really shines. If you thought you had learned how to type less with ZLE in this chapter, wait until you see braces and qualifiers in action. Keep those elbows greased and that confidence up though, as there's still much more waiting for us to right around the corner.