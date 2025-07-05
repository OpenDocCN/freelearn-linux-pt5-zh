# Repository Management

In this chapter, we will cover the following recipes:

*   Creating a new git repository
*   Cloning a remote git repository
*   Adding and committing changes with git
*   Creating and merging branches with git
*   Sharing your work
*   Pushing a branch to a server
*   Retrieving the latest sources for the current branch
*   Checking the status of a git repository
*   Viewing git history
*   Finding bugs
*   Committing message ethics
*   Using fossil
*   Creating a new fossil repository
*   Cloning a remote fossil repository
*   Opening a fossil project
*   Adding and Committing Changes with Fossil
*   Using branches and forks with fossil
*   Sharing your work with fossil
*   Updating your local fossil repository
*   Checking the status of a fossil repository
*   Viewing fossil history

# Introduction

The more time you spend developing applications the more you come to appreciate software that tracks your revision history. A revision control system lets you create a sandbox for new approaches to problems, maintain multiple branches of released code, and provide a development history in the event of intellectual property disputes. Linux and Unix support many source code control systems ranging from the early and primitive SCCS and RCS to concurrent systems such as **CVS** and **SVN** and the modern distributed development systems such as **GIT** and **FOSSIL**.

The big advantage of Git and Fossil over older systems such as CVS and SVN is that a developer can use them without being connected to a network. Older systems such as CVS and RCS worked fine when you were at the office, but you could not check the new code or examine the old code while working remotely.

Git and Fossil are two different revision control systems with some similarities and some differences. Both support the distributed development model of revision control. Git provides source code control and has a number of add-on applications for more information while Fossil is a single executable that provides revision control, trouble tickets, a Wiki, web pages and technical notes.

Git is used for the Linux kernel development and has been adopted by many open source developers. Fossil was designed for the SQLite development team and is also widely used in both the open source and closed source communities.

Git is included with most Linux distributions. If it's not available on your system, you can install it with either yum (Redhat or SuSE) or apt-get (Debian or Ubuntu).

```
    $ sudo yum install git-all
 $ sudo apt-get install git-all

```

