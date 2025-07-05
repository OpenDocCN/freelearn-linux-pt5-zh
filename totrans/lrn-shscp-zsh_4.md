# Chapter 4. Globbing

In this chapter, we will get to know one of the most powerful features of zsh: filename generation. We will learn new ways of dealing with the system's files and directories and even expand the functionalities of some of the more traditional commands by applying parameter substitution and modifiers. The chapter also serves as an introduction to zmv, a built-in function that provides a number of useful functionalities to deal with both the mundane and the more complex tasks regarding files. We will learn to use zmv for renaming, copying, and linking files based on our newly learned patterns. Feeling excited already?

# Quoting your strings

A safe way of declaring your string variables involves the usage of quotes. Think of it as a way of telling the function "*here* starts and *over here* ends my string". Although not necessary on this particular example, you can quote a phrase when using `echo` as follows:

```
% echo 'this is a quoted phrase'
> this is a quoted phrase

```

Single quotes are treated as delimiters by the shell and as such, they are completely ignored. The same rule applies to the `print` built-in function:

```
% print 'this is a quoted phrase'
> this is a quoted phrase

```

So, what's the point of using quotes then? Well, imagine for a moment that your output looks something like the following:

```
% echo this is a backslash: \
~>

```

Yes, that will trigger a continuation line, so there's seemingly no way around it, save for using quotes. Let's try it again:

```
% echo 'this is a backslash: \'
> this is a backslash: \

```

So, as a rule of thumb, we use single quotes when there are special characters on our string as follows:

```
% echo 'special characters like * # and \ need to be quoted'
> special characters like * # and \ need to be quoted

```

Now, what's it that makes these special? Well, earlier in this book, we saw that comments are defined by a `#` sign; we can use the `*` character as a wildcard that matches filenames and the `\` character can be used for escaping sequences with special meaning. Think of all these as *special characters* that will never literally mean what the keyboard says, unless you quote them.

### Tip

Some special characters need to be "escaped". This means that they will have a different meaning other than the characters they represent, unless there's a `\` character before them, that is.

For example, `echo *.rb` will list all the files that have an `.rb` extension. If you wanted to list a directory named `*.rb`—weird, I know—you would have to call `echo` escaping the `*` special character as follows:

```
% echo \*.rb

```

Also worth noting is that `\` is actually a special character, so in cases where a literal backslash is required, you will need to escape it too:

```
% echo \\
> \

```

As we saw in the previous chapter, a single backslash (`\`) will only trigger a continuation line.

### Tip

You can make the shell output the raw string by supplying the `(q)` argument:

```
% string="This is a *string* with various 'special' characters"
% echo ${(q)string}
> this\ is\ a\ \*string\*\ with\ various\ \'special\'\ characters
```

## Double quotes

Okay, so what happens when we need to use the niceties of the special characters and also need them to appear as their literal representations? Enter the double quotes.

### Tip

The option `RC_QUOTES` allows you to use single quotes within a single-quoted string:

```
% setopt rcquotes
% echo 'a single ''quoted'' string'
> a single ''quoted'' string
```

Double quotes work by allowing you to retain the value of any string and also enabling *parameter substitution* and *shell expansion* within them.

Take a long, hard look at the following example:

```
% echo "My username is $(whoami) and my home folder is located at '$HOME'."
> My username is gfestari and my home folder is located at '/Users/gfestari'.

