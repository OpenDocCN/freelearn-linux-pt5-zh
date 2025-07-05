# Texting and Driving

In this chapter, we will cover the following recipes:

*   Using regular expressions
*   Searching and mining text inside a file with grep
*   Cutting a file column-wise with cut
*   Using `sed` to perform text replacement
*   Using `awk` for advanced text processing
*   Finding the frequency of words used in a given file
*   Compressing or decompressing JavaScript
*   Merging multiple files as columns
*   Printing the n^(th) word or column in a file or line
*   Printing text between line numbers or patterns
*   Printing lines in the reverse order
*   Parsing e-mail address and URLs from text
*   Removing a sentence in a file containing a word
*   Replacing a pattern with text in all the files in a directory
*   Text slicing and parameter operations

# Introduction

Shell scripting includes many problem-solving tools. There is a rich set of tools for text processing. These tools include utilities, such as `sed`, `awk`, `grep`, and `cut`, which can be combined to perform text processing needs.

These utilities process files by character, line, word, column, or row to process text files in many ways.

Regular expressions are a basic pattern-matching technique. Most text-processing utilities support regular expressions. With regular expression strings, we can filter, strip, replace, and search within text files.

This chapter includes a collection of recipes to walk you through many solutions to text processing problems.

# Using regular expressions

Regular expressions are at the heart of pattern-based text-processing. To use regular expressions effectively, one needs to understand them.

Everyone who uses `ls` is familiar with glob style patterns. Glob rules are useful in many situations, but are too limited for text processing. Regular expressions allow you to describe patterns in finer detail than glob rules.

A typical regular expression to match an e-mail address might look like this:

```
[a-z0-9_]+@[a-z0-9]+\.[a-z]+. 

```

If this looks weird, don't worry; it is really simple once you understand the concepts through this recipe.

# How to do it...

**Regular expressions** are composed of text fragments and symbols with special meanings. Using these, we can construct a regular expression to match any text. Regular expressions are the basis for many tools. This section describes regular expressions, but does not introduce the Linux/Unix tools that use them. Later recipes will describe the tools.

Regular expressions consist of one or more elements combined into a string. An element may be a position marker, an identifier, or a count modifier. A position marker anchors the regular expression to the beginning or end of the target string. An identifier defines one or more characters. The count modifier defines how many times an identifier may occur.

Before we look at some sample regular expressions, let's look at the rules.

# Position markers

A position marker anchors a regular expression to a position in the string. By default, any set of characters that match a regular expression can be used, regardless of position in the string.

| **regex** | **Description** | **Example** |
| --- | --- | --- |
| `^` | This specifies that the text that matches the regular expression must start at the beginning of the string | `^tux` matches a line that starts with `tux` |
| `$` | This specifies that the text that matches the regular expression must end with the last character in the target string | `tux$` matches a line that ends with `tux` |

# Identifiers

Identifiers are the basis of regular expressions. These define the characters that must be present (or absent) to match the regular expression.

| **regex** | **Description** | **Example** |
| --- | --- | --- |
| `A` character | The regular expression must match this letter. | `A` will match the letter A |
| `.` | This matches any one character. | `"Hack."` matches `Hack1`, `Hacki`, but not `Hack12` or `Hackil`; only one additional character matches |
| `[]` | This matches any one of the characters enclosed in the brackets. The enclosed characters may be a set or a range. | `coo[kl]` matches `cook` or `cool`; [0-9] matches any single digit |
| `[^]` | This matches any one of the characters except those that are enclosed in square brackets. The enclosed characters may be a set or a range. | `9[^01]` matches `92` and `93`, but not `91` and `90`; `A[^0-9]` matches an `A` followed by anything except a digit |

# Count modifiers

An Identifier may occur once, never, or many times. The Count Modifier defines how many times a pattern may appear.

| **regex** | **Description** | **Example** |
| --- | --- | --- |
| `?` | This means that the preceding item must match one or zero times | `colou?r` matches `color` or `colour`, but not `colouur` |
| `+` | This means that the preceding item must match one or more times | `Rollno-9+` matches `Rollno-99` and `Rollno-9`, but not `Rollno-` |
| `*` | This means that the preceding item must match zero or more times | `co*l` matches `cl`, `col`, and `coool` |
| `{n}` | This means that the preceding item must match n times | `[0-9]{3}` matches any three-digit number; `[0-9]{3}` can be expanded as `[0-9][0-9][0-9]` |
| `{n,}` | This specifies the minimum number of times the preceding item should match | `[0-9]{2,}` matches any number that is two digits or longer |
| `{n, m}` | This specifies the minimum and maximum number of times the preceding item should match | `[0-9]{2,5}` matches any number that has two digits to five digits |

# Other

Here are other characters that fine–tune how a regular expression will be parsed.

| `()` | This treats the terms enclosed as one entity | `ma(tri)?x` matches `max` or `matrix` |
| `&#124;` | This specifies alternation-; one of the items on either of side of `&#124;` should match | `Oct (1st &#124; 2nd)` matches `Oct 1st` or `Oct 2nd` |
| `\` | This is the escape character for escaping any of the special characters mentioned previously | `a\.b` matches `a.b`, but not `ajb`; it ignores the special meaning of `.` because of `\` |

For more details on the regular expression components available, you can refer to [http://www.linuxforu.com/2011/04/sed-explained-part-1/](http://www.linuxforu.com/2011/04/sed-explained-part-1/).

# There's more...

Let's see a few examples of regular expressions:

This regular expression would match any single word:

```
( +[a-zA-Z]+ +) 

```

The initial `+` characters say we need 1 or more spaces.

The `[a-zA-Z]` set is all upper– and lower–case letters. The following plus sign says we need at least one letter and can have more.

The final `+` characters say we need to terminate the word with one or more spaces.

This would not match the last word in a sentence. To match the last word in a sentence or the word before a comma, we write the expression like this:

```
( +[a-zA-Z]+[?,\.]? +) 

```

The `[?,\.]?` phrase means we might have a question mark, comma, or a period, but at most one. The period is escaped with a backslash because a bare period is a wildcard that will match anything.

It's easier to match an IP address. We know we'll have four three-digit numbers separated by periods.

The `[0-9]` phrase defines a number. The `{1,3}` phrase defines the count as being at least one digit and no more than three digits:

```
[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3} 

```

We can also define an IP address using the `[[:digit:]]` construct to define a number:

```
[[:digit:]]{1,3}\.[[:digit:]]{1,3}\.[[:digit:]]{1,3}\.[[:digit:]]{1,3} 

