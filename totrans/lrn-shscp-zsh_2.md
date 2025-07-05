# Chapter 2. Alias and History

In this chapter, we'll expand on the basics of zsh while focusing on aliases, one of the most time-saving features available. We'll take a closer look at how aliases work and learn to replace long, boring commands with our own short versions and automate the whole process within the startup files. We'll then move on to brace expansion, in order to avoid typing extra keystrokes whenever we can. Instead of typing the same things over again, we'll learn how to work with zsh's history and history expansion mechanisms and incorporate these new features into our workflow.

# Working with aliases

An *alias* is an alternative way of saying the same thing. Think of it as a nickname for your commands. Though, unlike the embarrassing nicknames that you might get after a party, the alias mechanism provided by your shell is a handy shortcut to a series of commands and options under a friendlier name. The whole point of an alias is to do more and, preferably, type less.

I bet I got your attention with that last "type less" part. Allow me to explain:

The `ls` command lists a directory's contents. A quick look at its manpage (`man ls`) tells us that there are quite a few options there:

```
ls -a # lists all files, even those hidden that start with a dot
ls -l # shows more information for each file, like size and permissions
```

Using aliases we can go ahead and do something like the following:

```
% alias la='ls -a'

```

### Note

No spaces are allowed around the equals (`=`) sign. If the right-hand side of the assignment (that is, the part that comes after the equals) contains spaces or tabs, then make sure you use quotation marks around it as follows:

```
% alias talk='echo "quack!"'
% talk
> quack!
```

Now guess what happens if you type `la`? Go ahead and try it. The shell reads your alias—`la` in this particular case—and expands it. The whole process is similar to looking up the meaning of a word in the dictionary. Although here, once it's been found, the meaning is executed.

We can do basically the same for the `-l` option:

```
% alias ll='ls -l'

```

Or even mix and match as shown in the following line:

```
% alias lla='ls -laF' 

```

That last snippet uses both the `l` and `a` flags together with `F`, meaning it behaves just the same as the `–la` switch, with the added option to format the output so as to easily tell files and folders apart.

### Tip

Aliases apply only to interactive shells. Your shell will disable all of your existing aliases if it's being run in the non-interactive mode. Keep this in mind when creating your scripts.

You have two ways of declaring an alias. The first one is straight from the command line, as we have been doing so far. This nets you an alias which you can use instantly; the downside is that changes are only present temporarily for the duration of your current session. Close your terminal emulator or log out of the system and it goes the way of the dodo. The basic syntax for declaring an alias is as follows:

```
alias [shortname]=<longname or command(s)>

```

You can use this approach for something you'll be typing a lot but won't come back to it later. Most of the time though, we'll need something a bit more resilient. Something we can use every time we work with the command line.

Enter the startup files; if you recall from the previous chapter, startup files are read every time the shell starts, and its configuration is loaded for the current session. Kind of what the doctor recommended.

Let's open up your `.zshrc` then, and add the aliases we've been working on so far:

```
# put this on your .zshrc
alias la='ls -aF'
alias ll='ls -lF'
alias lla='ls -laF'

```

Save your changes, `source` (or use its alias, the dot (`.`)) your file, and aliases will be set for you to use on every future session of the shell. Despite their different behavior, both the ways of alias declaration sport the same syntax.

## Quoting characters

Any given character can be quoted by adding a `\` character in front of it. This is particularly useful when dealing with "special characters" which have an additional meaning, such as `$`, and even an actual `\` character. Take, for example, the following `echo` sentence:

```
# this is wrong!
% echo 'that's a quoted sentence for you'
quote>

```

The shell prompt indicates it's waiting for a quote character to be (properly) closed. The problem here is that we did not properly escape the apostrophe on "that's":

```
% echo 'that\'s a quoted sentence for you'
> that's a quoted sentence for you

```

That's all nice and working, but what happens when we have a great number of escaping to do?

```
% echo 'Escaping single quotes like this \' with backslashes \\ is really tedious'
> Escaping single quotes like this ' with backslashes \ is really tedious