```

The shell works inside the double quotes by executing the command within the `$()` construct before anything else. In this particular case, we are using the `whoami` program to tell the current user ID—`gfestari` in this particular case—(if that's also your name, then *hello*, long-lost brother).

The shell then moves on to expand the environment variable `$HOME`, which holds the current user's home folder currently pointing at `/Users/gfestari` on my system. Notice how the single quotes are treated like any other character within double quotes.

# Getting started with Globbing

Filename Generation, popularly known as **Globbing** (as in, Global substitution), is the ability of the shell to generate filenames from patterns. This is but the name for the process that allows the shell to read a pattern and generate a series of filenames; as a matter of fact, you might notice you have been using Globbing for quite a while in this book, the only difference is, we're now formally introducing the feature. Also, be aware that whenever we mention *filenames* in this text, it means both file *and* folder names, as you can use pretty much the same patterns to match both.

The really important thing you need to remember when dealing with Globbing is that filename substitution happens in the shell *right before* the line you typed is sent to the command. In other words, you type, zsh does the substitution, and *only then* sends the result, an expanded string and not whatever you just typed, to the function or program. There are ways around this, but just be mindful.

### Tip

If you'd like to take a deeper look at some of the features covered in this chapter, you can always refer to the official documentation by typing `man zshexpn`

## Globbing with the stars

Globbing works by using a series of special characters known as *operators*, to create a pattern that is later expanded by the shell into more complex, traditional strings without you even noticing the extra effort required. Arguably, the most popular of these operators is the asterisk or star (`*`). The star works as a *wildcard*, allowing you to match any filenames, even if you provide no pattern at all:

```
% echo *
README.md todo.txt draft.txt new_file.txt

```

This will list any file and folder on your current directory. Notice how we only needed a *single star* for this. However, if we want all files with a `.txt` extension, we simply need to provide the appropriate pattern: anything that ends with the desired extension.

```
% echo *.txt
todo.txt draft.txt new_file.txt

```

What happens is that zsh reads the `*.txt` pattern, transforms it into its literal meaning (all the filenames with a `txt` extension), and only then passes the result as the argument for `echo`, which in turn never deals with the actual pattern.

Arguably, the best thing the star has going on is its versatility. Just like a drunken sailor, a star can get along with practically anything, not just files:

```
% echo *folder
out_folder src_folder

```

### Tip

You can use the `NO_CASE_GLOB` option if you want to make Globbing case-insensitive (that is, treat upper and lowercase characters as equals).

```
% setopt nocaseglob
% echo *.jpg
photo.jpg pic.JPG
```

It's not all sunshine and rainbows though. There's a fine print detail that you should consider when using the star operator: hidden files. If you recall from [Chapter 2](ch02.html "Chapter 2. Alias and History"), *Alias and History*, we used an alias, `la` (or `ls -a`), in order to list the hidden files within a directory; otherwise, the command wouldn't list them.

Because of how big a headache it could cause you to do things like `rm *` and end up deleting a parent folder, most Unix shells will simply ignore hidden files for most commands. The same rules apply to Globbing when using the wildcard operator. A workaround for dealing with this behavior would be to explicitly use a pattern along the lines of `.*some_pattern` in order to include hidden files just like the following:

```
% echo .*zsh*
.zsh_aliases .zsh_funcs .zsh_history .zsh_prompt .zshenv .zshrc

```

We use two stars in order to list all the files that start with a dot (traditional hidden files in Unix) and contain a `zsh` pattern somewhere in their name. In other words, our startup files.

The takeaway lesson here: *you can use the star anywhere on a pattern*, you don't have to limit yourself with length or just extensions; be mindful of the hidden files though, as a star won't show you any hidden files, you'll need something along the lines of `.*some_pattern` for that to work.

### Note

You can always circumvent the "ignore files starting with dots" behavior by setting the `GLOBDOTS` option; however, it's advisable you refrain from setting it permanently on your startup files as it can lead to issues such as you deleting the parent (`.`) directory and so on.

The most important thing to keep in mind when using this option on your scripts or functions is ensuring a call to `setopt NO_GLOBDOTS` right before exiting. Most times though, you'll do just fine by using the `.*` pattern discussed previously.

## Questions for any single character

The question mark symbol works pretty much like the star, except it matches a single character instead of many. For example, you can use `ls ???` to list the contents of any three-lettered directory, or get a bit more practical and use the following to list any two-lettered extension file:

```
% echo *.??
script.sh

```

We can even view all files with an extension via the following, similar expression:

```
% echo main.?*
main.c main.o main.tmp