```

We know that an IP address is in the range of four integers (each from 0 to 255), separated by dots (for example, `192.168.0.2`).

This regex will match an IP address in the text being processed. However, it doesn't check for the validity of the address. For example, an IP address of the form `123.300.1.1` will be matched by the regex despite being an invalid IP.

# How it works...

Regular expressions are parsed by a complex state machine that tries to find the best match for a regular expression with a string of target text. That text can be the output of a pipe, a file, or even a string you type on the command line. If there are multiple ways to fulfill a regular expression, the engine will usually select the largest set of characters that match.

For example, given the string `this is a test` and a regular expression `s.*s`, the match will be `s is a tes`, not `s is`.

For more details on the regular expression components available, you can refer to [http://www.linuxforu.com/2011/04/sed-explained-part-1/](http://www.linuxforu.com/2011/04/sed-explained-part-1/).

# There's more...

The previous tables described the special meanings for characters used in regular expressions.

# Treatment of special characters

Regular expressions use some characters, such as `$`, `^`, `.`, `*`, `+`, `{`, and `}`, as special characters. But, what if we want to use these characters as normal text characters? Let's see an example of a regex, `a.txt`.

This will match the character `a`, followed by any character (due to the `.` character), which is then followed by the `txt` string. However, we want `.` to match a literal `.` instead of any character. In order to achieve this, we precede the character with a backward slash `\` (doing this is called escaping the character). This indicates that the regex wants to match the literal character rather than its special meaning. Hence, the final regex becomes `a\.txt`.

# Visualizing regular expressions

Regular expressions can be tough to understand. Fortunately, there are utilities available to help in visualizing regex. The page at [http://www.regexper.com](http://www.regexper.com) lets you enter a regular expression and creates a graph to help you understand it. Here is a screenshot describing a simple regular expression:

![](img/B05265_04_image.png)

# Searching and mining text inside a file with grep

If you forget where you left your keys, you've just got to search for them. If you forget what file has some information, the `grep` command will find it for you. This recipe will teach you how to locate files that contain patterns.

# How to do it...

The `grep` command is the magic Unix utility for searching text. It accepts regular expressions and can produce reports in various formats.

1.  Search `stdin` for lines that match a pattern:

```
        $ echo -e "this is a word\nnext line" | grep word 
 this is a word

```

2.  Search a single file for lines that contain a given pattern:

```
        $ grep pattern filename
 this is the line containing pattern

```

Alternatively, this performs the same search:

```
        $ grep "pattern" filename
 this is the line containing pattern

```

3.  Search multiple files for lines that match a pattern:

```
        $ grep "match_text" file1 file2 file3 ... 

```

4.  To highlight the matching pattern, use the `-color` option. While the option position does not matter, the convention is to place options first.

```
        $ grep -color=auto word filename
 this is the line containing word

```

5.  The `grep` command uses basic regular expressions by default. These are a subset of the rules described earlier. The `-E` option will cause `grep` to use the **Extended Regular Expression** syntax. The `egrep` command is a variant of `grep` that uses extended regular expression by default:

```
        $ grep -E "[a-z]+" filename

```

Or:

```
        $ egrep "[a-z]+" filename

```

6.  The `-o` option will report only the matching characters, not the entire line:

```
        $ echo this is a line. | egrep -o "[a-z]+\."
 line

```

7.  The `-v` option will print all lines, except those containing `match_pattern`:

```
        $ grep -v match_pattern file

```

The `-v` option added to `grep` inverts the match results.

8.  The `-c` option will count the number of lines in which the pattern appears:

```
        $ grep -c "text" filename
 10

```

It should be noted that `-c` counts the number of matching lines, not the number of times a match is made. Consider this example:

```
        $ echo -e "1 2 3 4\nhello\n5 6" | egrep  -c "[0-9]"
 2

```

Even though there are six matching items, `grep` reports `2`, since there are only two matching lines. Multiple matches in a single line are counted only once.

9.  To count the number of matching items in a file, use this trick:

```
        $ echo -e "1 2 3 4\nhello\n5 6" | egrep -o "[0-9]" | wc -l
 6

```

10.  The `-n` option will print the line number of the matching string:

```
        $ cat sample1.txt
 gnu is not unix
 linux is fun
 bash is art
 $ cat sample2.txt
 planetlinux
 $ grep linux -n sample1.txt
 2:linux is fun

```

Or

```
        $ cat sample1.txt | grep linux -n

```

If multiple files are used, the `-c` option will print the filename with the result:

```
        $ grep linux -n sample1.txt sample2.txt
 sample1.txt:2:linux is fun
 sample2.txt:2:planetlinux

```

11.  The `-b` option will print the offset of the line in which a match occurs. Adding the `-o` option will print the exact character or byte offset where the pattern matches:

```
        $ echo gnu is not unix | grep -b -o "not"
 7:not

```

Character positions are numbered from `0`, not from `1`.

12.  The `-l` option lists which files contain the pattern:

```
        $ grep -l linux sample1.txt sample2.txt
 sample1.txt
 sample2.txt

```

The inverse of the `-l` argument is `-L`. The `-L` argument returns a list of nonmatching files.

# There's more...

The `grep` command is one of the most versatile Linux/Unix commands. It also includes options to search through folders, select files to search, and more options for identifying patterns.

# Recursively searching many files

To recursively search for a text in files contained in a file hierarchy, use the following command:

```
    $ grep "text" . -R -n

```

In this command, `.` specifies the current directory.

The options `-R` and `-r` mean the same thing when used with `grep`.

Consider this example:

```
    $ cd src_dir
 $ grep "test_function()" . -R -n
 ./miscutils/test.c:16:test_function();

```

`test_function()` exists in line number 16 of `miscutils/test.c`. The `-R` option is particularly useful if you are searching for a phrase in a website or source code tree. It is equivalent to this command:

```
    $ find . -type f | xargs grep "test_function()"

```

# Ignoring case in patterns

The `-i` argument matches patterns without considering the uppercase or lowercase:

```
    $ echo hello world | grep -i "HELLO"
 hello

```

# grep by matching multiple patterns

The `-e` argument specifies multiple patterns for matching:

```
    $ grep -e "pattern1" -e "pattern2"

```

This will print the lines that contain either of the patterns and output one line for each match. Consider this example:

```
    $ echo this is a line of text | grep -o -e "this" -e "line"
 this
 line

```

Multiple patterns can be defined in a file. The `-f` option will read the file and use the line-separated patterns:

```
    $ grep -f pattern_filesource_filename

```

Consider the following example:

```
    $ cat pat_file
 hello
 cool

 $ echo hello this is cool | grep -f pat_file
 hello this is cool

```

# Including and excluding files in a grep search

`grep` can include or exclude files in which to search with wild card patterns.

To recursively search only for the `.c` and `.cpp` files, use the -include option:

```
    $ grep "main()" . -r  --include *.{c,cpp}

```

Note that `some{string1,string2,string3}` expands as `somestring1 somestring2 somestring3`.

Use the `-exclude` flag to exclude all `README` files from the search:

```
    $ grep "main()" . -r --exclude "README" 