```

Luckily, zsh provides the `RCQUOTES` option as a workaround, which allows you to use double single quotes (`''`) for escaping:

```
% setopt rcquotes
% echo 'Look ma'' I''m escapin'' single quotes'
> Look ma' I'm escapin' single quotes

```

What about double quotes then? Well, these are truly special and out of the bunch, as they allow you to perform parameter and command substitution as we'll see in no time. What you have to remember when using double quotes is that either `"`, `\`, `$`, and `` ` `` characters need to be escaped with a backslash.

Let's give double quotes a try:

```
% echo "'echo \"\$HOME\"' will print out '$HOME'"
> 'echo "$HOME"' will print out '/Users/gfestari'

```

In the preceding example, the `$HOME` environment variable gets replaced by the actual value (`/Users/gfestari`) when the `$` character is not quoted.

You can also use backquote within double quotes for executing programs:

```
% echo "zshenv is located at: `locate zshenv`"
> zshenv is located at: /etc/zshenv

```

The shell will first execute `locate zshenv` as if it were any other command, and substitute its output within the parameters being passed to `echo`.

As you can see, you can work around single quotes' limitations for most day-to-day usage, and turn to double quotes, escape sequences, and parameter expansions whenever you have a particular need for doing so.

## Single and double quoting aliases

Single quotes (`'`) are required when using spaces on your alias assignments; nevertheless, it's generally advisable to use them regardless of the spaces on your right-hand side. Granted, this is a "just to be safe" approach, but trust me when I say it will save you from a couple of avoidable headaches when declaring your aliases.

On the other hand, if you wish to use things such as environment variables or parameter substitution within your assignment expression (think of the prompt escape sequences we saw in the previous chapter), double quotes (`"`) are required instead. Imagine you wish to output the current user's name with an alias. As we saw earlier, the direct way of accessing it is via the environment variable `$USERNAME`. The first thing that comes to mind then is to use the following alias:

```
# This is wrong!
alias saymyname='echo $USERNAME'

```

Unfortunately, this won't work with single quotes. The correct way of doing this is with double quotation marks as follows:

```
% alias saymyname="echo $USERNAME"
% saymyname
> gfestari

```

Complex expressions with variables generally need to be quoted, we use single quotation marks for that. If your alias requires variables to be expanded before the alias is used, go with double quotes.

As you can see, the `alias` mechanism is indeed a nifty powerful feature. If used properly, it even allows you to reshape a command's meaning:

```
% alias ls='ls --color=auto'

```

Or its equivalent on OS X:

```
% alias ls='ls –G'

```

### Tip

You can define aliases from another alias. Following the preceding example, if you do the following:

```
alias ls='ls --color=auto'
alias la='ls -a'
```

The `la` alias will behave just as if you typed `ls --color=auto –a`, there's no need to type `--color=auto` again on your definition.

This alias alters the behavior of `ls` by calling it with the `color` flag every time you type it, instead of using its more vanilla version. While this comes in handy for this particular scenario, it can be really dangerous for commands such as `rm` if not used with caution.

For example, imagine what would happen if you aliased a force removal of files to `rm`:

```
# Be careful when doing things like this!
alias rm='rm -f'

```

Here you are forcing the deletion of files without a warning. What will happen is that someone, unknowingly, will execute the wrong command and end up bashing their heads on the keyboard after a mistaken deletion. The takeaway message here is that the further you stay from overriding an existing command with your "l33t alias", the better. Think of all those broken keyboards. Don't be that guy.

So, what if you don't know whether you are bypassing any of the aliases' current set for your session? Well, there's a command for that. Typing `alias` will list all the aliases for the current session:

```
% alias
> la='ls -aF'
ll='ls -lF'
lla='ls -laF'
saymyname="echo $USERNAME"

```

You can also get the information for a particular alias simply by specifying its name, as follows:

```
% alias la
> la='ls -aF' 

```

And you can disable, albeit temporarily, any existing alias by typing:

```
% unalias <aliasname>

```