```

This is similar to the wildcard qualifier; however, you won't be able to match any filenames with leading dots unless you explicitly declare so.

## Brackets for a sequence of characters

You can use the square brackets construct to match a group of characters within a pattern. For example, you can use `[ML]*` to match any filename that starts with either an uppercase letter `M` or `L`.

```
% ls
Log.log Main.rb README.md script.sh
% echo [ML]*
Log.log Main.rb

```

Notice how we need to combine the character class operator with the wildcard in order to denote the filenames that might have more than a single uppercase letter.

Even more useful is the use of a hyphen (or minus sign) in order to name ranges of contiguous characters to match. For example, you can use the `[A-Z]*` pattern to match any file that starts with an uppercase letter from the alphabet. Likewise, you can use the same pattern for contiguous natural numbers:

```
% echo *.log_[1-9]
out.log_1 out.log_2 out.log_3

```

Simple enough, right? Remember you can declare your own character classes. Here's an example that matches any filename starting with any number from one to five or an uppercase `M`:

```
% echo [1-5M]*.*
Main.rb

```

Just as before, a `[.]*` pattern won't work as you might expect; in fact, it won't work at all.

### Note

**A note about ranges**

If your system is using a non-English alphabet or something other than the ASCII character set, chances are you might expect things like `ü` to match classes like `[a-z]`. This behavior, however, is ruled by the `LANG` and `LC_*` family of environment variables and is *very* system dependent, not to mention, beyond the scope of this book.

## Using safer ranges on your scripts

Although nothing to write home about if you have been using any modern shell lately, there's a series of shortcuts that save you from boredom when working with the garden variety of character classes. You can access them via the `[[:shortcut:]]` pattern.

So, for example, if you needed any letter from the alphabet (say, the range that includes both uppercase and lowercase English characters `[A-Za-z]`), you could use the `alpha` shortcut to list any filename that starts with a character from the alphabet like so:

```
% echo [[:alpha:]]*

```

Feeling enthusiastic about character sets already? The following table lists some of the popular ones:

| Character set | Description |
| --- | --- |
| `ascii` | Anything from the ASCII character set (see `man ascii`) |
| `lower` | Lowercase character |
| `upper` | Uppercase character |
| `alpha` | Letter |
| `digit` | Number |
| `alnum` | Alphanumeric character |
| `print` | Any printable character |
| `blank` | Space or tab character |
| `space` | Space character (tab, carriage return, newline and co.) |
| `punct` | Anything but an `alnum` nor a `space` |

You can combine multiple patterns and character sets; just remember that the innermost brackets belong to the character set, and everything else goes between the outermost brackets. For example, if we want all the files that start with either a `digit` character or the lowercase `b` letter, we might roll with the following:

```
% echo [[:digit:]b]*.c
bindings.c

```

As you can see, the inner set of brackets declares the character set, while the `b` character is there just as though we had typed `[b]`.

### Avoiding characters

Okay, we have been giving patterns a warm welcome so far, but what happens when we want the thing that does *not* match whatever we're looking for? Turns out there's also an easy way to tell zsh "I want the filenames that have nothing to do with this particular pattern", so let's get to it.

Suppose we have the following files in a given directory:

```
% ls
bindings.c  bindings.h  bindings.o  main.c  main.o

```

And we just want to select the actual code files, the ones ending in `.c` and `.h`, and avoid everything ending in `.o`. With what we have learned so far, we could get away with something along the lines of the following:

```
% echo *.[hc]
bindings.c bindings.h main.c

```

But as you can see, the more complex our requirements, the more likely we end up with a gigantic mess of a character class. Luckily, we can get the complement of a class via the caret (`^`) operator:

```
% echo *.[^o]
bindings.c bindings.h main.c

```

What we did back there was told zsh to expand the class for those filenames *that do not match* the `o` extension. Notice how the rest of the pattern remains unchanged and the caret is immediately after the left bracket that does the actual negation. Feel free to read this as "anything but whatever comes inside the brackets".

### Tip

You can negate a character set by using a caret before the inner brackets. For example, if we wish to skip files that start with an uppercase letter, we might as well do the following:

```
% echo [^[:upper:]]*
```

## Handling mismatches

So far we have seen how to make the shell interpret our patterns and attempt to match whatever filenames it can. During the remainder of this Globbing trip of ours, we'll take a look at what happens with the unlucky patterns, those that fail to match anything and how the shell deals with them.

Let's try listing some nonexistent zip files:

```
% ls
bindings.c  bindings.h  bindings.o  main.c  main.o