```

The `--exclude-dir` option will exclude the named directories from the search:

```
    $ grep main . -r -exclude-dir CVS

```

To read a list of files to exclude from a file, use `--exclude-from FILE`.

# Using grep with xargs with the zero-byte suffix

The `xargs` command provides a list of command-line arguments to another command. When filenames are used as command-line arguments, use a zero-byte terminator for the filenames instead of the default space terminator. Filenames can contain space characters, which will be misinterpreted as name separators, causing a filename to be broken into two filenames (for example, `New file.txt` might be interpreted as two filenames `New` and `file.txt`). Using the zero-byte suffix option solves this problem. We use `xargs` to accept `stdin` text from commands such as `grep` and `find`. These commands can generate output with a zero-byte suffix. The `xargs` command will expect `0` byte termination when the `-0` flag is used.

Create some test files:

```
    $ echo "test" > file1
 $ echo "cool" > file2
 $ echo "test" > file3

```

The `-l` option tells `grep` to output only the filenames where a match occurs. The `-Z` option causes `grep` to use the zero-byte terminator (`\0`) for these files. These two options are frequently used together. The `-0` argument to `xargs` makes it read the input and separate filenames at the zero-byte terminator:

```
    $ grep "test" file* -lZ | xargs -0 rm

```

# Silent output for grep

Sometimes, instead of examining at the matched strings, we are only interested in whether there was a match or not. The quiet option (`-q`), causes `grep` to run silently and not generate any output. Instead, it runs the command and returns an exit status based on success or failure. The return status is `0` for success and nonzero for failure.

The `grep` command can be used in quiet mode, for testing whether a match text appears in a file or not:

```
#!/bin/bash  
#Filename: silent_grep.sh 
#Desc: Testing whether a file contain a text or not  

