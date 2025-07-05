# Chapter 5. Completion

This is what most users switch to zsh for: completion. In this chapter, we'll meet one of the best features of zsh: `compsys`. Known as "the new" completion mechanism, this chapter focuses on its various functions and configuration. We will learn to tweak the completion behavior so that it's no longer restricted to filenames and bump it up using styles and our own functions. When we're done, you should be able to read most zsh scripts as well as tweak many of the existing functions.

# Getting started with completion

Nobody really likes to type boring filenames, and that's what got completion started back in the day—type a few letters of a filename, press *Tab*, and the shell will do the rest for you. Zsh goes the extra mile though and actually allows you to complete almost anything. By default, the *Tab* key is bound to a completion command in zsh.

Like Bash, zsh defaults to filename completion. Unlike anything else, however, zsh can enable the completion for practically everything that dares to rear its head in the command line—paths, external and built-in commands, aliases, functions, and options; you name it. And even if you can't name it, you can program it, as we will learn shortly.

Originally, zsh used a built-in module with a special syntax in order to provide completion. Luckily for us, this was eventually replaced by an even simpler mechanism. We'll focus on the new completion system that is entirely based on shell functions.

Go ahead and pop open that `.zshrc` file of yours, and add the following in order to activate shell completion:

```
autoload -U compinit
compinit
```

This addition will make the shell load the completion system and start it automatically. The `-U` flag tells the shell to avoid expanding any aliases. This will make double tapping *Tab* trigger the completion mode.

### Note

`compinit` is an essential part of the completion system. As such, you won't be able to test anything from here on until you have updated and sourced your `.zshrc` file or at least run `autoload -U compinit && compinit` in your terminal.

Remember to source your files, and then let's go ahead and try our newly enabled completion. Type `ec` and press *Tab*:

```
% ec <Tab>
% echo 

```

The shell automatically completes the external command as `echo`. How nice of zsh, isn't it?

### Tip

As we have previously noted, zsh has two ways of performing completions in the command line. You can learn more about "the old way" of doing things by typing `man zshcompctl`, for academic purposes, of course.

Completion can also be applied to environment variables, for example:

```
% echo $HOM <Tab>
% echo $HOME

```

By default, zsh enables the `AUTO_LIST` option that handles the resolution of ambiguous matches, providing you with all the possible completions. To see this in action, let's go back to the previous example; only this time, we will make the completion less obvious by typing only `HO` as follows:

```
% echo $HO<Tab>
Completing parameter
HOME  HOST

```

The shell doesn't know for sure what we mean, so it presents us with a list of possible matches below the prompt instead. This list will be updated if the criteria changes, so we need to only worry about hitting the *Tab* key.

Now, let's try option completion with `ls`, as follows:

```
% ls -<Tab>

```

The following screenshot shows you how completion is triggered for the `ls` command:

![Getting started with completion](img/2937OS_05_01.jpg)

Menu selection in action

Seeing that there are actually a couple of viable options to pick from, zsh presents you with a menu that you can cycle through by repeatedly hitting the *Tab* key or using the arrow keys.

Finally, you can also use completion for expanding commands as follows:

```
% echo `which zsh`<Tab>
% echo /usr/local/bin/zsh

```

You can see where this is going—completion is awesome enough for us to want it to be applied everywhere, and not just in the word that's being typed. Before we start writing our own functions however, we will take a look at zsh styles, the options by which we can configure the behavior of the `zstyle` built-in.

## Getting assertive with zstyle

Unlike the shell options that we have been setting—and unsetting—throughout this book, zstyles demand a bit more complex syntax as a trade-off for enabling a context-sensitive completion.

Zstyles are defined via the `zstyle` keyword, followed by a colon-delimited list of arguments:

```
:completion:function:completer:command:argument:tag
```

The first argument, `completion`, is used for defining a context, as any given style could behave differently in different contexts. Nothing to write home about though, as we'll get to see in no time.

The second argument is the name of the style by which it will be referenced by the built-in. The remainder of the arguments are what give the style their unique behavior for completion.

Patterns make a comeback here as well, so you can use them as tokens for each of the subsequent arguments when defining a style. As usual, order matters when you want to define your styles, so try to put the less-specific or general-purpose styles at the bottom of your definitions, otherwise you'll end up overriding your more-specific functions.