% echo *.zip
zsh: no matches found: *.zip

```

It seems that zsh defaults to an error message and aborts the execution of the command. Luckily, there are plenty of things for us to do about it in the form of options.

First, there's `NULL_GLOB`, which will make the shell discard any pattern without a proper match. The following is an example, where a blank line gets printed when no matches are performed:

```
% setopt null_glob
% echo *.zip
> 

```

This comes in handy when passing multiple patterns, but can make you call some programs without any arguments whatsoever, so consider that before updating your startup files willy-nilly.

```
% echo *.c *.zip
bindings.c main.c

```

The first pattern (`*.c`) matches and lists all files with a `.c` extension; whereas the second pattern (`*.zip`) doesn't match anything and is discarded (a null second entry is passed to `echo`).

Moving on, there's also the `NOMATCH` option, which you can unset to achieve a behavior that pretty much emulates bash; any pattern that does not match is passed as a *literal argument* to the command. This is relatively easy to test with the following example:

```
% unsetopt nomatch
% echo *.zip
*.zip

```

What do you know? Seems the manpage was right and now the failing `*.zip` pattern acts just as though we had called `echo '*.zip'`. This works differently from `NULL_GLOB`, in that the pattern is also ignored by the shell, but passed *as an argument* to the program regardless of it matching anything.

### Tip

Remember you could also use `setopt NO_NOMATCH` instead of `unsetopt`.

Lastly, there's an option which mimics the legacy behavior of `csh`, aptly named `CSH_NULL_GLOB`. Yes, naming conventions spare no expenses. Anyway, here's what happens when you set it:

```
% setopt csh_null_glob
% echo *.zip
zsh: no match

```

Seems it's back to the "error message and abort command" zone for us. Like the curious learners we are, let's kick it up a notch and see what happens when dealing with multiple patterns:

```
% echo *.c *.zip
bindings.c main.c