Fossil is available as source or executable from [http://www.fossil-scm.org](http://www.fossil-scm.org).

**Using Git**

The git system uses the `git` command with many subcommands to perform individual actions. We'll discuss git clone, git commit, git branch, and others.

To use git you need a code repository. You can either create one yourself (for your projects) or clone a remote repository.

# Creating a new git repository

If you are working on your own project, you will want to create your own repository. You can create the repository on your local system, or on a remote site such as GitHub.

# Getting ready

All projects in git need a master folder that holds the rest of the project files.

```
    $ mkdir MyProject
 $ cd MyProject

```

# How to do it...

The `git init` command creates the `.git` subfolder within your current working directory and initializes the files that configure `git`.

```
    $ git init

```

# How it works...

The `git init` command initializes a `git` repository for local use. If you want to allow remote users access this repository, you need to enable that with the `update-server-info` command:

```
    $ git update-server-info

```

# Cloning a remote git repository

If you intend to access someone else's project, either to contribute new code or just to use the project, you'll need to clone the code to your system.

You need to be online to clone a repository. Once you've copied the files to your system, you can commit new code, backtrack to older revisions, and so on. You can't send any new code changes upstream to the site you cloned from until you are online again.

# How to do it...

The `git clone` command copies files from the remote site to your local system. The remote site might be an anonymous repository such as GitHub, or a system where you need to log in with an account name and perhaps password.

Clone from a known remote site such as GitHub:

```
    $ git clone http://github.com/ProjectName

```

Clone from a login/password protected site (perhaps your own server):

```
    $ git clone clif@172.16.183.130:gitTest
 clif@172.16.183.130's password:

```

# Adding and committing changes with git

With distributed version control systems such as git, you do most of your work with your local copy of the repository. You can add new code, change code, test, revise, and finally commit the fully tested code. This encourages frequent small commits on your local repository and one large commit when the code is stable.

# How to do it...

The `git add` command adds a change in your working code to the staging area. It does not change the repository, it just marks this change as one to be included with the next commit:

```
    $ vim SomeFile.sh
 $ git add SomeFile.sh

```

Doing a `git add` after every edit session is a good policy if you want to be certain you don't accidently leave out a change when you commit your changes.

You can also add new files to your repository with the git add command:

```
    $ echo "my test file" >testfile.txt
 $ git add testfile.txt

```

Alternatively, you can add multiple files:

```
    $ git add *.c

```

The `git commit` command commits the changes to the repository:

```
    $ vim OtherFile.sh
 $ git add OtherFile.sh
 $ git commit

```

The `git commit` command will open the editor defined in your **EDITOR** shell variable and pre-populate like this:

```
    # Please enter the commit message for your changes. Lines starting   
    # with '#' will be ignored, and an empty message aborts the commit. 
 #
 # Committer: Clif Flynt <clif@cflynt.com>
 #
 # On branch branch1
 # Changes to be committed:
 #   (use "git reset HEAD <file>..." to unstage)
 #
 #       modified:   SomeFile.sh
 #       modified:   OtherFile.sh

```

After you enter a comment your changes will be saved in your local copy of the repository.

This does not push your changes to the main repository (perhaps `github`), but other developers can **pull** the new code from your repository if they have an account on your system.

You can shorten the add/commit events with the `-a` and `-m` arguments to commit:

*   `-a`: This adds the new code before committing
*   `-m`: This defines a message without going into the editor

```
    git commit -am "Add and Commit all modified files."

```

# Creating and merging branches with git

If you are maintaining an application you may need to return to an earlier branch to test. For instance, the bug you're fixing may have been around, but unreported, for a long time. You'll want to find when the bug was introduced to track down the code that introduced it. (Refer to `git bisect` in the *Finding bugs* recipe in this chapter.)

When you add new features, you should create a new branch to identify your changes. The project maintainer can then merge the new branch into the master branch after the new code is tested and validated. You can change and create new branches with the git's `checkout` subcommand.

# Getting ready...

Use `git init` or `git clone` to create the project on your system.

# How to do it...

To change to a previously defined branch:

```
    $ git checkout OldBranchName

```

# How it works...

The checkout subcommand examines the `.git` folder on your system and restores the snapshot associated with the desired branch.

Note that you cannot change to an existing branch if you have uncommitted changes in your current workspace.

You can create a new branch when you have uncommitted changes in the current workspaces. To create a new branch, use git checkout's `-b` option:

```
    $ git checkout -b MyBranchName
 Switched to a new branch 'MyBranchName'

```

This defines your current working branch to be `MyBranchName`. It sets a pointer to match `MyBranchName` to the previous branch. As you add and commit changes, the pointer will diverge further from the initial branch.

When you've tested the code in your new branch, you can merge the changes back into the branch you started from.

# There's more...

You can view the branches with the `git branch` command:

```
    $ git branch
 * MyBranchName
 master

```

The current branch is highlighted with an asterisk (`*`).

# Merging branches

After you've edited, added, tested, and committed, you'll want to merge your changes back into the initial branch.

# How to do it...

After you've created a new branch and added and committed your changes, change back to the original branch and use the `git merge` command to merge the changes in your new branch:

```
    $ git checkout originalBranch 
 $ git checkout -b modsToOriginalBranch
 # Edit, test
 $ git commit -a -m "Comment on modifications to originalBranch"
 $ git checkout originalBranch 
 $ git merge modsToOriginalBranch

```

# How it works...

The first `git checkout` command retrieves the snapshot for the starting branch. The second `git checkout` command marks your current working code as also being a new branch.

The `git commit` command (or commands) move the snapshot pointer for the new branch further and further away from the original branch. The third `git checkout` command restores your code to the initial state before you made your edits and commits.

The `git merge` command moves the snapshot pointer for the initial branch to the snapshot of the branch you are merging.

# There's more...

After you merge a branch, you may not need it any longer. The `-d` option will delete the branch:

```
    $ git branch -d MyBranchName

```

# Sharing your work

Git lets you work without connecting to the Internet. Eventually, you'll want to share your work.

There are two ways to do this, creating a patch or pushing your new code to the master repository.

**Making a patch...**

A patch file is a description of the changes that have been committed. Another developer can apply your patch files to their code to use your new code.

The format-patch command will collect your changes and create one or more patch files. The patch files will be named with a number, a description and `.patch`.

# How to do it...

The format-patch command requires an identifier to tell Git what the first patch should be. Git will create as many patch files as it needs to change code from what it was then to what it should be.

There are several ways to identify the starting snapshot. One common use for a set of patches is to submit the changes you've made to a given branch to the package maintainer.

For example, suppose you've created a new branch off the master for a new feature. When you've completed your testing, you may send a set of patch files to the project maintainer so they can validate your work and merge the new feature into the project.

The `format-patch` sub-command with the name of a parent branch will generate the patch file to create your current branch:

```
    $ git checkout master
 $ git checkout -b newFeature
 # Edits, adds and commits.
 $ git format-patch master
 0001-Patch-add-new-feature-to-menu.patch
 0002-Patch-support-new-feature-in-library.patch

```

Another common identifier is a git snapshot **SHA1**. Each git snapshot is identified by an SHA1 string.

You can view a log of all the commits in your repository with the `git log` command:

```
    $ git log
 commit 82567395cb97876e50084fd29c93ccd3dfc9e558
 Author: Clif Flynt <clif@example.com>
 Date:   Thu Dec 15 13:38:28 2016 -0500

 Fixed reported bug #1

 commit 721b3fee54e73fd9752e951d7c9163282dcd66b7
 Author: Clif Flynt <clif@example.com>
 Date:   Thu Dec 15 13:36:12 2016 -0500

 Created new feature

```

The `git format-patch` command with an SHA1 identifier looks like this:

```
    $ git format-patch SHA1

```

You can use a unique leading segment of the SHA1 identifier or the full, long string:

```
    $ git format-patch 721b
 $ git format-patch 721b3fee54e73fd9752e951d7c9163282dcd66b7

```

You can also identify a snapshot by its distance from your current location with a `-#` option.

This command will make a patch file for the most recent change to the master branch:

```
    $ git format-patch -1 master

```

This command will make a patch file for the two most recent changes to the `bleedingEdge` branch:

```
    $ git format-patch -2 bleedingEdge

```

**Applying a patch**

The `git apply` command applies a patch to your working code set. You'll have to check out the appropriate snapshot before running this command.

You can test that the patch is valid with the `--check` option.

If your environment is correct for this patch, there will be no return. If you don't have the correct branch checked out, the patch `-check` command will generate an error condition:

```
    $ git apply --check 0001-Patch-new-feature.patch
 error: patch failed: feature.txt:2
 error: feature.txt: patch does not apply

```

When the `--check` option does not generate an error message, use the `git apply` command to apply the patch:

```
    $ git apply 0001-Patch-new-feature.patch

```

# Pushing a branch to a server

Eventually, you'll want to share your new code with everyone, not just send patches to individuals.

The `git push` command will push a branch to the master.

# How to do it...

If you have a unique branch, it can always be pushed to the master repository:

```
    $ git push origin MyBranchName

```

If you've modified an existing branch, you may receive an error message as follows:

*   `remote: error`: Refusing to update checked out branch: `refs/heads/master`
*   `remote: error`: By default, updating the current branch in a non-bare repository

In this case, you need to push your changes to a new branch on the remote site:

```
    $ git push origin master:NewBranchName

```

You'll also need to alert the package maintainer to merge this branch into the master:

```
    # On remote
 $ git merge NewBranchName

```

Retrieving the latest sources for the current branch. If there are multiple developers on a project, you'll need to synchronize with the remote repository occasionally to retrieve data that's been pushed by other developers.

The `get fetch` and `git pull` commands will download data from the remote site to your local repository.

Update your repository without changing the working code.

The `git fetch` and `git pull` command will download new code but not modify your working code set.

```
    get fetch SITENAME

```

The site you cloned your repository from is named origin:

```
    $ get fetch origin

```

To fetch from another developer's repository, use the following command:

```
    $ get fetch Username@Address:Project

```

Update your repository and the working code.

The `git pull` command performs a fetch and then merges the changes into your current code. This will fail if there are conflicts you need to resolve:

```
    $ git pull origin
 $ git pull Username@Address:Project

```

# Checking the status of a git repository

After a concentrated development and debugging session you are likely to forget all the changes you've made.  The `>git status` command will remind you.

# How to do it...

The `git status` command reports the current status of your project. It will tell you what branch you are on, whether you have uncommitted changes and whether you are out of sync with the origin repository:

```
    $ git status
 # On branch master
 # Your branch is ahead of 'origin/master' by 1 commit.
 #
 # Changed but not updated:
 #   (use "git add <file>..." to update what will be committed)
 #   (use "git checkout -- <file>..." to discard changes in working   
     directory)
 #
 #modified:   newFeature.tcl

```

# How it works...

The previous recipe shows `git status` output when a change has been added and committed and one file was modified but not yet committed.

This line indicates that there has been a commit that hasn't been pushed:

```
# Your branch is ahead of 'origin/master' by 1 commit. 

```

Lines in this format report on files that have been modified, but not yet committed:

```
    #modified:   newFeature.tcl
 git config --global user.name "Your Name"
 git config --global user.email you@example.com

```

If the identity used for this commit is wrong, you can fix it with the following command:

```
    git commit --amend --author='Your Name <you@example.com>'
 1 files changed, 1 insertions(+), 0 deletions(-)
 create mode 100644 testfile.txt

```

# Viewing git history

Before you start working on a project, you should review what's been done. You may need to review what's been done recently to keep up with other developer's work.

The `git log` command generates a report to help you keep up with a project's changes.

# How to do it...

The `git log` command generates a report of SHA1 IDs, the author who committed that snapshot, the date it was committed, and the log message:

```
    $ git log
 commit fa9ef725fe47a34ab8b4488a38db446c6d664f3e
 Author: Clif Flynt <clif@noucorp.com>
 Date:   Fri Dec 16 20:58:40 2016 -0500
 Fixed bug # 1234

```

# Finding bugs

Even the best testing groups let bugs slip out into the field. When that happens, it's up to the developers to figure out what the bug is and how to fix it.

Git has tools to help.

Nobody deliberately creates bugs, so the problem is probably caused by fixing an old bug or adding a new feature.

If you can isolate the code that causes the issue, use the `git blame` command to find who committed the code that caused the problem and what the commit SHA code was.

# How to do it...

The `git blame` command returns a list of commit hash codes, author, date, and the first line of the commit message:

```
    $ git blame testGit.sh 
 d5f62aa1 (Flynt 2016-12-07 09:41:52 -0500 1) Created testGit.sh
 063d573b (Flynt 2016-12-07 09:47:19 -0500 2) Edited on master repo.
 2ca12fbf (Flynt 2016-12-07 10:03:47 -0500 3) Edit created remotely   
    and merged.

```

# There's more...

If you have a test that indicates the problem, but don't know the line of code that's at issue, you can use the `git bisect` command to find the commit that introduced the problem.

# How to do it...

The `git bisect` command requires two identifiers, one for the last known good code and one for the bad release. The bisect command will identify a revision midway between the good and bad for you to test.

After you test the code, you reset the good or bad pointer. If the test worked, reset the good pointer, if the test failed, reset the bad pointer.

Git will then check out a new snapshot midway between the new good and bad locations:

```
 # Pull the current (buggy) code into a git repository
 $ git checkout buggyBranch

 # Initialize git bisect.
 $ git bisect start

 # Mark the current commit as bad
 $ git bisect bad

 # Mark the last known good release tag
 # Git pulls a commit at the midpoint for testing.

 $ git bisect good v2.5
 Bisecting: 3 revisions left to test after this (roughly 2 steps)
 [6832085b8d358285d9b033cbc6a521a0ffa12f54] New Feature

 # Compile and test
 # Mark as good or bad
 # Git pulls next commit to test
 $ git bisect good
 Bisecting: 1 revision left to test after this (roughly 1 step)
 [2ca12fbf1487cbcd0447cf9a924cc5c19f0debf9] Merged. Merge branch   
    'branch1'

```

# How it works...

The `git bisect` command identifies the version of your code midway between a known good and known bad version. You can now build and test that version. After testing, rerun `git bisect` to declare that branch as good or bad. After the branch is declared, `git bisect` will identify a new version, halfway between the new good and bad markers.

# Tagging snapshots

Git supports tagging specific snapshots with a mnemonic string and an additional message. You can use the tags to make the development tree clearer with information such as *Merged in new memory management* or to mark specific snapshots along a branch. For example, you can use a tag to mark **release-1.0** and **release-1.1** along the **release-1** branch.

Git supports both lightweight tags (just tagging a snapshot) and tags with associated annotation.

Git tags are local only. `git push` will not push your tags by default. To send tags to the origin repository, you must include the -tags option:

```
    $ git push origin --tags

```

The `git tag` command has options to add, delete, and list tags.

# How to do it...

The `git tag` command with no argument will list the visible tags:

```
    $ git tag
 release-1.0
 release-1.0beta
 release-1.1

```

You can create a tag on your current checkout by adding a tag name:

```
    $ git tag ReleaseCandidate-1

```

You can add a tag to a previous commit by appending an SHA-1 identifier to the git tag command:

```
    $ git log --pretty=oneline
 72f76f89601e25a2bf5bce59551be4475ae78972 Initial checkin
 fecef725fe47a34ab8b4488a38db446c6d664f3e Added menu GUI
 ad606b8306d22f1175439e08d927419c73f4eaa9 Added menu functions
 773fa3a914615556d172163bbda74ef832651ed5 Initial action buttons

 $ git tag menuComplete ad606b

```

The `-a` option will attach annotation to a tag:

```
    $ git tag -a tagWithExplanation
 # git opens your editor to create the annotation

```

You can define the message on the command line with the `-m` option:

```
    $ git tag -a tagWithShortMessage -m "A short description"

```

The message will be displayed when you use the `git show` command:

```
    $ git show tagWithShortMessage

 tag tagWithShortmessage
 Tagger: Clif Flynt <clif@cflynt.com>
 Date:   Fri Dec 23 09:58:19 2016 -0500

 A short description
 ...

```

The `-d` option will delete a tag:

```
    $ git tag
 tag1
 tag2
 tag3 
 $ git tag -d tag2
 $ git tag
 tag2
 tag3F

```

# Committing message ethics

The commit message is free form text. It can be whatever you think is useful. However, there are comment conventions used in the Git community.

# How to do it...

*   Use 72 characters or less on each line. Use blank lines to separate paragraphs.
*   The first line should be 50 characters or less and summarize why this commit was made. It should be specific enough that someone reading just this line will understand what happened.
*   Don't write `Fix bug` or even `Fix bugzilla bug #1234`, write `Remove silly messages that appear each April 1`.

The following paragraphs describe details that will be important to someone following up on your work. Mention any global state variables your code uses, side effects, and so on. If there is a description of the problem you fixed, include the URL for the bug report or feature request.

# Using fossil

The fossil application is another distributed version control system. Like Git, it maintains a record of changes regardless of whether the developer has access to the master repository site. Unlike Git, fossil supports an auto-sync mode that will automatically push commits to the remote repository if it's accessible. If the remote site is not available at commit time, fossil saves the changes until the remote site becomes available.

Fossil differs from Git in several respects. The fossil repository is implemented in a single SQLite database instead of a set of folders as Git is implemented. The fossil application includes several other tools such as a web interface, a trouble-ticket system, and a wiki, while Git uses add-on applications to provide these services.

Like Git, the main interface to fossil is the `fossil` command with subcommands to perform specific actions like creating a new repository, cloning an existing repository, adding, committing files, and so on.

Fossil includes a help facility. The fossil help command will generate a list of supported commands, and `fossil help CMDNAME` will display a help page:

```
    $ fossil help
 Usage: fossil help COMMAND
 Common COMMANDs:  (use "fossil help -a|-all" for a complete list)
 add        cat        finfo      mv         revert     timeline 
 ...

```

# Getting ready

Fossil may not be installed on your system, and is not maintained by all repositories.
The definitive site for fossil is [h t t p ://w w w . f o s s i l - s c m . o r g](http://www.fossil-scm.org) .

# How to do it...

Download a copy of the fossil executable for your platform from [http://www.fossil-scm.org](http://www.fossil-scm.org) and move it to your `bin` folder.

# Creating a new fossil repository

Fossil is easy to set up and use for your own projects as well as existing projects that you join.

The `fossil new` and `fossil init` commands are identical. You can use either depending on your preference.

# How to do it...

The `fossil new` and `fossil init` commands create an empty fossil repository:

```
    $ fossil new myProject.fossil
 project-id: 855b0e1457da519d811442d81290b93bdc0869e2
 server-id:  6b7087bce49d9d906c7572faea47cb2d405d7f72
 admin-user: clif (initial password is "f8083e")

 $ fossil init myProject.fossil
 project-id: 91832f127d77dd523e108a9fb0ada24a5deceedd
 server-id:  8c717e7806a08ca2885ca0d62ebebec571fc6d86
 admin-user: clif (initial password is "ee884a")

```

# How it works...

The `fossil init` and fossil new commands are the same. They create a new empty repository database with the name you request. The `.fossil` suffix is not required, but it's a common convention.

# There's more...

Let us look at some more recipes:

# Web interface to fossil

The fossil web server provides either local or remote access to many features of the fossil system including configuration, trouble ticket management, a wiki, graphs of the commit history, and more.

The `fossil ui` command starts an http server and attempts to connect your local browser to the fossil server. By default, this interface connects you to the UI and you can perform any required task.

# How to do it...

```
    $ fossil ui
 Listening for HTTP requests on TCP port 8080

 #> fossil ui -P 80
 Listening for HTTP requests on TCP port 80

```

# Making a repository available to remote users

The fossil server command starts a fossil server that allows a remote user to clone your repository. By default, fossil allows anyone to clone a project. Disable the checkin, checkout, clone, and download zip capabilities on the `Admin/Users/Nobody` and `Admin/Users/Anonymous` pages to restrict access to only registered users.

The web interface for configuration is supported when running fossil server, but instead of being the default, you must log in using the credentials provided when you created the repository.

The fossil server can be started with a full path to the repository:

```
    $ fossil server /home/projects/projectOne.fossil

```

The fossil server can be started from a folder with the fossil repository without defining the repository:

```
    $ cd /home/projects
 $ ls
 projectOne.fossil

 $ fossil server
 Listening for HTTP requests on TCP port 8080

```

# Cloning a remote fossil repository

Because the fossil repository is contained in a single file, you can clone it simply by copying that file. You can send a fossil repository to another developer as an e-mail attachment, put it on a website, or copy it to a USB memory stick.

The fossil scrub command removes user and password information that the web server may require from the database. This step is recommended before you distribute copies of your repository.

# How to do it...

You can clone fossil from a site running fossil in the server mode with the fossil clone command. The fossil clone command distributes the version history, but not the users and password information:

```
    $ fossil clone http://RemoteSite:port projectName.fossil

```

# How it works...

The fossil clone command copies the repository from the site you've specified to a local file with a name you provide (in the example: `projectName.fossil`).

# Opening a fossil project

The fossil open command extracts the files from a repository. It's usually simplest to create a subfolder under the folder with the fossil repository to hold the project.

# How to do it...

Download the fossil repository:

```
    $ fossil clone http://example.com/ProjectName project.fossil

```

Make a new folder for your working directory and change to it:

```
    $ mkdir newFeature
 $ cd newFeature

```

Open the repository in your working folder:

```
    $ fossil open ../project.fossil

```

# How it works...

The fossil open command extracts all the folders, subfolders, and files that have been checked into the fossil repository.

# There's more...

You can use fossil open to extract specific revisions of the code in the repository. This example shows how to check out the 1.0 release to fix an old bug. Make a new folder for your working directory and change it as follows:

```
    $ mkdir fix_1.0_Bug
 $ cd fix_1.0_Bug

```

Open the repository in your working folder:

```
    $ fossil open ../project.fossil release_1.0

```

# Adding and committing changes with fossil

Once you've created a repository, you want to add and edit files. The fossil add command adds a new file to a repository and the fossil commit command commits changes to the repository. This is different from Git in which the `add` command marks changes to be added and the commit command actually does the commit.

# How to do it...

The next examples show how fossil behaves if you have not defined the `EDITOR` or `VISUAL` shell variables. If `EDITOR` or `VISUAL` are defined, fossil will use that editor instead of prompting you on the command line:

```
    $ echo "example" >example.txt
 $ fossil add example.txt
 ADDED  example.txt

 $ fossil commit
 # Enter a commit message for this check-in. Lines beginning with #   
    are ignored.
 #
 # user: clif
 # tags: trunk 
 #
 # ADDED      example.txt

 $ echo "Line 2" >>example.txt
 $ fossil commit
 # Enter a commit message for this check-in. Lines beginning with #    
    are ignored.
 #
 # user: clif
 # tags: trunk
 #
 # EDITED     example.txt

```

# There's more...

When you edit a file you only need to commit. By default, the commit will remember all your changes to the local repository. If auto-sync is enabled, the commit will also be pushed to the remote repository:

```
    $ vim example.txt
 $ vim otherExample.txt
 $ fossil commit
 # Enter a commit message for this check-in. Lines beginning with #    
    are ignored.
 #
 # user: clif
 # tags: trunk
 #
 # EDITED     example.txt, otherExample.txt

```

# Using branches and forks with fossil

In an ideal world, a development tree is a straight line with one revision following directly from the previous. In reality, developers frequently work from a stable code base and make changes that are then merged back into the mainline development.

The fossil system distinguishes temporary divergences from the mainline code (for example, a bug fix in your repository) from permanent divergences (like the 1.x release that gets only bug fixes, while new features go into 2.x).

The convention in fossil is to refer to intentional divergences as branches and unintentional divergences as forks. For example, you might create a branch for a new code you are developing, while trying to commit a change to a file after someone else has committed a change to that file would cause a fork unless you first update and resolve collisions.

Branches can be temporary or permanent. A temporary branch might be one you create while developing a new feature. A permanent branch is when you make a release that is intended to diverge from the mainline code.

Both temporary and permanent branches are managed with tags and properties.

When you create a fossil repository with fossil `init` or fossil new, it assigns the tag `trunk` to the tree.

The fossil branch command manages branches. There are subcommands to create new branches, list branches, and close branches.

# How to do it

1.  The first step in working with branches is to create one. The fossil branch new command creates a new branch. It can either create a branch based on your current checkout of the project, or you can create a branch at an earlier state of the project.
2.  The fossil branch new command will create a new branch from a given checkin:

```
        $ fossil branch new NewBranchName Basis-Id
 New branch: 9ae25e77317e509e420a51ffbc43c2b1ae4034da

```

3.  The `Basis-Id` is an identifier to tell fossil what code snapshot to branch from. There are several ways to define the `Basis-Id`. The most common of these are discussed in the next section.
4.  Note that you need to perform a checkout to update your working folder to the new branch:

```
        $ fossil checkout NewBranchName

```

# How it works...

`NewBranchName` is the name for your new branch. A convention is to name branches in a way that describes the modification being made. Branch names such as `localtime_fixes` or `bug_1234_fix` are common.

The `Basis-Id` is a string that identifies the node where the branch diverges. This can be the name of a branch if you are diverging from the head of a given branch.

The following commands show how to create a branch from the tip of a trunk:

```
    $ fossil branch new test_rework_parse_logic trunk
 New branch: 9ae25e77317e509e420a51ffbc43c2b1ae4034da

 $ fossil checkout test_rework_parse_logic 

```

The fossil commit command allows you to specify a new branch name at commit time with the `--branch` option:

```
    $ fossil checkout trunk

 # Make Changes

 $ fossil commit --branch test_rework_parse_logic

```

# There's more...

# Merging forks and branches

Branches and forks can both be merged back into their parent branch. The forks are considered temporary and should be merged as soon as the modifications are approved. Branches are considered permanent, but even these may be merged back into the mainline code.

The fossil merge command will merge a temporary fork into another branch.

# How to do it...

1.  To create a temporary fork and merge it back into an existing branch, you must first check out the branch you intend to work on:

```
        $ fossil checkout trunk

```

2.  Now you can edit and test. When you're satisfied with the new code, commit the new code onto a new branch. The `--branch` option creates a new branch if necessary and sets your current branch to the new `branch`:

```
        $ fossil commit --branch new_logic

```

3.  After the code has been tested and verified, you can merge it back into the appropriate branch by performing a checkout of the branch you want to merge into, then invoke the fossil merge command to schedule the merge, and finally commit the merge:

```
        $ fossil checkout trunk
 $ fossil merge new_logic
 $ fossil commit

```

4.  Fossil and Git behave slightly differently in this respect. The `git merge` command updates the repository, while the fossil merge command doesn't modify the repository until the merge is committed.

# Sharing your work with fossil

If you use multiple platforms for development, or if you work on someone else's project, you need to synchronize your local repository with the remote, master repository. Fossil has several ways to handle this.

# How to do it...

By default fossil runs in the `autosync` mode. In this mode, your commits are immediately propagated to the remote repository.

The `autosync` setting can be enabled and disabled with the fossil setting command:

```
    $ fossil setting autosync off
 $ fossil setting autosync on

```

When `autosync` is disabled (fossil is running in manual merge mode), you must use the fossil push command to send changes in your local repository to the remote:

```
    $ fossil push

```

# How it works...

The `push` command pushes all changes in your local repository to the remote repository. It does not modify any checked out code.

# Updating your local fossil repository

The flip side of pushing your work to the remote repository is updating your local repository. You'll need to do this if you do some development on your laptop while the main repository is on your companies server, or if you are working on a project with multiple people and you need to keep up to date on their new features.

# How to do it...

The fossil server does not push updates to remote repositories automatically. The `fossil pull` command will pull updates to your repository. It updates the repository, but does not change your working code:

```
    $ fossil pull

```

The `fossil checkout` command will update your working code if there were changes in the repository:

```
    $ fossil checkout

```

You can combine the pull and checkout subcommands with the `fossil update` command:

```
    $ fossil update
 UPDATE main.tcl
 -------------------------------------------------------------------   
    ------------
 updated-to:   47c85d29075b25aa0d61f39d56f61f72ac2aae67 2016-12-20    
    17:35:49 UTC
 tags:         trunk
 comment:      Ticket 1234abc workaround (user: clif)
 changes:      1 file modified.
 "fossil undo" is available to undo changes to the working checkout.

```

# Checking the status of a fossil repository

Before you start any new development, you should compare the state of your local repository to the master repository. You don't want to waste time writing code that conflicts with code that's been accepted.

# How to do it...

The `fossil status` command will report the current status of your project, whether you have uncommitted edits and whether your working code is at the tip:

```
    $ fossil status
 repository:   /home/clif/myProject/../myProject.fossil
 local-root:   /home/clif/myProject/
 config-db:    /home/clif/.fossil
 checkout:     47c85d29075b25aa0d61f39d56f61f72ac2aae67 2016-12-20     
    17:35:49 UTC
 parent:       f3c579cd47d383980770341e9c079a87d92b17db 2016-12-20     
    17:33:38 UTC 
 tags:         trunk
 comment:      Ticket 1234abc workaround (user: clif) 
 EDITED     main.tcl

```

If there has been a commit made to the branch you're working on since your last checkout, the status will include a line resembling the following:

```
    child:         abcdef123456789...  YYYY-MM-DD HH:MM::SS UTC

```

This indicates that there is a commit after your code. You will have to do a `fossil update` to bring your working copy of the code into sync before you can commit to the head of the branch. This may require you to fix conflicts by hand.

Note that fossil can only report the data in your local repository. If commits have been made but not pushed to the server and pulled into your local repository, they won't be displayed. You should invoke `fossil sync` before `fossil status` to confirm that your repository has all the latest information.

# Viewing fossil history

The `fossil server` and `fossil ui` commands start fossil's web server and let you view the history of check-ins and navigate through code via your favorite browser.

The timeline tab provides a tree-structured view of the branches, commits, and merges. The web interface supports viewing the source code associated with the commits and performing diffs between different versions.

# How to do it...

Start fossil in the UI mode. It will try to find your browser and open the main page. If that fails, you can point your browser to fossil:

```
    $ fossil ui
 Listening for HTTP requests on TCP port 8080

 $ konqueror 127.0.0.1:8080

```

![](img/image_06_001.png)

# Finding bugs

Fossil provides tools to help locate the commit where a bug was introduced:

| **Tools** | **Description** |
| `fossil diff` | This displays the difference between two revisions of a file |
| `fossil blame` | This generates a report showing the commit information for each line in a file |
| `fossil bisect` | This uses binary search to step between good and bad versions of an application |

# How to do it...

The `fossil diff` command has several options. When looking for the code that introduced a problem, we generally want to perform a diff on two versions of a file. The `-from` and `-to` options to `fossil diff` perform this action:

```
    $ fossil diff -from ID-1 -to ID-2FILENAME

```

`ID-1` and `ID-2` are identifiers used in the repository. They may be SHA-1 hashes, tags or dates, and so on. The `FILENAME` is the file that was committed to fossil.

For example, to find the difference between two revisions of `main.tcl` use the following command:

```
    $ fossil diff -from 47c85 -to 7a7e25 main.tcl

 Index: main.tcl
 ==================================================================
 --- main.tcl
 +++ main.tcl
 @@ -9,10 +9,5 @@

 set max 10
 set min 1
 + while {$x < $max} { 
 - for {set x $min} {$x < $max} {incr x} {
 -   process $x
 - }
 -

```

# There's more...

The differences between two revisions are useful, but it's more useful to see the entire file annotated to show when lines were added.

The `fossil blame` command generates an annotated listing of a file showing when lines were added:

```
$ fossil blame main.tcl
7806f43641 2016-12-18    clif: # main.tcl
06e155a6c2 2016-12-19    clif: # Clif Flynt
b2420ef6be 2016-12-19    clif: # Packt fossil Test Script
a387090833 2016-12-19    clif:
76074da03c 2016-12-20    clif: for {set i 0} {$i < 10} {incr
i} {
76074da03c 2016-12-20    clif: puts "Buy my book"
2204206a18 2016-12-20    clif: }
7a7e2580c4 2016-12-20    clif:

```

When you know that there's a problem in one version but not in another, you need to center in on the version where the problem was introduced.

The `fossil bisect` command provides support for this. It lets you define a good and bad version of the code and automatically checks out the version between those to be tested. You can then mark this version as good or bad and fossil will repeat the process. Fossil bisect also generates reports showing how many versions have been tested and how many need to be tested.

How to do it...

The `fossil bisect reset` command initializes the good and bad pointers. The `fossil bisect good` and `fossil bisect bad` commands mark versions as good or bad and check out the version of the code that's midway between the good and bad version:

```
$ fossil bisect reset
$ fossil bisect good 63e1e1
$ fossil bisect bad 47c85d
UPDATE main.tcl
-----------------------------------------------------------------------
updated-to:   f64ca30c29df0f985105409700992d54e 2016-12-20 17:05:44 UTC
tags:         trunk
comment:      Reworked flaky test. (user: clif)
changes:      1 file modified.
 "fossil undo" is available to undo changes to the working checkout.
 2 BAD     2016-12-20 17:35:49 47c85d29075b25aa
 3 CURRENT 2016-12-20 17:05:44 f64ca30c29df0f98
 1 GOOD    2016-12-19 23:03:22 63e1e1290f853d76

```

After testing the `f64ca` version of the code, you can mark it good or bad and `fossil bisect` will check out the next version for testing.

There's more...

The `fossil bisect status` command generates a report of the available versions and marks the tested versions:

```
$ fossil bisect status
2016-12-20 17:35:49 47c85d2907 BAD
2016-12-20 17:33:38 f3c579cd47
2016-12-20 17:30:03 c33415c255 CURRENT NEXT
2016-12-20 17:12:04 7a7e2580c4
2016-12-20 17:10:35 24edea3616
2016-12-20 17:05:44 f64ca30c29 GOOD

```

# Tagging snapshots

Every node in the fossil graph can have one or more tags attached to it. Tags can identify releases, branches, or just particular milestones that you may want to refer to. For example, you may want a release-1 branch with tags for release-1.0, release-1.1, and so on. A tag can be used with checkout or merge instead of using the SHA1 identifier.

Tags are implemented with the fossil tag command. Fossil supports several subcommands to add, cancel, find, and list tags.

The `fossil tag add` command creates a new tag:

```
    $ fossil tag add TagName Identifier

```

# How to do it...

The `TagName` is whatever you want to call the branch.

Identifier is an identifier for the node to be tagged. The identifier can be one of the following:

1.  **A branch name**: Tag the most recent commit on this branch
2.  **An SHA1 identifier**: Tag the commit with this SHA1 identifier
3.  **A datestamp (YYYY-MM-DD)**: Tag the commit just previous to this datestamp
4.  **A timestamp (YYYY-MM-DD HH:MM:SS)**: Tag the commit just previous to this timestamp

```
 # Tag the current tip of the trunk as release_1.0
        $ fossil add tag release_1.0 trunk

        # Tag the last commit on December 15 as beta_release_1
        $ fossil add tag beta_release_1 2016-12-16

```

# There's more...

A tag can be used as an identifier to create a fork or branch:

```
    $ fossil add tag newTag trunk
 $ fossil branch new newTagBranch newTag
 $ fossil checkout newTagBranch

```

A tag can create a branch with a commit and the `-branch` option:

```
    $ fossil add tag myNewTag 2016-12-21
 $ fossil checkout myNewTag
 # edit and change
 $ fossil commit -branch myNewTag

```