The most general type of style you can define is `:completion:*`, which will apply to almost anything, so be careful when ordering something that resembles it.

As you might have imagined, zsh has a few tricks up its sleeve, such as being capable of displaying some useful messages with the list of matches. For this to work though, we need to enable the following style:

```
zstyle ':completion:*' format %d
```

By adding this to your `.zshrc` file, you can now get a bit more information whenever zsh is performing a completion. For example:

```
% true<Tab>
no argument or option

```

The astute reader might have noticed the `%d` pattern lying within the style format. That's right, we can use the same escape sequences as that we used when defining our prompts.

### Tip

Tired of hearing beeps already? That's zsh's way of telling us that an ambiguous completion was attempted. You can put off this rather annoying attitude towards ambiguity by unsetting the `LIST_BEEP` option in your `.zshrc` file:

```
unsetopt LIST_BEEP
```

As we mentioned earlier, you can also narrow down the behavior of your styles to a more specific context. For example, you could use any of the following:

```
zstyle ':completion:*:descriptions' format '%B%d%b'
zstyle ':completion:*:messages' format %d
zstyle ':completion:*:warnings' format 'No matches for: %d'
```

This is just to set a custom pattern for the messages belonging to `warnings`, `messages`, and `descriptions` groups. As you can see, `warnings` will now be reported as `No matches for: <argument>`, which is a bit less dronish.

You could also add a little more flair to your results with something along the following lines:

```
zstyle ':completion:*' group-name ''
```

This will display all the different types of matches separately. If no tag or group is defined for a particular match, it'll get displayed under the `default` group.

### Tip

Did the menu selection tickle your fancy? Here's how we make it available for all of your matches:

```
zstyle ':completion:*' menu select=1
```

Getting comfortable with the styles? Glad to hear. As you can see from the examples, there's no arcane magic involved here—just some documentation and creativity to fill the gap between you and your custom styles.

# Command correction

Completion can also correct any misspelled commands that you might have typed. We'll use the following format for our style:

```
zstyle ':completion:*' completer _expand _complete _correct
```

And we'll test the autocorrect functionality with the following:

```
% prnti<Tab>
corrections (2 errors)
print   printf
original
prnti

```

Zstyle noticed that we misspelled `print` and is being quite verbose regarding this. Remember you can use the *Tab* key to cycle through the list of available options.

Alternatively, you could use the `correct` option if you want a more "hold me by the hand" approach. Specifically, this option will make zsh ask you for confirmation every time it suggests a correction:

```
% setopt correct
% prnti<Tab>
zsh: correct 'prnti' to 'print' [nyae]?

```

This peculiar `nyae` acronym stands for *No*, *Yes*, *Abort*, and *Edit*, and works in the following way:

*   `n`: This will force the shell to execute whatever you typed in the command line (`prnti` in this particular case).
*   `y`: This will execute the correction (effectively, changing `prnti` to **print** in this example).
*   `a`: This will abort and allow you to type a completely different command. Think of it as a panic button.
*   `e`: This will allow you to edit the current text in the command line. Use this for a more fine-grained control in case suggestions made by the shell are completely off.

What about command options? You know, those flags we pass around all the time? Well, turns out there is a style for that too. The following will make the commands show the descriptions for their options:

```
zstyle ':completion:*' verbose yes
```

These can be easily accounted for; now, go ahead and type the following:

```
% print -<Tab>

```

```
-- option --
-C  -- print arguments in specified number of columns
-D  -- substitute any arguments which are named directories using ~ notation
-N  -- print arguments separated and terminated by nulls
-O  -- sort arguments in descending order
(list goes on...)

```

Not too shabby, right? Remember how I mentioned we wouldn't be in such a dire need for manpages after we learned some styles? No? Well, we won't be in such... never mind.

## Completers

The third entry on the zstyle is reserved for completers. These are the functions that handle the different types of completions available. By default, the list of completers consists of a single function, `_complete`, but each member of the completers family will add its own unique behavior to your styles.

```
zstyle ':completion:*' completer _expand _complete _correct
```

Used in your `.zshrc` file, this completer will use globbing for expanding the input and match it against the `_complete` and `_correct` completers. The `_correct` completer is used here for correcting any typos and spelling mistakes. We're leaving it at the end of the argument list so that `_complete` takes precedence.

### Note

When used within a style, completer names omit the leading underscore:

```
zstyle ':completion::complete:*' use-cache on
```