```

Ok, now that's a lot nicer. What happens is that `CSH_NULL_GLOB` will show you an error message and abort the command line whenever any single pattern does not match, but will go ahead and discard the failing patterns if there's at least one that matches. Think of this as the product of that night of unrestrained passion between zsh's default behavior and `NULL_GLOB`. And while we're at it, don't blame me for that mental picture.

Before we move on to another subject though, there's another option you should familiarize yourself with when dealing with patterns. But first, let's take a look at what happens when we try to pass a wrong pattern to the shell:

```
% echo *[[:alpha:]
zsh: bad pattern: *[[:alpha:]

```

Notice how we missed the closing bracket (`]`)? The shell complains about the pattern and we are left with the sour taste of failed scripting. Let's try that again, but now we'll set the following option:

```
% setopt no_bad_pattern
% echo *[[:alpha:]
*[[:alpha:]

```

We turned on `NO_BAD_PATTERN` (or unset `BAD_PATTERN`, whatever floats your boat) and guess what happened? That's right; the bad pattern *is ignored* by the shell expansion mechanism and passed instead as an argument to the command. Pretty handy if you don't want those pesky warnings while experimenting with your newly learned patterns.

# Extended Globbing

As you might have noticed at this point, when it comes to Globbing, zsh goes above and beyond the call of duty and then some more. What we'll discuss next is the more advanced aspects of Globbing, commonly referred to as *extended Globbing*. Put simply, we'll learn a new set of characters and expressions that expand on what we have been using to provide even more functionality to the shell's operations. However, before we ride that horse, pry open that `.zshrc` file of yours and add the following option:

```
setopt EXTENDED_GLOB
```

Or call it from your terminal if you plan on adding it later on. As we'll see in no time, extended Globbing is there to give a special meaning to characters like `#`, which if you recall, is typically used for comments. Now let's get our hands dirty.

## Special patterns

Zsh's vast repertoire also includes a series of shortcuts or special patterns that aim to make mundane tasks a bit more tolerable. We will get familiarized with them in this section.

### Recursive searching

Arguably, the most popular pattern out there is recursive searching. Accessible through the `**/` combination, this pattern tells zsh to perform a recursive search, starting from the current directory and working its way inwards along the directory tree.

For example, here's how we look for all the markdown files (files which typically have the `.md` extension) on the current working directory:

```
% echo **/*.md
README.md brew/README.md git/README.md scripts/README.md zsh/README.md

```

Then there's also the `***/` flavor, which tells the shell to follow symbolic links. Be careful though, as it can lead to errors such as "file name too long", which is the operating system's way of telling you that either the rabbit hole is too deep, or you have a circular reference somewhere.

### Tip

Keep in mind that specialized tools like `find` or The Silver Searcher ([https://github.com/ggreer/the_silver_searcher](https://github.com/ggreer/the_silver_searcher)) will run circles around the shell's directory recursion mechanism. Thus, you should avoid relying on it for "serious" operations.

As for the caveats of using the recursive pattern expression, you might eventually be greeted with an "argument list too long" warning from the system. This usually means the shell is taking up too much memory space when expanding the `**/` pattern into the directory structure, which in turn could happen if you have a really complex tree to work on. A workaround, if you insist on using the recursive expansion, is to pass each argument with the help of `xargs` as follows:

```
% find **/*.md | xargs echo

```

I know, this example is a bit dumb as the same could be accomplished just with a simple `find **/*.md` for a multiple-row result. The idea here is that you get to know how to `pipe` the results of the find into `echo` by splitting them with `xargs`, so bear with me.

Lastly, there's somewhat of a hack you can use in case you want to exclude the current directory from the pattern:

```
% echo */**/*.md

```

That way, only filenames that include `base_dir/any_dir` will match the pattern.

### Alternate patterns

Having to choose between two options and then being given a third one clearly inferior, can make a person rethink his decision... or so the story goes. Luckily, the shell is not a complex creature like us, and we can provide it with a choice of patterns to select should one fail. We do that by using the parentheses with a pipe construct, like the following example:

```
% echo [[:upper:]]*.(md|txt)
README.md README.txt

```

We continue on our search for the `README` files, using a named range to specify the filename we want with an uppercase letter before defining either an `md` or a `txt` extension. Simple, right? Well, not quite. Just be careful so as not to start the command line with parentheses, as this might make them run in a subshell instead. Zsh is smart enough to discriminate between intended usages, so you'll probably be safe most of the time. Try not to push your luck though.

Before we move on, it bears mentioning you can't use a pattern that contains a `/` character within the group alternatives we just learned. You have been warned!

### Numeric ranges

You can make the shell match any series of digits it encounters with the `<->` special pattern. What makes this construct great though, is that it can match any series of digits without a length restriction (this is because the shell processes each digit independently and not as a whole integer).

Take, for example, the following directory:

```
% ls
log.txt      log_002.txt  log_010.txt  log_031.txt
log_001.txt  log_009.txt  log_030.txt

```

We want to work with those files that match the `log_xxx.txt` pattern, where `xxx` is a digit. Let's put what we just learned to good use:

```
% echo log_<->.txt
log_001.txt log_002.txt log_009.txt log_010.txt log_030.txt log_031.txt

```

What if we want those logfiles from `10` upwards? Zsh has you covered:

```
% echo log_<10->.txt
log_010.txt log_030.txt log_031.txt

```

As you can see, the `<->` pattern can define a range with lower and upper bounds. Let's try again, this time for files between `10` and `20`:

```
% echo log_<10-20>.txt
log_010.txt

```

Another cool feat of this expression is that it doesn't take into account leading zeroes, allowing you to sort things such as `00010` and `00013`. Speaking of which, there's the `NUMERIC_GLOB_SORT` option, which you can also set in order to output a sorted numeric match of any pattern matches (and that's *any* as in, not just the numeric range pattern).

```
% setopt numericglobsort
% echo log_*
log_001.txt log_002.txt log_009.txt log_010.txt log_030.txt log_031.txt

```

### Revisiting the caret operator

As we saw earlier, we use the caret (`^`) operator to negate patterns (remember: "anything but what matches this"). Here's another way to use the caret:

```
% ls
README.md  README.txt  bindings.c  bindings.h  bindings.o  main.c  main.o

% echo b^*.o
bindings.c bindings.h

```

So basically, we're telling the shell to expand that pattern so as to match the filenames that start with `b` but do not have an `.o` extension.

We can then safely say that the `pattern^other_pattern` expressions work by matching the first pattern and avoiding matches on the `other_pattern` side of the expression. A word of caution now that we are using special characters with different meanings though is, remember to wrap names or expressions that you want taken literally with single quotes, like in the following example:

```
% echo '^c'

```

Otherwise, you might be asking for trouble.

### The tilde operator

Similar to the caret operator's second usage, the tilde (`~`) operator can be used to define a pattern that consists of a part that should match and a second part that shouldn't:

```
% ls
README.md  README.txt  bindings.c  bindings.h  bindings.o  main.c  main.o

% echo b*~*.o
bindings.c bindings.h

```

Basically, this is just a combination of two patterns: `b*` and `*.o`, linked with the "do not match what follows" operator: `~`. Again, we can read that as "match everything that starts with a lowercase b and does not match anything that ends with .o".

If you recall, we used `b^*.o` with the caret, so the tilde version seems a bit more straightforward if I might say so. But don't take my word for it. Let's use the tilde to exclude, for example, any files within a temporary directory:

```
% ls tmp
delete_me.sh  out.txt

% echo **/*.sh~tmp/*
src/script.sh

```

What happens is that the shell runs the first pattern (`**/*.sh`) and recursively checks for all files with the `sh` extension. The preliminary result is a list of possible filenames that is then matched against the second pattern (`tmp/*`). The filenames that match the latter are removed from the list, and we are left with the filenames we were searching for.

Just for academic purposes, it might be a good time to mention that `**/` is equivalent to the `(*/)#` pattern. As it stands, the special operator `#` will match a single repeating character (in parentheses), or a recurrent expression (in brackets).

## Glob qualifiers

Besides operators, zsh boasts qualifiers, which are essentially a sort of filters you apply to your pattern in order to restrict things like matching only files or folders, type of permissions for those filenames, or even the owner of such entries.

So in the following example, we'll list all the *directories* that match the `*tmp` pattern. Notice the `(/)` construct, that's what intuitively sets files and folders apart:

```
% echo *tmp(/) 
tmp

```

What about matching only vanilla files then? Fair enough, `(.)` is your designed qualifier for files-only restrictions.

```
% ls -F
README.txt  script.zsh  zsh/  src/

```

Suddenly, a wild filename appears:

```
% echo *zsh(.)
script.zsh

```

We have a `zsh` directory and a script file with a `.zsh` extension. Typically, we would roll with an `echo *zsh` construct to list both of them, or a more restrictive `echo *.zsh` construct if we were just looking for files with an extension; however, the `(.)` qualifier is arguably better suited for complex tree searches or when dealing with lots of similar filenames and directories.

What follows is a "cheatsheet" for the most common qualifiers:

*   `(N)`: Remove argument if no matches are found, silently ignore errors. Acts as a per-command `NO_GLOB` option.
*   `(@)`: Symlink qualifier. Used for only selecting symbolic links.
*   `(-@)`: A special variation of the previous one. Use this to find any *broken* symlinks.
*   `(/)`: Directories only.
*   `(.)`: Files only. Whatever is not either a link, directory, or any of the previous will be selected by this.
*   `(*)`: Executable files. Directories need not apply. Think of this as `(.)` for those files with `+x` permissions.
*   `(r)`: File is readable by the current shell user.
*   `(w)`: File is writable by the current shell user.
*   `(x)`: File is executable by the current shell user.
*   `(U)`: File is owned by the current shell user.
*   `(R)`: File is readable by anyone.
*   `(W)`: File is writable by anyone.
*   `(X)`: File is executable by anyone.
*   `(u:root:)`: File is owned by the user `root`. You can replace the `:` character with any another pair of symbols such as curly braces: `(u{root})`. Just refrain from using pipes (`|`).
*   `(on)`: Sort filenames by name. The `echo *(on)` construct will be analogous to `ls`.
*   `(On)`: Reverse-sort filenames by name.
*   `(oL)`: Sort filenames by file size.
*   `(OL)`: Reverse-sort filenames by file size.
*   `(om)`: Sort filenames by modification date.
*   `(Om)`: Reverse-sort filenames by modification date.

As always, feel free to mix and match to spice up things. Like poking with `(*r^w)` for regular files that are readable but not writable by your user, or `(@,/)` for either symlinks or directories.

### Tip

Eager to find out more about qualifiers and what have you? Fret not dear reader, and embrace the mystical powers of... never mind, we'll just resort to *context completion*.

Type the following, and remember to press *Tab* right after the opening parentheses:

```
% echo *zsh*<Tab>*

```

This will yield context completion for the glob qualifiers listed here (and many more!).

What follows are the more complex batch of qualifiers, such as timestamps and file size, which require a bit more explaining before delving right into their usage.

### Timestamp qualifiers

Unix systems typically record three timestamps on their filesystems: modification, access, and change times. With that in mind, you can use the following construct for Globbing filenames:

```
% echo *(mh-1)

```

This will provide you with the files modified in the last hour. You can easily check this result via an `ls -l` qualifier. The `m` there is the modification time, which is the most common type of timestamp you'll be interested in. Nevertheless, you could also check for either access (`(ah-1)`) or creation (`(ch-1)`) qualifiers within the last hour.

Regarding that "last hour" bit, it's represented by the `h-1` qualifier, where `h` stands for hour (yes, yes, I know) and could be replaced by either minutes (`m`), weeks (`w`), or Months (an uppercase "`M`"). Note that the default unit for this qualifier is days, so `(m-1)` will mean a day ago or, more precisely, up to 24 hours before the current system time.

Similarly, the plus operator can be translated as "more than", allowing you to describe such patterns as `(mw+3)`, which is a concise way of saying "more than three weeks from today". Finally, you can also specify a range by combining the two operators:

```
% echo *(m-5mh+2)

```

This will provide the files modified between five and two hours.

### File size qualifiers

The last qualifier you'll get to know today is the file size. As you might have guessed already, we can query filenames on the basis of their size on the disk:

*   `(Lm+size)`: The file size is larger than `size` megabytes. For example: `(Lm+5)`—larger than five megabytes.
*   `(Lm-size)`: The file is smaller than `size` megabytes. For example: `(Lm-2)`—smaller than two megabytes.
*   `(Lk+size)`: The file size is larger than `size` kilobytes. For example: `(Lk+5000)`—larger than 5000 kilobytes.
*   `(Lk-size)`: The file is smaller than `size` kilobytes. For example: `(Lm-2000)`—smaller than 2000 kilobytes.

# The zmv function

In the previous chapter, we learned about `zle`; zsh's module in charge of the command line. It's time we take advantage of our newly learned Globbing skills and get acquainted with `zmv`, a function that was created to make copying, moving, and linking files a breeze.

So, you ask, what's the deal with zmv? What's special about this built-in function in comparison to, say vanilla `cp`, is that zmv works its magic based on patterns. Further, as we'll see in this section, zmv is designed to be safe by default, which means it will ask you for a confirmation before taking on any kind of risky operation such as overwriting files.

Before we get started though, you should add the following to your `.zshrc` file, remembering to source it or restarting your terminal emulator of choice:

```
autoload zmv
```

This will make zsh load the function on startup, making it available to your session. You can now just type `zmv` and you'll be greeted with a fairly straightforward set of instructions. Basically, the zmv syntax expects two patterns: one for matching filenames and a second one into which the results will be converted:

```
zmv [OPTIONS] old_pattern new_pattern
```

As you might have guessed, zmv goes along with a great deal of Globbing, which is why we are only getting acquainted with it now. Here's how we can use it to rename our `.txt` files into markdown (`.md`):

```
% zmv -Wv '*.txt' '*.rb'
mv -- README.txt README.md

```

We used the verbose `-v` option flag, so we can learn more from the output. The `zmv` function works by expanding both patterns and then delegating the actual functionality to a more capable command such as `cp`, `ln`, or in this particular case, `mv`.

You can use the `-W` option to allow automatic conversion of the wildcards. Combined with `noglob`, you can add a brand new functionality to the `mv` command, which resembles the special behavior of the Windows systems' `cmd` variant:

```
alias mmv='noglob zmv -W'
```

You can now move files and rename them on the same call:

```
% mmv *.c.orig orig/*.c

```

As for the rest of the option flags that apply to zmv, here's a handful of the most relevant:

*   `-f`: Force overwriting of destination files
*   `-i`: Interactive prompt for each operation
*   `-n`: No execution, just print what happens
*   `-v`: Verbose—print a line as it is executed
*   `-w`: Implicitly add parenthesis to wildcards in the pattern
*   `-W`: Like `-w`, but turn wildcards in replacement patterns into references

However, don't even think you'll need to remember these. As we'll see in the next chapter, you can always use *Tab* for context completion or, in zmv's particular case, you can get the full list by simply typing `zmv` and pressing *Return* on your terminal. Just know there are at least a couple of options available to you.

### Tip

You can do what's popularly known as a dry run by passing the `-n` flag. This will make zmv only print out what will be done without actually doing it. This is by far the best way of testing and debugging your scripts before… well, you know, panic ensues.

```
% ls foo
% zmv -n '(*)' '${(U)1}''mv -- foo FOO
```

Should you require more advanced usage, you could use several expressions such as the `old_pattern` parameter. Filenames that match these will in turn be grouped and accessible by the `new_pattern` expression following the `$1`, `$2`, … pattern. For example, we can use the following for recursively renaming pictures on a folder tree so that their extensions are all lowercase:

```
% zmv '(**/)(*).(#i)jpg' '$1$2.jpg'

```

Summing up, with a bit of Globbing and practice, you can get a lot of mileage out of your zmv usage. You just need an appropriate pattern to match and a string to actually use that pattern. `zmv` will actually ignore any file whose name is not changed during expansion and it doesn't even care if the target is supposed to be a directory or a simple file.

### Tip

You can access zmv's advanced documentation by typing `man zshcontrib`.

# Summary

This is the part of our journey that requires us to pack up our things and wrap up the chapter. On this occasion though, we went from using Globbing as something we thought was "quite like a regular expression" to understanding what is actually a whole different beast. Luckily for us, that beast was pretty easy to tame once we learned the behavior of the most popular operators and qualifiers. We then expanded on those constructs with more special patterns and got to know **zmv** in order to make most of our daily tasks a breeze. Summing up, we can say that we:

*   Learned about quotes, escaping symbols, and double quotes together with shell expansion within them
*   Got started with Globbing and parameter substitution within the command line
*   Kicked it up a notch and dove headfirst into extended Globbing, learning about recursive searching, and operators for negating and excluding patterns
*   We learned about glob qualifiers, how to use them to discriminate files by the system time and size
*   And finally discovered zmv, which lets us put all of the preceding things together to make working with complex filenames something like a walk in the park

Seems like we have seen a whole lot so far, which will cater to most of our needs. Not a bad deal, if I might say so. Actually, I might, as that's one of the advantages of wearing the writer's hat.

The next chapter covers completion. And we have come together quite well so far, so I won't lie to you (again); completion is actually what makes most people never look back once they try zsh. You have tasted a sample of it so far, but there's plenty more waiting for you, right around this page.

Next up then is [Chapter 5](ch05.html "Chapter 5. Completion"), *Completion*. Hurry up!