Simply replace `aliasname` with the name of the alias you want to turn off. This comes in really handy on those occasions when you're using a particularly strict program whose options, or even command-line syntax, is overridden by an alias.

### Tip

There are a few ways by which you can prevent the shell from executing an alias that is called just as another command. Single-quoted commands and commands prepended with a backslash (`\`) as well as those typed as relative or absolute paths are not treated as aliases by the shell.

For example, you could use either of the following if you wish to avoid a call to an alias:

```
% 'ls'
% \ls
```

or even:

```
% /usr/bin/ls
```

Zsh also has `command` that will execute any argument as an external command instead of as a function or built in. Thus, you can also use it to avoid aliases. That would leave us with the following for the preceding example:

```
% command ls
```

You can get more information via `man zshbuiltins`.

As with many other things we'll be discovering throughout this book, an alias is not a golden hammer and as such, you shouldn't be aliasing willy-nilly throughout your terminal sessions. Here are a few simple considerations to keep in mind before you embark into your less-typing adventure with your pack o'aliases:

*   Is my alias easier to remember? `echo -n` is way simpler than aliasing something like `echodontprinttrail`. Keep it simple, don't use aliases for the sake of it. The "Future You" will be really thankful two months from now.
*   Is my alias easier to type? An awkward alias is an awful alias. Using `alias grepcola='grep --color=auto'` instead of a simpler `grep`, really? Remember: clear, concise names are awesome, but something you can't remember for the life of `ping` is not cool.
*   Is my alias overriding some behavior just for the sake of it? Think of the previous `rm -f` example. Most of the time we would like to stay away from something like that; however, prompting the user each time seems like a sensible feature to add to our toolbox. Aliasing `rm='rm -i'` so the shell requires confirmation before deleting something seems a bit... nicer. Be careful with these kinds of tricks though, as relying too much on such an alias could lead to a false sense of security. That is, imagine what would happen if you get used to `rm` constantly waiting for confirmation and eventually use it recklessly on a different environment?

## Global aliases

If you enjoyed the simplicity aliases brought to the table, then global aliases are the icing on the cake. As the name implies, *global aliases* are, well, aliases that can be used anywhere, allowing you to treat filters or certain commands as a simple suffix.

Let's see some examples:

```
alias -g L='|less'

```

Pay particular attention to the `-g` option, as in a global alias.

Now you can append the `less` pager to any command's output, just by adding the `L` suffix:

```
% ls -la /etc L

```

Another option that is frequently spotted in the wild is redirecting standard error (`stderr`) and standard output (`stdout`) to `/dev/null`, so any given command can run silently:

```
alias -g NUL="> /dev/null 2>&1"

```

This will allow you to call things such as `command NUL` without the need to pollute your current terminal window with thousands of log lines and messages.

Just for the sake of clarity, never mind sticking to standards, it's advisable for you to try and define your global aliases just like your global variables, all in caps.

## Hashes

You can use a hash to give a particular directory an alias. This is particularly convenient for your workspace:

```
% echo $GOPATH
> /Users/gfestari/workspace/go

```

I don't want to type `/Users/gfestari/workspace/go` each time I want to reach the `src` folder in my `$GOPATH` directory. So how about putting hash to a good use?

```
% hash -d gosrc=$HOME/go/src

```

And now we can get there as quick as typing `cd ~gosrc` (pay attention to the leading `~` character).

Here's another example, this time using the `/var/www` directory:

```
% hash -d www=/var/www
% cd ~www
/var/www

```

You can go ahead and hash your most frequently visited directories. Just remember to add the required entries to your `.zshrc`, so you don't have to type the same thing over and over again.

For bonus points, set the `AUTO_CD` option, so you only need to input the directory's name whenever you want to change the working directories:

```
% setopt autocd
% ~www
> /var/www

```

Now go ahead and start showing off with your acquaintances, I'll wait here.

## Putting it all together

Before we move on to the next topic, here's a couple of things to try with our newly found aliases.

Raise your hand if you found yourself typing `cd ..` more than a few times on a terminal session. I know, I feel your pain. How about making it simpler?

We could try the following instead:

```
% alias ..='cd ..'

```

Now it's just a matter of typing `.` to move your current working directory up one level`.` Not bad, uh? We can take it a bit further:

```
alias ...='cd ../..'
alias ....='cd ../../..'

```

I'd argue that going more levels in for directory changing is pushing things a bit too far, but feel free to extend your aliases as you see fit.

What about creating directories? I bet that, like myself, more than once you saw the following:

```
% mkdir dir1/dir2
> mkdir: dir1: no such file or directory

```

This happens because `dir1` doesn't exist. So what we do is—you guessed it—create an alias that allows us to automatically create the *parent* directory and also, be more verbose (as in, "list directories as they are created") about the output:

```
alias mkdir='mkdir -pv'

```

Now try to issue `mkdir dir1/dir2` and see what happens. You can also apply the same switch to commands such as `cp` and `mv`, just remember to quote your assignments!

### Tip

You can use the `COMPLETE_ALIASES` option in your startup files in order to force the shell to treat aliases as a distinct command for completion purposes. In other words, the alias won't get substituted before attempting completion.

# Expansion

The shell allows you to perform different types of manipulations right before executing a line. In the following section we'll learn how to take advantage of each of the different forms of expansion and substitution available in zsh.

## Parameter expansion

Parameter expansion allows you to replace known variables in between the assignments of the command line. Simply put, parameter substitution is the mechanism by which the shell can change the following:

```
% foo=Hello

```

It will be changed to the following:

```
% echo "${foo}, world!"
> Hello, world!

```

Notice how the variable `foo` we declared in the previous line is replaced inside the arguments of `echo` with its actual value. You should be paying special attention to that peculiar `${}` construction. What happens is that when zsh reads the `${foo}` construction, it immediately knows it has to replace what's in it with whatever value it holds.

The astute reader might also have taken notice of the double quotes that surround the `echo` arguments. It's important to remember that just like aliases and prompt sequences, parameter substitution will work for arguments passed between double quotes, just like every other variable out there.

## Command substitution

Like parameter expansion, command substitution allows the shell to execute a command call and replace its output within a specially formed syntax. Command substitution usually takes the form of `` `command` ``, a program name wrapped around back quotes.

There's another form of program substitution available in newer shells such as zsh, which takes the form of `$(command)`. Both forms of substitution, ``` `` ``` and `$()`, mean the same; however, back quotes are considered a bit more portable, as they are recognized on pretty much every other shell out there.

In the wild, command substitution is frequently used to find out the full path to a command:

```
% print $(which zsh)
/usr/local/bin/zsh

```

Or, to make it more portable:

```
% print `which zsh`
/usr/local/bin/zsh

```

## Arithmetic expansion

Don't let the name discourage you; just like parameter substitution, arithmetic expansion is yet another form of substitution to help us sail swiftly across the command line. As its name implies, you can expand your input into a series of elements that will otherwise require you to type a lot.

Let's try it:

```
% echo $(( 5 + 4 ))
> 9

```

We started with some rather simple arithmetic expression (I know, I know; Math). But fret not, what just happened can be easily explained. We already know `echo` prints information into the standard output, so there are no mysteries there. What follows it is just an arithmetic expression using the `$(( ))` construct. Notice that, unlike parameter substitution, this kind of arithmetic substitution requires an extra set of parentheses. This is our way of letting zsh know that it needs to work with numbers, and that's why our `5 + 4` is treated as such.

The same rules apply to the following:

```
% echo $(( 5 + 4 * 3 ))
> 17

```

Which leads us to realize we need more parentheses for operator precedence:

```
% echo $(( (5 + 4) * 3 ))
> 27

```

Remember, the `$(( ))` construct is just a special construct, it's what tells zsh to treat what resides inside as arithmetic expressions.

Interestingly, we can invite parameter substitution to this party too. Looks like variables can also be substituted inside arithmetic expressions:

```
% num=5+4
% echo $(( num * 3 ))
> 27

```

In the preceding snippet, we declare a variable to hold our `5 + 4` expression; this makes `num` a container that, when asked what's up, will yell out our `5 + 4` expression. An easy way to check this is:

```
% echo ${num}
> 5+4

```

Note however, that by using `num` in the expression we did not require an extra set of parentheses in order to set operator precedence. This is because our `num` variable gets replaced on the following line with its value, which leaves us with an expression equivalent to `(5 + 4) * 3`. Expressions get evaluated before they are replaced, otherwise the result of the preceding call would have been `17`.

Let's kick it up a notch with another handy arithmetic substitution:

```
% num=5+
% echo $(( $num 4 ))
> 9

```

On this opportunity, we are leaving the `num` expression as something that resembles "add whatever follows to it". This is why when it is evaluated on the next line, it gets replaced for what you'd expect, in this case `5 +`. See that `$` right before the `num` variable? Remember parameter substitution from the beginning of this section? That's what's going on here. Without that `$num` there, zsh simply does not know how to deal with the `num` assignment:

```
# This is horribly wrong!
% num=5+
% echo $(( num 4 ))
> zsh: bad math expression: operator expected at `4 '

```

Remember, if you wish to substitute a parameter, use `$`:

```
% echo $(( $num 4 ))

```

### Tip

You can always have a look at all supported types of expansions by typing `man zshexpn` on your console.

## Brace expansion

Another useful type of expansion is known as brace expansion. As the name implies, its syntax has to do with the use of curly braces (`{}`)—I suppose "curly brace expansion" was a bit too verbose when they were picking a name for it. Brace expansion allows you to declare an array of entries as follows:

```
% echo picture.jp{eg,g}
> picture.jpeg picture.jpg

```

What happens then is that the `{eg,g}` construct gets expanded into an array containing the elements `eg` and `g`. The shell then loops through those elements, passing two arguments to the `echo` command, which basically has the same meaning as typing the following:

```
% echo picture.jpeg
% echo picture.jpg

```

But you saved yourself from quite a few keystrokes and the accompanying boredom. Let's try another example:

```
% touch log_00{1,2,3}.txt
% ls
> log_001.txt  log_002.txt  log_003.txt

```

This time we are creating simple logfiles with the pattern `log_00<num>.txt`. The shell expands the `{1,2,3}` element into the elements `1`, `2`, and `3`, and then calls the `touch` command three times:

```
% touch log_001.txt
% touch log_002.txt
% touch log_003.txt

```

In case you didn't notice, we use commas (`,`) in order to declare each of the elements inside curly braces. Now, you might be thinking "what happens when we use a longer array?" Here's when it gets even more interesting; declare a range of values:

```
% touch log_{007..011}.nfo
% ls | grep .nfo
log_007.nfo  log_008.nfo [...] log_010.nfo  log_011.nfo

```

It's worth noting a couple of things going on with the preceding example. I took the liberty to format the output of the list. But that (`…`) implies files `007` to `011` do exist. Firstly, we are now using brace expansion to extend a range, this time from nine up to eleven. The next thing that's also worth mentioning is that zsh is smart enough to notice the leading zeros and use it as a padding for the other values instead of replacing them with, say, vanilla integers. That is why you see the sequence starting with `log_007.nfo` and ending with `log_011.nfo`.

On the second line, we use a pipe symbol (`|`) to link or redirect output between different commands on your shell. This way we are listing the contents of the file, and redirecting the output into the utility `grep`, so we can filter said output by the `.nfo` extension.

Arrays can get even more interesting when we sparkle a bit more math in them:

```
% foo=(A B C)
% bar=(1 2 3)
% echo $^foo-$^bar
> A-1 A-2 A-3 B-1 B-2 B-3 C-1 C-2 C-3

```

In the preceding snippet, we declare two arrays, one containing the elements `A`, `B`, and `C`, and the other with the elements `1`, `2`, and `3`. The call to `echo` then passes the argument `${^foo}-${^bar}`. Notice the `^` operator (curly braces were implicit on the previous call, so I added them here for the sake of clarity). Again, we are telling zsh to expand the variables that come after the `$` character, only this time we get a **Cartesian product** instead of, say, `A B C-1 2 3`. This is because the `^` operator serves as the array expansion expression. So as far as zsh is concerned, we're using each element of the array independently.

For a more detailed description of how array expansion and the `^` operator works, visit `man zshoptions` (particularly, the `RC_EXPAND_PARAM` section) and `man zshexpn`.

### Tip

As with other sequences, some characters are considered "special" and need to be escaped. Commas and single quotes need to be escaped with a backslash:

```
% echo \'{\,,\'}\'' needs to be escaped'
> ',' needs to be escaped ''' needs to be escaped
```

# Working with history

Like an elephant, many modern Unix shells tend to remember in great detail the copious amount of commands entered while working with them. As many others, zsh too boasts a history log and an even more convenient way of accessing each of its entries. Being able to glimpse at what you have been up to is not only practical from a work-log perspective, but also as a way to speed things up. Think about it; you could use `history` to see (and eventually edit) a previously typed command, get a bit of context as to what's going on with your system, or avoid retyping the same thing over and over. Being able to easily retrieve a command from the past sounds awesome, because it is indeed a really neat feature.

We'll now take a look at how to use zsh's history expansion to work with previous entries in the command line.

### Note

**Working with history**

A more traditional approach to recalling history entries is by using the up arrow and down arrow keys on your keyboard to scroll through history entries. We'll have a closer look at how to alter this behavior when we examine the zsh line editor (ZLE) module in the next chapter. For now, we'll pretend that these are the only keys to move around history.

## History expansion

One of the ways zsh provides for you to access your history is via the so-called history expansion. This works whenever your input begins with the bang `!` special character. As we saw in the previous chapter, the default behavior of the `!` character can be overridden by setting the `histchars` shell parameter to something different:

```
% set histchars='@^#'

```

Unlike other shells though, zsh accepts up to three parameters when setting `histchars`. In addition to expansion (changed to `@`), the other two are used for substitution (`^`) and comments (`#`) respectively.

By replacing the default bang (`!`) with the `@` character, you can now do things like calling your last executed command line as follows:

```
% ls *.txt
> readme.txt notes.txt
% @@
% ls *.txt
> readme.txt notes.txt

```

By redefining `histchars` you'll be able to use commands that actually require special characters such as `!` without the need to escape them or worry about history substitution. You can choose any combination that you want, but, as a rule of thumb, try to stick with the less frequently used characters so that it is actually worth the effort.

### Note

History expansion will only work if you are running an interactive shell and the option `NO_BANG_HIST` is unset in your `.zshrc` file.

Accessing your history entries is done via what we call *event designators*. Like escape sequences, designators are fancy names for constructs that the shell expands in order to know exactly what needs to be retrieved from history. One of the most popular and helpful event designators is the double bang (`!!`), which by itself refers to the most recent command entered:

```
% sh myscript.sh
> myscript.sh: Error: you need to be root to execute this.
% sudo !!
> myscript.sh: executing myscript.sh

```

As you can see, the `!!` character can be really useful for those occasions when you forget to run something on elevated privileges. What happens then is that zsh immediately expands the reference to the last command in history and replaces it in the line that contains the `sudo` call, saving you from entering the whole line again.

Having the shell making substitutions and automatically executing commands demands a bit more "blind faith" than most of us would like to deposit on their shell. Luckily, we can set the `HIST_VERIFY` option in `.zshrc` to force zsh into asking for confirmation every time you bang a command:

```
% setopt HIST_VERIFY
% echo 'Hello!'
> Hello!
% !!
% echo 'Hello!'

```

As you can see, the shell completes the input in your prompt using the previous command, but does not execute it. This is really useful for things like elevated privileges or sudo commands. Feel free to go ahead and add `setopt HIST_VERIFY` to your `.zshrc` file, as we'll assume it is being used from now on.

That's really neat for the command we just typed, but what if the previous command is further back in the history timeline? Well, then we need to use the vanilla event bang:

```
% !cat
% cat /etc/hosts | grep 127.0.1.1

```

Here my last executed command that had `cat` in it was a printout of my `hosts` file (`cat /etc/hosts`), followed by a call to `grep` as I was looking for lines that have `127.0.1.1` on them.

If you connect to a remote host using SSH, you could use something like the following to retrieve the last run connection:

```
% !ssh
% ssh gfestari@192.168.1.10

```

As you can see, the syntax for history expansion is fairly easy to remember. Just put a `!` character together with the command you're looking for and let zsh work its magic.

### Tip

*Word designators* indicate the words of the command line that will be included in a history reference. What follows is a quick reference of the available designators:

*   `^`: The first argument.
*   `$`: The last argument.
*   `%`: The most recent match for a given word.
*   `x-y`: A range of words. Negative indexes like `-i` mean `0-i`; thus, `-1` would mean "the penultimate entry".
*   `*`: All the arguments. Return null for events with just one word.

Note that a `%` word designator will only work when used as `!%`, `!:%`, or `!?str?:%`; anything else and you will be greeted with an error.

For a more in-depth look at word designators and history expansion semantics, please refer to `man zshexpn`, particularly the section titled "HISTORY EXPANSION".

Let's kick it up a notch then; you can combine the special characters `^` and `$` in order to access the first and last arguments of a history entry respectively:

```
% mkdir new_folder
% cd !^
% cd new_folder

```

The `^` character gets expanded into the first argument of the `mkdir` command, which in this particular case is `new folder`.

```
% touch log1.txt log2.txt
% nano !$
% nano log2.txt

```

Here the same happens with `$`, only this time the last argument of the `touch` command is expanded so we can eventually edit it using `nano`.

If you are familiar with regular expressions, both of these designators' behavior shouldn't be too surprising. However, if what you need to do is access some string that is not located either at the beginning (`^`) or end (`$`) of the history, then you need the `?` designator:

```
% !?etc
> cat /etc/hosts | grep 127.0.1.1 

```

The preceding expression matches the most recent command containing `etc`. Generally speaking, the syntax for using the `?` event designator can then be summed up as follows:

```
!?str[?]

```

The optional `?` you see there at the end is only necessary if the command is followed by any text that is not to be considered part of `str`; for example:

```
% !?etc?^
> /etc/hosts

```

Did you notice how both the `?` characters serve as delimiters for the `etc` keyword? Think of them as parentheses that wrap the expression you're trying to match. The caret operator (`^`) is there as we are interested in the first argument of that particular command line, which coincidentally is the `/etc/hosts` string.

There's lots more we can do with the history bang operator. Another neat trick is that it can refer to a particular line in your history. As before, the syntax is merely a tweak of what we already know:

```
!<hist_number> 
% !103 # this retrieves the 103rd entry in your $HISTFILE.
% !4   # this retrieves the 4th entry.

```

But what about knowing which line I want to use? Well, that's a bit more complex, but not as much as using `grep`, `ack`, or whatever it is that kids are using these days to search within your history file:

```
% history | grep nano
> 2045  nano /etc/hosts

```

Using `grep` and searching for entries that feature `nano`, I can see that I edited `/etc/hosts` with it, and that the record resides on line `2045` of my `$HISTFILE`. If we wanted to open the hosts file again, it'll be a simple matter of calling:

```
% !2045
% nano /etc/hosts

```

And now for a bit of mix and match:

```
% history | grep git
> 1571  cd ../git/dotfiles
 1572  git status
 1573  git diff zsh/zsh_funcs
 1574  git diff zsh/zshrc
 1584  history | grep git

```

On this opportunity I'm looking for `git` entries. As you can see from the results, there are quite a few things I've been doing with it. Combine what we have learned so far, and we can accomplish quite a few things:

```
% more !1573$
% more zsh/zsh_funcs

```

As you can see, we used the bang operator together with the `$` selector to refer to the last argument of line 1573 of our history.

Interestingly, you can also use a negative integer to refer to the nth-to-last entry:

```
% !-2 # this will retrieve the 2nd to last entry in history.
% !-97 # this does the same to the 97th to last entry.

```

Negative indexes should be pretty familiar territory for some programmers (I'm looking at you, Python and Ruby developers).

## History substitution

Another helpful feature of the history expansion on zsh is command substitution. Using this kind of substitution, you can avoid re-entering a whole line of your shell history just so you can edit a comparatively smaller section of it.

Raise your hand if you have made something like the following:

```
% ls
> dir1  file.txt
% mv fiel.txt dir1/
mv: rename fiel.txt to dir1/fiel.txt: No such file or directory

```

It seems I misspelled the `file.txt` name, so what now? Traditional history usage would suggest we just press the up arrow key to recall the previous line, navigate left to the `fiel` typo, re-type the correct name, and be done with it. The zsh approach however, is a bit more practical:

```
% ^fiel^file
% mv file.txt dir1/

```

What sorcery is this? Put simply, the chained `^` operator allows you to match the first occurrence of a word and replace it with the word attached to the second `^` operator. A more general syntax would be:

```
^history-entry^word-replacement

```

### Tip

You can prevent a command from being added to your history by setting `HIST_IGNORE_SPACE` in your startup options. This will make the shell ignore the lines that start with a space.

```
% echo "this line will be recorded in history"
%  echo "this will not"
```

## More useful options

To round off this section, here are a couple of history-related options worth considering when populating your startup files, in addition to what we have already discussed throughout this chapter. Just put any (or all) of these on your `.zshrc` and remember to append `setopt` before each entry.

*   `EXTENDED_HISTORY`: Saves a timestamp and duration for each history entry run. An excellent addition for the data analysis aficionado.
*   `HIST_IGNORE_ALL_DUPS`: Ignores duplicate entries when showing results.
*   `HIST_FIND_NO_DUPS`: Does not display eventual duplicates of a line that has already been found.
*   `HIST_REDUCE_BLANKS`: Removes extra spaces and tabs from history entries.
*   `INC_APPEND_HISTORY`: Adds entries to the history as they are typed, that is, doesn't wait until the shell exits. Probably one of the most awesome features of zsh. You know you want this.
*   `SHARE_HISTORY`: Shares history between different zsh processes. Another great option to compliment the previous entry.

# Summary

In this chapter we had a look at some of the most prominent time-saving features of zsh. The purpose of this entry in our shell adventure was to start accomplishing more by typing less. Thus, this chapter focused on understanding aliases, how they work, and how to roll our own keystroke-saving definitions in a way that won't cause more trouble than what they attempt to solve.

We then moved onto expansions, learning the ways of arithmetic and brace expansion in order to make command-line related chores feel more like a breeze. Finally, we took a closer look at how to work with history, going beyond the keyboard arrow-mashing approach and learning history expansion and event designators in order to avoid repeating ourselves into oblivion.

By now you should have a fairly solid notion regarding the following:

*   **Aliases**: We learned what an alias is and how to define a useful shortcut for our commands together with a handful of tips to start your collection.
*   **Parameter expansion, command substitution, and arithmetic and brace expansion**: How to replace entries on the command line with the output of any given program, the result of an arithmetic expression, and even how to expand arrays so you don't have to type the same thing more than once.
*   **History expansion and substitution**: How to apply all of the above, together with some more specific constructs such as the double bang (`!!`) to the shell's history and avoid repeating yourself to boredom.

Not bad at all. Pat yourself on the back or go grab a beer, by now you should feel confident enough to work your way around zsh without problems. That's great, but there's still much more left for us to discover, so don't slack! Next in store for us is ZLE, the zsh line editor. We'll get to know another of zsh's cooler features and discover that we don't actually require a dedicated program in order to perform some of the more advanced text processing on the command line. Besides saving us hundreds of hours of mind-numbing repeated keystrokes, we'll also learn how to customize the editor's shortcuts and key bindings so we don't have to rely on guesswork anymore.