This style configures the `_complete` completer by enabling a cache layer for any completions that require it, improving the overall responsiveness of such functions.

Similar to `_correct`, `_approximate` will carry out the same tasks with the added benefit of allowing a few extra characters to be misspelled at the cursor position. Notice that you will need to put `_approximate` before `_correct`, should you need to use both in your style.

As a function, zstyle also uses flags. Of particular interest to us is the `-e` option, which tells zstyle to evaluate the final string as an argument on each call. This allows us to use more dynamic styles such as the following:

```
# One error for every three characters
zstyle -e ':completion:*:approximate:*' max-errors 'reply=( $(( ($#PREFIX+$#SUFFIX)/3 )) numeric )'
```

This configures the `approximate` completer to evaluate the argument for the `max-errors` parameter dynamically, each time it is invoked. The `reply=( $(( ($#PREFIX+$#SUFFIX)/3 )) numeric )` string uses the `reply` hook for displaying the results within the line editor and sets its value as the expression, `(PREFIX + SUFFIX)/3`. This is our way of saying "one error for every three characters". Both `PREFIX` and `SUFFIX` are variables that contain the values before and after the cursor position, respectively.

## Ignoring matches

Sometimes, some matching suggestions jump out at you as being completely out of place. Luckily for us, the developers of zsh have included an `_ignore` completer.

Take the following directory tree as an example:

```
zsh
├── README.md
├── Completion/
├── Misc/
├── Scripts/
└── Util/
```

When working on any of the subdirectories mentioned previously—for example, the `Completion` folder—see what happens when we try to change directory, using `cd`, to another at the same level:

```
% cd ../ <Tab>
directory
Completion/
Misc/
Scripts/
Util/

```

Having the `Completion` mechanism display the folder we're currently located in is a bit awkward, and it makes the whole `cd` deal a bit pointless. In order to make the shell a bit more context-sensitive, we can alter the completion behavior for the `cd` command using the `ignore-parents`, `parent`, and `pwd` options:

```
zstyle ':completion:*:cd:*' ignore-parents parent pwd
```

The following will remove the respective matches from the completion results. Notice how `Completion` is now missing from the results:

```
% cd ../ <Tab>
directory
Misc/
Scripts/
Util/

```

While we're at it, you can use the following style to remove the trailing slash when using a directory as an argument:

```
zstyle ':completion:*' squeeze-slashes true
```

# Function definitions

Finally, we will turn our attention to `compsys`, zsh's completion system. This is one of the most complex parts of the shell for users and developers alike. Before we dive into `compsys`, however, we need to make a quick stop and meet an actual function in the wild.

### Tip

As usual, you can learn more about `compsys` via the manpages. Of particular interest are `man zshcompsys` and `man zshcompwid`.

Here's what one of these looks like:

```
hi() {
print 'Hello, world'
}
```

Here, we have defined the `hi` function, which is how we'll call it again later when we need it. This will, in turn, print `Hello, world` every time we use it. So let's get to it, shall we?

Open your terminal emulator of choice, and type the following (one line at a time):

```
% hi() {
function> print 'Hello, World!'
function> }

```

Notice how zsh realized this was indeed a function we were trying to define and immediately used the continuation prompt (`function>`), allowing you to continue working on it? How nice of zsh to wait for us until we properly close our curly braces.

Now, go ahead and test your first function:

```
% hi
Hello, World!

```

They grow so fast!

And, now for the sad part—this was defined for your current session only, just like when we defined aliases back in [Chapter 2](ch02.html "Chapter 2. Alias and History"), *Alias and History*, at the beginning of our zsh adventure. If you want `hi()`, or any other function to tag along in each interactive session of yours, you'll need to add it to your startup files.

A word of advice though: once you start with the completion and functions, these startup files will get pretty crowded. So, it's probably best that you start relocating your functions into a more comfy space like their own `.zsh_functions` file. Fret not, as the process is easy.

First, we create a hidden file; you can name it whatever you fancy, but we'll go with `.zsh_functions` (see the leading dot, so we can tell the system that it can hide it).

```
% touch ~/.zsh_functions

```

Once you have created the file in your `$HOME` directory, it's simply a matter of adding your functions in here. You can use your favorite editor; we'll just roll with `cat` here for convenience:

```
% cat >> ~/.zsh_functions
greet() {
 print 'Hello, World!'
}

```