if [ $# -ne 2 ]; then 
  echo "Usage: $0 match_text filename" 
  exit 1 
fi 

match_text=$1  
filename=$2 
grep -q "$match_text" $filename 

if [ $? -eq 0 ]; then 
  echo "The text exists in the file" 
else 
  echo "Text does not exist in the file" 
fi 

```

The `silent_grep.sh` script accepts two command-line arguments, a match word (`Student`), and a file name (`student_data.txt`):

```
    $ ./silent_grep.sh Student student_data.txt 
 The text exists in the file 

```

# Printing lines before and after text matches

Context-based printing is one of the nice features of `grep`. When grep finds lines that match the pattern, it prints only the matching lines. We may need to see *n* lines before or after the matching line. The `-B` and `-A` options display lines before and after the match, respectively.

The `-A` option prints lines after a match:

```
    $ seq 10 | grep 5 -A 3
 5
 6
 7
 8

```

The `-B` option prints lines before the match:

```
    $ seq 10 | grep 5 -B 3
 2
 3
 4
 5

```

The `-A` and `-B` options can be used together, or the `-C` option can be used to print the same number of lines before and after the match:

```
    $ seq 10 | grep 5 -C 3
 2
 3
 4
 5
 6
 7
 8

```

If there are multiple matches, then each section is delimited by a `--` line:

```
    $ echo -e "a\nb\nc\na\nb\nc" | grep a -A 1
 a
 b
 --
 a
 b

```

# Cutting a file column-wise with cut

The cut command splits a file by column instead of lines. This is useful for processing files with fixed-width fields, **Comma Separated Values** (**CSV** files), or space delimited files such as the standard log files.

# How to do it...

The `cu`t command extracts data between character locations or columns. You can specify the delimiter that separates each column. In the `cut` terminology, each column is known as a **field**.

1.  The -f option defines the fields to extract:

```
        cut -f FIELD_LIST filename

```

`FIELD_LIST` is a list of columns that are to be displayed. The list consists of column numbers delimited by commas. Consider this example:

```
        $ cut -f 2,3 filename

```

Here, the second and the third columns are displayed.

2.  The `cut` command also reads input from `stdin`.

*Tab* is the default delimiter for fields. Lines without delimiters will be printed. The `-s` option will disable printing lines without delimiter characters. The following commands demonstrate extracting columns from a tab delimited file:

```
        $ cat student_data.txt 
 No  Name  Mark  Percent
 1  Sarath  45  90
 2  Alex  49  98
 3  Anu  45  90

 $ cut -f1 student_data.txt
 No 
 1 
 2 
 3 

```

3.  To extract multiple fields provide multiple field numbers separated by commas, using the following options:

```
        $ cut -f2,4 student_data.txt
 Name     Percent
 Sarath   90
 Alex     98
 Anu      90

```

4.  The `--complement` option will display all the fields except those defined by `-f`. This command displays all fields except `3`:

```
        $ cut -f3 --complement student_data.txt
 No  Name    Percent 
 1   Sarath  90
 2   Alex    98
 3   Anu     90

```

5.  The `-d` option will set the delimiter. The following command shows how to use `cut` with a colon-separated list:

```
        $ cat delimited_data.txt
 No;Name;Mark;Percent
 1;Sarath;45;90
 2;Alex;49;98
 3;Anu;45;90

 $ cut -f2 -d";" delimited_data.txt
 Name
 Sarath
 Alex
 Anu

```

# There's more

The `cut` command has more options to define the columns displayed.

# Specifying the range of characters or bytes as fields

A report with fixed-width columns will have varying numbers of spaces between the columns. You can't extract values based on field position, but you can extract them based on the character location. The `cut` command can select based on bytes or characters as well as fields.

It's unreasonable to enter every character position to extract, so cut accepts these notations as well as the comma-separated list:

| `N-` | From the *N^(th)* byte, character, or field, to the end of the line |
| `N-M` | From the *N^(th)* to *M^(th)* (included) byte, character, or field |
| `-M` | From the first to *M^(th)* (included) byte, character, or field |

We use the preceding notations to specify fields as a range of bytes, characters, or fields with the following options:

*   `-b` for bytes
*   `-c` for characters
*   `-f` for defining fields

Consider this example:

```
    $ cat range_fields.txt
 abcdefghijklmnopqrstuvwxyz
 abcdefghijklmnopqrstuvwxyz
 abcdefghijklmnopqrstuvwxyz
 abcdefghijklmnopqrstuvwxy

```

Display the second to fifth characters:

```
    $ cut -c2-5 range_fields.txt
 bcde
 bcde
 bcde
 bcde

```

Display the first two characters:

```
    $ cut -c -2  range_fields.txt
 ab
 ab
 ab
 ab

```

Replace `-c` with `-b` to count in bytes.

The `-output-delimiter` option specifies the output delimiter. This is particularly useful when displaying multiple sets of data:

```
    $ cut range_fields.txt -c1-3,6-9 --output-delimiter ","
 abc,fghi
 abc,fghi
 abc,fghi
 abc,fghi

```

# Using sed to perform text replacement

`sed` stands for **stream editor**. It's most commonly used for text replacement. This recipe covers many common `sed` techniques.

# How to do it...

The `sed` command can replace occurrences of a pattern with another string. The pattern can be a simple string or a regular expression:

```
 $ sed 's/pattern/replace_string/' file 

```

Alternatively, `sed` can read from `stdin`:

```
 $ cat file | sed 's/pattern/replace_string/'

```

If you use the `vi` editor, you will notice that the command to replace the text is very similar to the one discussed here. By default, `sed` only prints the substituted text, allowing it to be used in a pipe.

```
 $ cat /etc/passwd | cut -d : -f1,3 | sed 's/:/ - UID: /'
 root - UID: 0
 bin - UID: 1
 ...

```

1.  The `-I` option will cause `sed` to replace the original file with the modified data:

```
        $ sed -i 's/text/replace/' file

```

2.  The previous example replaces the first occurrence of the pattern in each line. The `-g` parameter will cause `sed` to replace every occurrence:

```
        $ sed 's/pattern/replace_string/g' file

```

The `/#g` option will replace from the *N^(th)* occurrence onwards:

```
        $ echo thisthisthisthis | sed 's/this/THIS/2g' 
 thisTHISTHISTHIS

 $ echo thisthisthisthis | sed 's/this/THIS/3g' 
 thisthisTHISTHIS

 $ echo thisthisthisthis | sed 's/this/THIS/4g' 
 thisthisthisTHIS

```

The `sed` command treats the character following `s` as the command delimiter. This allows us to change strings with a `/` character in them:

```
        sed 's:text:replace:g'
 sed 's|text|replace|g'

```

When the delimiter character appears inside the pattern, we have to escape it using the `\` prefix, as follows:

```
        sed 's|te\|xt|replace|g'

```

 `\|` is a delimiter appearing in the pattern replaced with escape.

# There's more...

The `sed` command supports regular expressions as the pattern to be replaced and has more options to control its behavior.

# Removing blank lines

Regular expression support makes it easy to remove blank lines. The `^$` regular expression defines a line with nothing between the beginning and end == a blank line. The final `/d` tells sed to delete the lines, rather than performing a substitution.

```
    $ sed '/^$/d' file

```

# Performing replacement directly in the file

When a filename is passed to `sed`, it usually prints to `stdout`. The `-I` option will cause `sed` to modify the contents of the file in place:

```
    $ sed 's/PATTERN/replacement/' -i filename

```

For example, replace all three-digit numbers with another specified number in a file, as follows:

```
    $ cat sed_data.txt
 11 abc 111 this 9 file contains 111 11 88 numbers 0000

 $ sed -i 's/\b[0-9]\{3\}\b/NUMBER/g' sed_data.txt
 $ cat sed_data.txt
 11 abc NUMBER this 9 file contains NUMBER 11 88 numbers 0000

```

The preceding one-liner replaces three-digit numbers only. `\b[0-9]\{3\}\b` is the regular expression used to match three-digit numbers. `[0-9]` is the range of digits from `0` to `9`. The `{3}` string defines the count of digits. The backslash is used to give a special meaning for `{` and `}` and `\b` stands for a blank, the word boundary marker.

It's a useful practice to first try the `sed` command without `-i` to make sure your regex is correct. After you are satisfied with the result, add the `-i` option to make changes to the file. Alternatively, you can use the following form of `sed`:

```
    sed -i .bak 's/abc/def/' file

```

In this case, `sed` will perform the replacement on the file and also create a file called `file.bak`, which contains the original contents.

# Matched string notation ()

The `&` symbol is the matched string. This value can be used in the replacement string:

```
    $ echo this is an example | sed 's/\w\+/[&]/g'
 [this] [is] [an] [example]

```

Here, the `\w\+` regex matches every word. Then, we replace it with `[&]`, which corresponds to the word that is matched.

# Substring match notation (\1)

`&` corresponds to the matched string for the given pattern. Parenthesized portions of a regular expression can be matched with `\#`:

```
    $ echo this is digit 7 in a number | sed 's/digit \([0-9]\)/\1/'
 this is 7 in a number

```

The preceding command replaces `digit 7` with `7`. The substring matched is `7`. `\(pattern\)` matches the substring. The pattern is enclosed in `()` and is escaped with backslashes. For the first substring match, the corresponding notation is `\1`, for the second, it is `\2`, and so on.

```
    $ echo seven EIGHT | sed 's/\([a-z]\+\) \([A-Z]\+\)/\2 \1/'
 EIGHT seven

```

`([a-z]\+\)` matches the first word and `\([A-Z]\+\)` matches the second word; `\1` and `\2` are used for referencing them. This type of referencing is called **back referencing**. In the replacement part, their order is changed as `\2 \1`, and hence, it appears in the reverse order.

# Combining multiple expressions

Multiple `sed` commands can be combined with pipes, patterns separated by semicolons, or the `-e PATTERN` option:

```
    sed 'expression' | sed 'expression'

```

The preceding command is equivalent to the following commands:

```
    $ sed 'expression; expression'

```

Or:

```
    $ sed -e 'expression' -e expression'

```

Consider these examples:

```
    $ echo abc | sed 's/a/A/' | sed 's/c/C/'
 AbC
 $ echo abc | sed 's/a/A/;s/c/C/'
 AbC
 $ echo abc | sed -e 's/a/A/' -e 's/c/C/'
 AbC

```

# Quoting

The `sed` expression is commonly quoted with single quotes. Double quotes can be used. The shell will expand double quotes before invoking sed. Using double quotes is useful when we want to use a variable string in a `sed` expression.

Consider this example:

```
 $ text=hello
 $ echo hello world | sed "s/$text/HELLO/" 
 HELLO world 

```

`$text` is evaluated as `hello`.

# Using awk for advanced text processing

The `awk` command processes data streams. It supports associative arrays, recursive functions, conditional statements, and more.

# Getting ready

The structure of an `awk` script is:

```
awk ' BEGIN{ print "start" } pattern { commands } END{ print "end"}' file

```

The `awk` command can also read from `stdin`.

An `awk` script includes up to three parts–:`BEGIN`, `END`, and a common statement block with the pattern match option. These are optional and any of them can be absent in the script.

Awk will process the file line by line. The commands following `BEGIN` will be evaluated before `<code>awk</code>` starts processing the file. Awk will process each line that matches PATTERN with the commands that follow PATTERN. Finally, after processing the entire file, `<CODE>awk</CODE>` will process the commands that follow `END`.

# How to do it...

Let's write a simple `awk` script enclosed in single quotes or double quotes:

```
    awk 'BEGIN { statements } { statements } END { end statements }'

```

Or:

```
    awk "BEGIN { statements } { statements } END { end statements }"

```

This command will report the number of lines in a file:

```
    $ awk 'BEGIN { i=0 } { i++ } END{ print i}' filename

```

Or:

```
    $ awk "BEGIN { i=0 } { i++ } END{ print i }" filename

```

# How it works...

The `awk` command processes arguments in the following order:

1.  First, it executes the commands in the `BEGIN { commands }` block.
2.  Next, `awk` reads one line from the file or `stdin`, and executes the `commands` block if the optional pattern is matched. It repeats this step until the end of file.
3.  When the end of the input stream is reached, it executes the `END { commands }` block.

The `BEGIN` block is executed before `awk` starts reading lines from the input stream. It is an optional block. The commands, such as variable initialization and printing the output header for an output table, are common comamnds in the `BEGIN` block.

The `END` block is similar to the `BEGIN` block. It gets executed when `awk` completes reading all the lines from the input stream. This is commonly printing results after analyzing all the lines.

The most important block holds the common commands with the pattern block. This block is also optional. If it is not provided, `{ print }` gets executed to print each line read. This block gets executed for each line read by `awk`. It is like a `while` loop, with statements to execute inside the body of the loop.

When a line is read, `awk` checks whether the pattern matches the line. The pattern can be a regular expression match, conditions, a range of lines, and so on. If the current line matches the pattern, `awk` executes the commands enclosed in `{ }`.

The pattern is optional. If it is not used, all lines are matched:

```
    $ echo -e "line1\nline2" | awk 'BEGIN{ print "Start" } { print } \   
        END{ print "End" } '
 Start
 line1
 line2
 End

```

When `print` is used without an argument, `awk` prints the current line.

The print command can accept arguments. These arguments are separated by commas, they are printed with a space delimiter. Double quotes are used as the concatenation operator.

Consider this example:

```
    $ echo | awk '{ var1="v1"; var2="v2"; var3="v3"; \
 print var1,var2,var3 ; }'

```

The preceding command will display this:

```
    v1 v2 v3

```

The `echo` command writes a single line into the standard output. Hence, the statements in the `{ }` block of `awk` are executed once. If the input to `awk` contains multiple lines, the commands in `awk` will be executed multiple times.

Concatenation is done with quoted strings:

```
    $ echo | awk '{ var1="v1"; var2="v2"; var3="v3"; \
 print var1 "-" var2 "-" var3 ; }'
 v1-v2-v3

```

`{ }` is like a block in a loop, iterating through each line of a file.

It's a common practice to place initial variable assignments such as `var=0;` in the `BEGIN` block. The `END{}` block contains commands to print the results.

# There's more...

The `awk` command differs from commands such as `grep`, `find`, and `tr`, in that it does more than a single function with options to change the behavior. The `awk` command is a program that interprets and executes programs and includes special variables just like the shell.

# Special variables

Some special variables that can be used with `awk` are as follows:

*   `NR`: This stands for the current record number, which corresponds to the current line number when `awk` uses lines as records.
*   `NF`: This stands for the number of fields, and corresponds to the number of fields in the current record being processed. The default field delimiter is a space.
*   `$0`: This is a variable that contains the text of the current record.
*   `$1`: This is a variable that holds the text of the first field.
*   `$2`: This is a variable that holds the text of the second field.

Consider this example:

```
    $ echo -e "line1 f2 f3\nline2 f4 f5\nline3 f6 f7" | \

 awk '{
 print "Line no:"NR",No of fields:"NF, "$0="$0,   
        "$1="$1,"$2="$2,"$3="$3 
 }' 
 Line no:1,No of fields:3 $0=line1 f2 f3 $1=line1 $2=f2 $3=f3 
 Line no:2,No of fields:3 $0=line2 f4 f5 $1=line2 $2=f4 $3=f5 
 Line no:3,No of fields:3 $0=line3 f6 f7 $1=line3 $2=f6 $3=f7

```

We can print the last field of a line as `print $NF`, the next to last as `$(NF-1)`, and so on.

`awk` also supports a `printf()` function with the same syntax as in C.

The following command prints the second and third field of every line:

```
    $awk '{ print $3,$2 }'  file

```

We can use NR to count the number of lines in a file:

```
    $ awk 'END{ print NR }' file

```

Here, we only use the `END` block. Awk updates `NR` as each line is read. When `awk` reaches the end of the file, NR will contain the last line number. You can sum up all the numbers from each line of `field 1` as follows:

```
    $ seq 5 | awk 'BEGIN{ sum=0; print "Summation:" } 
 { print $1"+"; sum+=$1 } END { print "=="; print sum }' 
 Summation: 
 1+ 
 2+ 
 3+ 
 4+ 
 5+ 
 ==
 15

```

# Passing an external variable to awk

Using the `-v` argument, we can pass external values other than `stdin` to `awk`, as follows:

```
    $ VAR=10000
 $ echo | awk -v VARIABLE=$VAR '{ print VARIABLE }'
 10000

```

There is a flexible alternate method to pass many variable values from outside `awk`. Consider the following example:

```
    $ var1="Variable1" ; var2="Variable2"
 $ echo | awk '{ print v1,v2 }' v1=$var1 v2=$var2
 Variable1 Variable2

```

When an input is given through a file rather than standard input, use the following command:

```
    $ awk '{ print v1,v2 }' v1=$var1 v2=$var2 filename

```

In the preceding method, variables are specified as key-value pairs, separated by a space, and `(v1=$var1 v2=$var2 )` as command arguments to `awk` soon after the `BEGIN`, `{ }`, and `END` blocks.

# Reading a line explicitly using getline

The `awk` program reads an entire file by default. The `getline` function will read one line. This can be used to read header information from a file in the `BEGIN` block and then process actual data in the main block.

The syntax is `getline var`. The `var` variable will contain the line. If `getline` is called without an argument, we can access the content of the line with `$0`, `$1`, and `$2`.

Consider this example:

```
    $ seq 5 | awk 'BEGIN { getline; print "Read ahead first line", $0 }     
    { print $0 }'
 Read ahead first line 1 
 2
 3
 4
 5

```

# Filtering lines processed by awk with filter patterns

We can specify conditions for lines to be processed:

```
 $ awk 'NR < 5' # first four lines
    $ awk 'NR==1,NR==4' #First four lines
    $ # Lines containing the pattern linux (we can specify regex)
    $ awk '/linux/' 
    $ # Lines not containing the pattern linux
    $ awk '!/linux/' 

```

# Setting delimiters for fields

By default, the delimiter for fields is a space. The `-F` option defines a different field delimiter.

```
    $ awk -F: '{ print $NF }' /etc/passwd

```

Or:

```
    awk 'BEGIN { FS=":" } { print $NF }' /etc/passwd

```

We can set the output field separator by setting `OFS="delimiter"` in the `BEGIN` block.

# Reading the command output from awk

Awk can invoke a command and read the output. Place a command string within quotes and use the vertical bar to pipe the output to `getline`:

```
    "command" | getline output ;

```

The following code reads a single line from `/etc/passwd` and displays the login name and home folder. It resets the field separator to a `:` in the `BEGIN` block and invokes `grep` in the main block.

```
    $ awk 'BEGIN {FS=":"} { "grep root /etc/passwd" | getline; \
        print $1,$6 }'
 root /root

```

# Associative arrays in Awk

Awk supports variables that contain a number or string and also supports associative arrays. An associative array is an array that's indexed by strings instead of numbers. You can recognize an associative array by the index within square brackets:

```
    arrayName[index]

```

An array can be assigned a value with the equal sign, just like simple user-defined variables:

```
    myarray[index]=value

```

# Using loop inside awk

Awk supports a numeric `for` loop with a syntax similar to `C`:

```
    for(i=0;i<10;i++) { print $i ; } 

```

Awk also supports a list style for loop that will display the contents of an array:

```
    for(i in array) { print array[i]; } 

```

The following example shows how to collect data into an array and then display it. This script reads lines from `/etc/password`, splits them into fields at the `:` markers, and creates an array of names in which the index is the login ID and the value is the user's name:

```
    $ awk 'BEGIN {FS=":"} {nam[$1]=$5} END {for {i in nam} \
        {print i,nam[i]}}' /etc/passwd
 root root
 ftp FTP User
 userj Joe User

```

# String manipulation functions in awk

The language of `awk` includes many built-in string manipulation functions:

*   `length(string)`: This returns the string length.
*   `index(string, search_string)`: This returns the position at which `search_string` is found in the string.
*   `split(string, array, delimiter)`: This populates an array with the strings created by splitting a string on the delimiter character.
*   `substr(string, start-position, end-position)`: This returns the substring of the string between the start and end character offsets.
*   `sub(regex, replacement_str, string)`: This replaces the first occurring regular expression match from the string with `replacment_str`.
*   `gsub(regex, replacment_str, string)`: This is like `sub()`, but it replaces every regular expression match.
*   `match(regex, string)`: This returns whether a regular expression (regex) match is found in the string. It returns a non-zero output if a match is found, otherwise it returns zero. Two special variables are associated with `match()`. They are `RSTART` and `RLENGTH`. The `RSTART` variable contains the position at which the regular expression match starts. The `RLENGTH` variable contains the length of the string matched by the regular expression.

# Finding the frequency of words used in a given file

Computers are good at counting. We frequently need to count items such as the number of sites sending us spam, the number of downloads different web pages get, or how often words are used in a piece of text. This recipes show how to calculate word usage in a piece of text. The techniques are also applicable to log files, database output, and more.

# Getting ready

We can use the associative arrays of `awk` to solve this problem in different ways. **Words** are alphabetic characters, delimited by space or a period. First, we should parse all the words in a given file and then the count of each word needs to be found. Words can be parsed using regex with tools such as `sed`, `awk`, or `grep`.

# How to do it...

We just explored the logic and ideas about the solution; now let's create the shell script as follows:

```
#!/bin/bash 
#Name: word_freq.sh 
#Desc: Find out frequency of words in a file 

if [ $# -ne 1 ]; 
then 
  echo "Usage: $0 filename"; 
  exit -1 
fi 

filename=$1 
egrep -o "\b[[:alpha:]]+\b" $filename | \
  awk '{ count[$0]++ }
    END {printf("%-14s%s\n","Word","Count") ;
      for(ind in count)
        { printf("%-14s%d\n",ind,count[ind]); 
        }
      }

```

The script will generate this output:

```
    $ ./word_freq.sh words.txt 
 Word          Count 
 used           1
 this             2 
 counting   1

```

# How it works...

The `egrep` command converts the text file into a stream of words, one word per line. The `\b[[:alpha:]]+\b` pattern matches each word and removes whitespace and punctuation. The `-o` option prints the matching character sequences as one word in each line.

The `awk` command counts each word. It executes the statements in the `{ }` block for each line, so we don't need a specific loop for doing that. The count is incremented by the `count[$0]++` command, in which `$0` is the current line and `count` is an associative array. After all the lines are processed, the `END{}` block prints the words and their count.

The body of this procedure can be modified using other tools we've looked at. We can merge capitalized and non-capitalized words into a single count with the `tr` command, and sort the output using the sort command, like this:

```
egrep -o "\b[[:alpha:]]+\b" $filename | tr [A=Z] [a-z] | \ 
  awk '{ count[$0]++ } 
    END{ printf("%-14s%s\n","Word","Count") ; 
      for(ind in count) 
        {  printf("%-14s%d\n",ind,count[ind]); 
        }
      }' | sort 

```

# See also

*   The *Using awk for advanced text processing* recipe in this chapter explains the `awk` command
*   The *Arrays and associative arrays* recipe in [Chapter 1](195d920d-33c2-41d6-bd33-37d75f9c37f1.xhtml), *Shell Something Out*, explains arrays in Bash

# Compressing or decompressing JavaScript

JavaScript is widely used in websites. While developing the JavaScript code, we use whitespaces, comments, and tabs for readability and maintenance of the code. This increases the file size, which slows page loading. Hence, most professional websites use compressed JavaScript speed page loading. This compression (also known as **minified JS**) is accomplished by removing the whitespace and newline characters. Once JavaScript is compressed, it can be decompressed by replacing enough whitespace and newline characters to make it readable. This recipe produces similar functionality in the shell.

# Getting ready

We are going to write a JavaScript compressor tool as well as a decompressing tool. Consider the following JavaScript:

```
    $ cat sample.js
 function sign_out()
 { 

 $("#loading").show(); 
 $.get("log_in",{logout:"True"},

 function(){ 
 window.location="";
 }); 
 }

```

Our script needs to perform these steps to compress the JavaScript:

1.  Remove newline and tab characters.
2.  Remove duplicated spaces.
3.  Replace comments that look like `/* content */`.

To decompress or to make the JavaScript more readable, we can use the following tasks:

*   Replace `;` with `;\n`
*   Replace `{` with `{\n`, and `}` with `\n}`

# How to do it...

Using these steps, we can use the following command chain:

```
    $ cat sample.js |  \
 tr -d '\n\t' |  tr -s ' ' \
 | sed 's:/\*.*\*/::g' \
 | sed 's/ \?\([{}();,:]\) \?/\1/g' 

```

The output is as follows:

```
    function sign_out(){$("#loading").show();$.get("log_in",  
    {logout:"True"},function(){window.location="";});}

```

The following decompression script makes the obfuscated code readable:

```
    $ cat obfuscated.txt | sed 's/;/;\n/g; s/{/{\n\n/g; s/}/\n\n}/g' 

```

Or:

```
    $ cat obfuscated.txt | sed 's/;/;\n/g' | sed 's/{/{\n\n/g' | sed   
    's/}/\n\n}/g'

```

There is a limitation in the script: that it even gets rid of extra spaces where their presence is intentional. For example, if you have a line like the following:                 `var a = "hello world"` 
The two spaces will be converted into one space. You can fix problems such as this using the pattern-matching tools we have discussed. Also, when dealing with a mission-critical JavaScript code, it is advised that you use well-established tools to do this.

# How it works...

The compression command performs the following tasks:

*   Removing the `\n` and `\t` characters:

```
 tr -d '\n\t'  

```

*   Removing extra spaces:

```
 tr -s ' ' or sed 's/[ ]\+/ /g' 

```

*   Removing comments:

```
 sed 's:/\*.*\*/::g' 

```

`:` is used as a `sed` delimiter to avoid the need to escape `/` since we need to use `/*` and `*/`.

In sed, `*` is escaped as `\*`.

 `.*` matches all the text in between `/*` and `*/`.

*   Removing all the spaces preceding and suffixing the `{`, `}`, `(`, `)`, `;`, `:`, and `,` characters:

```
 sed 's/ \?\([{}();,:]\) \?/\1/g' 

```

The preceding `sed` statement works like this:

*   `/ \?\([{}();,:]\) \?/` in the `sed` code is the match part, and `/\1 /g` is the replacement part.
*   `\([{}();,:]\)` is used to match any one character in the `[ { }( ) ; , : ]` set (spaces inserted for readability). `\(` and `\)` are group operators used to memorize the match and back reference in the replacement part. `(` and `)` are escaped to give them a special meaning as a group operator.`\?` precedes and follows the group operators to match the space character that may precede or follow any of the characters in the set.
*   In the replacement part, the match string (that is, the combination of `:`, a space (optional), a character from the set, and again an optional space) is replaced with the character matched. It uses a back reference to the character matched and memorized using the group operator `()`. Back-referenced characters refer to a group match using the `\1` symbol.

The decompression command works as follows:

*   `s/;/;\n/g` replaces `;` with `;\n`
*   `s/{/{\n\n/g` replaces `{` with `{\n\n`
*   `s/}/\n\n}/g` replaces `}` with `\n\n}`

# See also

*   The *Using sed to perform text replacement* recipe in this chapter explains the `sed` command
*   The *Translating with tr* recipe in [Chapter 2](36986eeb-141a-496a-a6b1-4f78f612c14e.xhtml), *Have a Good Command*, explains the `tr` command

# Merging multiple files as columns

The can command can be used to merge two files by row, one file after the other. Sometimes we need to merge two or more files side by side, joining the lines from file 1 with the lines from file 2.

# How to do it...

The `paste` command performs column-wise concatenation:

```
    $ paste file1 file2 file3 ...

```

Here is an example:

```
    $ cat file1.txt
 1
 2
 3
 4
 5
 $ cat file2.txt
 slynux
 gnu
 bash
 hack
 $ paste file1.txt file2.txt
 1 slynux
 2 gnu
 3 bash
 4 hack
 5

```

The default delimiter is tab. We can specify the delimiter with `-d`:

```
    $ paste file1.txt file2.txt -d ","
 1,slynux
 2,gnu
 3,bash
 4,hack
 5,

```

# See also

*   The *Cutting a file column-wise with cut* recipe in this chapter explains how to extract data from text files

# Printing the nth word or column in a file or line

We often need to extract a few columns of useful data from a file. For example, in a list of students ordered by their scores, we want to get the fourth highest scorer. This recipe shows how to do this.

# How to do it...

The `awk` command is frequently used for this task.

1.  To print the fifth column, use the following command:

```
        $ awk '{ print $5 }' filename

```

2.  We can print multiple columns and insert a custom string between the columns.

 The following command will print the permission and filename of each file in the  current directory:

```
        $ ls -l | awk '{ print $1 " :  " $8 }'
 -rw-r--r-- :  delimited_data.txt
 -rw-r--r-- :  obfuscated.txt
 -rw-r--r-- :  paste1.txt
 -rw-r--r-- :  paste2.txt

```

# See also

*   The *Using awk for advanced text processing* recipe in this chapter explains the `awk` command
*   The *Cutting a file column-wise with cut * recipe in this chapter explains how to extract data from text files

# Printing text between line numbers or patterns

We may need to print a selected portion of a file, either a range of line numbers or a range matched by a start and end pattern.

# Getting ready

`Awk`, `grep`, or `sed` will select lines to print, based on condition. It's simplest to use `grep` to print lines that include a pattern. Awk is the most versatile tool.

# How to do it...

To print the text between line numbers or patterns, follow these steps:

1.  Print the lines of a text in a range of line numbers, `M` to `N`:

```
        $ awk 'NR==M, NR==N' filename

```

 Awk can read from `stdin`:

```
        $ cat filename | awk 'NR==M, NR==N'

```

2.  Replace `M` and `N` with numbers:

```
        $ seq 100 | awk 'NR==4,NR==6' 
 4 
 5 
 6

```

3.  Print the lines of text between a `start_pattern` and `end_pattern`:

```
        $ awk '/start_pattern/, /end _pattern/' filename

```

 Consider this example:

```
        $ cat section.txt 
 line with pattern1 
 line with pattern2 
 line with pattern3 
 line end with pattern4 
 line with pattern5 

 $ awk '/pa.*3/, /end/' section.txt 
 line with pattern3 
 line end with pattern4

```

The patterns used in `awk` are regular expressions.

# See also

*   The *Using awk for advanced text processing* recipe in this chapter explains the `awk` command

# Printing lines in the reverse order

This recipe may not seem useful, but it can be used to emulate the stack data structure in Bash.

# Getting ready

The simplest way to accomplish this is with the `tac` command (the reverse of cat). The task can also be done with `awk`.

# How to do it...

We will first see how to do this with `tac`.

1.  The syntax of `tac` is as follows:

```
        tac file1 file2 ...

```

 The `tac` command can also read from `stdin`:

```
        $ seq 5 | tac
 5 
 4 
 3 
 2 
 1

```

 The default line separator for `tac` is `\n`. The -s option will redefine this:

```
        $ echo "1,2" | tac -s ,
 2
 1

```

2.  This `awk` script will print lines in the reverse order:

```
        seq 9 | \
          awk '{ lifo[NR]=$0 } \
            END { for(lno=NR;lno>-1;lno--) { print lifo[lno]; }
                }'

```

`\` in the shell script is used to break a single-line command sequence into multiple lines.

# How it works...

The `awk` script stores each of the lines into an associative array using the line number as the index (`NR` returns the line number). After reading all the lines, `awk` executes the `END` block. The `NR` variable is maintained by `awk`. It holds the current line number. When `awk` starts the END block, `NR` is the count of lines. Using `lno=NR` in the `{ }` block iterates from the last line number to `0`, to print the lines in reverse order.

# Parsing e-mail address and URLs from text

Parsing elements such as e-mail addresses and URLs is a common task. Regular expressions make finding these patterns easy.

# How to do it...

The regular expression pattern to match an e-mail address is as follows:

```
    [A-Za-z0-9._]+@[A-Za-z0-9.]+\.[a-zA-Z]{2,4} 

```

Consider the following example:

```
    $ cat url_email.txt 
 this is a line of text contains,<email> #slynux@slynux.com.    
    </email> and email address, blog "http://www.google.com",    
    test@yahoo.com dfdfdfdddfdf;cool.hacks@gmail.com<br />
 <a href="http://code.google.com"><h1>Heading</h1>

```

As we are using extended regular expressions (`+`, for instance), we should use `egrep`:

```
    $ egrep -o '[A-Za-z0-9._]+@[A-Za-z0-9.]+\.[a-zA-Z]{2,4}'    
    url_email.txt
 slynux@slynux.com 
 test@yahoo.com 
 cool.hacks@gmail.com

```

The `egrep` regex pattern for an HTTP URL is as follows:

```
    http://[a-zA-Z0-9\-\.]+\.[a-zA-Z]{2,4}

```

Consider this example:

```
    $ egrep -o "http://[a-zA-Z0-9.]+\.[a-zA-Z]{2,3}" url_email.txt 
 http://www.google.com 
 http://code.google.com

```

# How it works...

Regular expressions are easy to design part-by-part. In the e-mail regex, we all know that an e-mail address takes the `name@domain.some_2-4_letter_suffix` form. Writing this pattern in the regex language will look like this:

```
[A-Za-z0-9.]+@[A-Za-z0-9.]+\.[a-zA-Z]{2,4} 

```

`[A-Za-z0-9.]+` means we need one or more characters in the `[]` block (`+` means at least one, maybe more). This string is followed by an `@` character. Next, we will see the domain name, a string of letters or numbers, a period, and then 2-4 more letters. The `[A-Za-z0-9]+` pattern defines an alpha-numeric string. The `\.` pattern means that a literal period must appear. The `[a-zA-Z]{2,4}` pattern defines 2, 3, or 4 letters.

An HTTP URL is similar to an e-mail, but we don't need the `name@` match part of the e-mail regex:

```
http://[a-zA-Z0-9.]+\.[a-zA-Z]{2,3} 

```

# See also

*   The *Using sed to perform text replacement* recipe in this chapter explains the `sed` command
*   The *Using regular expressions* recipe in this chapter explains how to use regular expressions

# Removing a sentence in a file containing a word

Removing a sentence that contains a specific word is a simple task with regular expressions. This recipe demonstrates techniques for solving similar problems.

# Getting ready

`sed` is the best utility for making substitutions. This recipe uses `sed` to replace the matched sentence with a blank.

# How to do it...

Let's create a file with some text to carry out the substitutions. Consider this example:

```
    $ cat sentence.txt 
 Linux refers to the family of Unix-like computer operating systems   
    that use the Linux kernel. Linux can be installed on a wide variety   
    of computer hardware, ranging from mobile phones, tablet computers   
    and video game consoles, to mainframes and supercomputers. Linux is 
    predominantly known for its use in servers.

```

To remove the sentence containing the words `mobile phones`, use the following `sed` expression:

```
    $ sed 's/ [^.]*mobile phones[^.]*\.//g' sentence.txt
 Linux refers to the family of Unix-like computer operating systems   
    that use the Linux kernel. Linux is predominantly known for its use   
    in servers.

```

This recipe assumes that no sentence spans more than one line, for example, a sentence should always begin and end on the same line in the text.

# How it works...

The `sed` regex `'s/ [^.]*mobile phones[^.]*\.//g'` has the `'s/substitution_pattern/replacement_string/g` format. It replaces every occurrence of `substitution_pattern` with the replacement string.

The substitution pattern is the regex for a sentence. Every sentence begins with a space and ends with `.`. The regular expression must match the text in the format `"space" some text MATCH_STRING some text "dot"`. A sentence may contain any characters except a "dot", which is the delimiter. The `[^.]` pattern matches any character except a period`.` The `*` pattern defines any number of those characters. The `mobile phones` text match string is placed between the pattern for non-period characters. Every match sentence is replaced by `//` (nothing).

# See also

*   The *Using sed to perform text replacement* recipe in this chapter explains the `sed` command
*   The *Using regular expressions* recipe in this chapter explains how to use regular expressions

# Replacing a pattern with text in all the files in a directory

We often need to replace a particular text with a new text in every file in a directory. An example would be changing a common URI everywhere in a website's source directory.

# How to do it...

We can use `find` to locate the files to have text modified. We can use `sed` to do the actual replacement.

To replace the `Copyright` text with the `Copyleft` word in all `.cpp` files, use the following command:

```
 find . -name *.cpp -print0 | \
        xargs -I{} -0 sed -i 's/Copyright/Copyleft/g' {}

```

# How it works...

We use `find` on the current directory (`.`) to find the files with a `.cpp` suffix. The find command uses -`print0` to print a null separated list of files (use `-print0` when filenames have spaces in them). We pipe the list to `xargs`, which will pass the filenames to `sed`, which makes the modifications.

# There's more...

If you recall, `find` has an `-exec` option, which can be used to run a command on each of the files that match the search criteria. We can use this option to achieve the same effect or replace the text with a new one:

```
    $ find . -name *.cpp -exec sed -i 's/Copyright/Copyleft/g' \{\} \;

```

Or:

```
    $ find . -name *.cpp -exec sed -i 's/Copyright/Copyleft/g' \{\} \+

```

These commands perform the same function, but the first form will call `sed` once for every file, while the second form will combine multiple filenames and pass them together to `sed`.

# Text slicing and parameter operations

This recipe walks through some simple text-replacement techniques and parameter-expansion shorthands available in Bash. A few simple techniques can help avoid writing multiple lines of code.

# How to do it...

Let's get into the tasks.

Replace some text from a variable:

```
    $ var="This is a line of text"
 $ echo ${var/line/REPLACED}
 This is a REPLACED of text"

```

The `line` word is replaced with `REPLACED`.

We can produce a substring by specifying the start position and string length, using the following syntax:

```
    ${variable_name:start_position:length}

```

Print from the fifth character onwards:

```
    $ string=abcdefghijklmnopqrstuvwxyz
 $ echo ${string:4}
 efghijklmnopqrstuvwxyz

```

Print eight characters starting from the fifth character:

```
    $ echo ${string:4:8}
 efghijkl

```

The first character in a string is at position `0`. We can count from the last letter as `-1`. When `-1` is inside a parenthesis, `(-1)` is the index for the last letter:

```
    echo ${string:(-1)}
 z
 $ echo ${string:(-2):2}
 yz

```

# See also

*   The *Using sed to perform text replacement* recipe in this chapter explains other character manipulation tricks