Press *Ctrl* + *D* to close the file.

Now, as we learned previously, this wouldn't do anything by itself unless we source the file. And since sourcing the file manually in each session would be a pain in the neck, we just need to go a step further and add the `.zsh_functions` sourcing to our startup files. So, go ahead and open your `.zshrc` file, and add the following:

```
[[ -f ~/.zsh_functions ]] && source ~/.zsh_functions
```

This is a conditional statement. The double square braces (`[[`) shown here are known as the `test` command (or *new test* if you have been around the command line for a while), and they help you compare strings and test for file attributes. The `-f` switch is for regular files and succeeds only if the file exists. So we're literally trying to say "test whether the `~/.zsh_functions` file exists". If the test passes, the following part of the command will get chained and we'll finally source our functions file.

As a side note, this expression supports filename globbing, so all the tricks we learned in [Chapter 4](ch04.html "Chapter 4. Globbing"), *Globbing*, still apply here.

You can source as many files as you like with this same mechanism; just remember to add the line into your `.zshrc` file, and don't forget about the test fail-switch, which will avoid sourcing files that do not exist in the system (and of course, errors).

As always, you can scuba dive into the `test` command simply by typing `man [` in your terminal. For more details regarding the `[[` compound command, check the *CONDITIONAL EXPRESSIONS* section under the `zshmisc(1)` manual entry.

Okay, I hear you. So what do functions have to do with completion? Well, everything! See, `compsys` is entirely made out of functions: functions that will be called automatically whenever you hit the *Tab* key. The difference lies in how these set of functions use some other special commands to interact with our old pal, ZLE, in order to show the available completions. Don't worry though; contrary to popular belief, there's no arcane magic in here.

## The path of the function

So, functions. A truckload of them to be more precise (well, you be the judge of this). How does zsh know where to look? It is easier than it sounds; the shell will load anything that belongs to its function path or `$fpath`, a series of directories that contain the files with the functions required for completion. Go ahead and have a look at it: 

```
% print -l $fpath

```

All the directories that show up in your function path list will be scanned and loaded by the shell during startup, provided you call `compinit` first. So remember to call `autoload –U compinit` in your `.zshrc` file. Note, however, that call will load anything that resides in your `$fpath`. If you happen to have a special requirement for a single function, you could call it explicitly via `autoload`. If you save the previous function as a file named `_greet` and put it into one of the directories within your `$fpath`, you could then use the following inside your startup files for loading the function into the shell automatically:

```
% cat >> _greet
echo 'Hello world!'

autoload -Uz _greet

```

See that `-Uz` flag? The `-U` flag works by telling the shell to use the name `_greet` to refer to the function we just created, whereas the `-z` flag tells zsh to load the function in the native mode. Both `-U` and `-z` flags are always added implicitly whenever you call `autoload`, but I'm leaving it there for you to be aware of them.

Okay, so it's all fun and single-line functions until someone needs something a bit more complex. Single functions within a file will be loaded without any problem whatsoever. So, how do we use helper functions (auxiliary methods for our main functionality) in our files? The zsh way states that we should define a function and name it just like the file and call it in the last line of the file:

```
_greet() {
    echo "Hello, World!"
}

_meet() {
    _greet
    echo "Ohai there $@"
}

_meet "$@"
```

That last line in the file takes care of calling the function named `_foo` inside the file, and passing it the same arguments used. So if you called it `meet John`, the arguments will be passed to the `meet` function.

Save the file as `meet` (no extension) inside any of your `$fpath` folders; restart your shell and call the following:

```
% meet John
Hello, World!
Ohai there john

```

### Tip

**Extending your fpath**

If you don't want to be messing around with copies or links to your functions, you can easily extend `fpath` with more folders by setting the variable as follows:

```
fpath=(~/my_folder $fpath)
```

This will prepend the folder, `my_folder`, to the shell's `fpath`, effectively extending it with whatever lies inside your folders. This is particularly useful for those times when you lack the appropriate permissions on a given system. Note that we are using the absolute path to the folder.

So let's take a look at a formal completion function. Don't worry, we'll start with an easy one, such as `_md5sum`, which is typically located under your `$ZSH_INSTALL_DIR/functions/` folder. Here it lies in all its glory:

```
#compdef md5sum

_arguments -S \
  '(-b --binary)'{-b,--binary}'[read in binary mode]' \
  '(-c --check)'{-c,--check}'[read MD5 sums from the FILEs and check them]' \
  '(-t --text)'{-t,--text}'[read in text mode]' \
  '--status[no output, status code shows success]' \
  '(-w --warn)'{-w,--warn}'[warn about improperly formatted checksum lines]' \
  '--help[display help and exit]' \
  '--version[output version information and exit]' \
  '*:files:_files'
```

Go ahead and test this by typing `md5sum -` followed by pressing the *Tab* key, and you'll be prompted with the options from `arguments`.

Your very first line of code in any completion function must be the `#compdef` clause, followed by the name of the program to be completed by the function (`md5sum`, in this particular case).

Next up is a call to the internal `_arguments` function, which does the actual handling of the options to be formatted and displayed on screen. This function is typically used when specifying the completion of commands whose arguments follow standard Unix conventions in their options and arguments' lists. Using the `-S` option, we declare that no option will be completed after `--` shows up on the line. This is the delimiter used to end the parsing of the option, so this argument would be typically ignored unless we explicitly say otherwise.

If you look closely though, you'll notice that each of the argument entries (split into continuation lines via `\`) follows the same pattern:

```
'(optional exclusion list)'{options}'[help text in brackets]'
```

Note that the curly braces around the option and its verbose variant are there to group them together, otherwise they are optional.

The exclusion list works by explicitly telling zsh what should not be included in the results. In other words, whenever the `option` parameter is typed, hide all the other options from `(exclusions)`. Take for example the following line:

```
'(-t --text)'{-t,--text}'[read in text mode]'
```

If `-t` or `--text` appears in the command line, do not show the `-t` or `--text` options as completions.

This makes even more sense for commands such as `ln`, where you want to avoid offering some potentially misguiding options:

```
'(-L -P)-H[with -R, follow symlinks on the command line]'
```

Hide the options `-L` and `-P` if `-H` is being used; this is because both the options are used for "always follow symbolic links" and "never follow symbolic links", respectively.

Finally, there's the last line of the `_md5sum` function:

```
'*:files:_files'
```

This uses the `_files` helper function that is somewhat the standard tool for completing filenames. With this line, we make sure that filenames are completed even if no other options' flags are suggested.

Moreover, `_files` uses an additional function, `_path_files`, and passes its arguments to the latter. On its own, `_path_files` is the de facto function for completing filenames within the completion system. As if it wasn't enough, `_path_files` has some really handy tricks up its sleeve such as completion of partial paths, which enables things such as `/u/bi/zs` to be completed to `/usr/bin/zsh`.

Then, there are also helper functions such as `_call_program`, which are used to execute any kind of commands available to the system. A common practice when using `_call_program` is to redirect the standard error to `/dev/null` (this is a nice way of saying it's silencing any error-induced screams) and allows us to save the output of the command into a variable.

And that's all there's to it. Well, at least for getting started with the completion mechanism and custom-made functions. Although, on some occasions, getting your hands dirty and extending the completion system with your own functions will only get you so far, this quick fly-by should be enough to get you excited about the possibilities lying there. Again, it's advisable that you try not to reinvent the wheel—as we'll see in the next chapter, there are many other projects out there that can give you a nice boost in the completion department.

You can now go ahead and take a deep dive into the `functions` folder of your zsh installation to start getting familiarized with the thousands of lines of code there. Who knows? Perhaps the starting template for the next completion function is just waiting there for you.

# Summary

We are almost done with this adventure, and it seems you are now more ready than ever to start tackling major annoyances like your favorite program not having a set of completion definitions. Even better, you can tweak and improve the existing functionality, which otherwise would make your work really frustrating.

Besides writing your own functions, we also learned how to tweak the shell behavior and go a step above filename completion. With a bit of practice and further tweaking, you can now become a real speed demon of the command line. Best of all, it only takes a couple of *Tab* presses to get there.

Summing it up, here's what's covered in this chapter:

*   The types of completion available to zsh—zstyles and functions, which allow you to customize the behavior of the completion mechanism and extend its functionality
*   The different types of completers (particularly `correct`, `approximate` & `ignore`) and their role when defining zstyles
*   A few tips for creating and extending your our own completion functions

Okay then, before I get sentimental, we should hurry to the next chapter that has a few suggestions before we're done with this journey of ours.