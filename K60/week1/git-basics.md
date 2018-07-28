# Git

## A Version Control System (VCS)

### About version control

Version control is a system that records changes to a file or set of files over time so that you can recall specific versions later.

#### Local Version Control Systems (LCSs)

Keep all changes to your files by storing it in a version database on your local machine.

#### Centralized Version Control Systems (CVCSs)

CVCSs are used for collaborating amongst developers of different systems. These systems have their own server that contains all the versioned files, and a number of client that check out files from that chentral place.

**Pros**

*   Everyone knows to a certain degree what everyone else on the project is doing.

*   Administrators have fine-grained control over who can do what.

**Cons**

*   The centralize server is a bottle neck of the whole system. If that server goes down, no body can collaborate at all or save versioned changes to anything they've working on.

*   If an error occurs at the server and no backups have been kept, you might loose basically everything.

#### Distributed Version Control Systems (DVCSs)

DVCSs, such as Git, help you avoid that certain problem with CVCSs. In Git, clients don't just check out the latest snapshot of the files; rather, they fully mirror the repository, including its full history. Thus, if any server dies, any of the client repositories can be copied back up to the server to restore it.

### Git advance properties.

*   High speed
*   Simple design
*   Strong support for non-linear development (thousands of parallel branches)
*   Fully distributed
*   Able to handle large projects efficiently

### Git in A Nutshell

#### Snapshots, Not Differences

The major difference between Git and other VCSs that is the way Git thinks about its data.

Other systems think of data as a set of chains, each chain store many versions of a file at each time.

Git thinks of data as  a consequence of snapshots. Every time you commit, Git basically takes a picture of what all your files look like at that moment and stores a reference to that snapshot. To be efficient, if files have not changed, Git doesn't store the file again, just a link to the previous identical file it has already stored.

#### Git Has Integrity

Everything in Git is check-summed before it is stored and is then referred to by that checksum. That means it is impossible to change the contents of any file without Git knowing about it.

#### Git Generally Only Add Data

Nearly every actions you make in Git only add data to the Git database, so that you can undo any action and restore any data without the fear of erasing it completely.

#### The Three States

*There are three state of a file under a Git project*:

*   Committed: Means that the data is safely stored in your local database.

*   Modified: Means that you have changed the file but have not committed it yet.

*   Staged: Means that you have marked a modified file in its current version to go into your next commit snapshot.

*This represents the three main sections of a Git project*:

*  _The git directory_ is where Git stores the metadata and object database for your project. This is what is copied when you clone a repositories from another computer.

*   _The working tree_ is a single checkout of one version of the project. These files are pulled out of the compressed database in the Git directory and placed on disk for you to use or modify.

*   _The staging area_ is a file, generally contained in your Git directory, that stores information about what will go into your next commit.

*Basic Git workflow*:

1.   You modify files in your working tree.

2.   You selectively stage just those changes you want to be part of your next commit, which adds only those changes to the staging area.

3.   You do a commit, which takes the files as they are in the staging area and stores that snapshot permanently to your Git directory.

*File states at different stages*

*   If a file is stored in the Git directory, it's considered committed.

*   If a file was added to the staging area and has been modified since then, it's staged.

*   If a file was changed since it was checked out but has not been staged, it is modified.

## Basic Usages

### Getting a Git Repository

You typically obtain a Git repository in one of two ways:

1.   Turning a local directory that is currently not under version control to a Git directory by this command.

    $ git init

2.   Cloning an existing repository

    $ git clone <url>


### Recording Changes to the Repository

Any file under Git can be either tracked and untracked. Git only knows about the modification of tracked files.

#### Checking the Status of Your Files

    $ git status

#### Tracking New Files

    $ git add path-name

If path-name is a directory, Git will add its files recursively.

#### Staging Modified Files

Changes in tracked files can be staged by `git add` command.

#### Ignoring Files

There some files that you don't want Git to automatically add or even show you as being untracked.

In such cases, you could create a file listing patterns to match them named .gitignore.

The rules for the patterns you can put in the .gitignore file are as follows:

*   Standard glob patterns work, and will be applied recursively throughout the entire working
tree.
*   You can start patterns with a forward slash (/) to avoid recursivity.
*   You can end patterns with a forward slash (/) to specify a directory.
*   You can negate a pattern by starting it with an exclamation point (!).

See more about common .gitignore configuration on this site [gitignore](https://github.com/github/gitignore).

#### Viewing Your Staged and Unstaged Changes

If you wanna know exactly what you changed on each file, you con use the command `git diff`.

To see what you've change but not yet staged, type `git diff` with no other arguments.

To see what you've staged that will go into your next commit, you can use `git diff --staged`. This command compares your staged changes to your last commit.

#### Committing Your Changes

    $ git commit -m 'Commit message.'

#### Skipping Staging Area

    $  git commit -a

to commit all tracked files without staging it.

#### Removing Files

To remove a file from Git, you have to remove it from your tracked files by the command

    $ git rm filename

and then commit again. After this operation, the file will be removed on the hard drive as well.

To tell Git not to track a file anymore, use --cached option:

    $ git rm --cached filename

#### Moving Files

Git doesn't explicitly track file movement, but still it has a `mv` command.

    $ git mv file_from file_to

### Viewing the Commit History

Simply using the command

    $ git log

Some useful options for `log` command:

Option  |  Description
--  |  --
-p  |  Show the patch introduced with each commit.
--stat  |   Show statistics for files modified in each commit.
--shortstat  |   Display only the changed/insertions/deletions line from the --stat command.
--name-only  |   Show the list of files modified after the commit information.
--name-status  |   Show the list of files affected with added/modified/deleted information as well.
--abbrev-commit  |   Show only the first few characters of the SHA-1 checksum instead of all 40.
--relative-date  |   Display the date in a relative format (for example, “2 weeks ago”) instead of using the full date format.
--graph  |   Display an ASCII graph of the branch and merge history beside the log output.
--pretty  |   Show commits in an alternate format. Options include oneline, short, full, fuller, and format (where you specify your own format).
--oneline  |   Shorthand for --pretty=oneline --abbrev-commit used together.

### Undoing Things

#### Unstaging a Staged File

    $ git reset HEAD filename

#### Unmodifying a Modified File

    $ git checkout -- filename

**CAUTION**: These two commands are dangerous, they can cause you to lose data. Only use them if you are absolutely certain about what you are doing.

### Working with Remotes

Remote repositories are versions of your project that are hosted somewhere on the Internet.

Collaborating with others involves managing these remote repositories and pushing and pulling data to and from them when you need to share work.

Managing remote repositories includes knowing how to add remote repositories, remove remotes that are no longer valid, manage various remote branches and define them as being tracked or not, and more.

#### Showing Your Remotes

By default, the remote repository you cloned from has the name 'origin'.

Show all remote servers' name and their URLs by this command

    $  git remote -v

#### Adding Remote Repositories

Adding new remote repository is pretty easy:

    $ git remote add rm-name Rommel

#### Fetching and Pulling from Your Remotes

To fetch all information on a remote repositories that you don't have yet:

    $ git fetch rm-name

Note that this command only downloads data to your local repository but doesn't merge it to your current branch yet, you have to do it manually.

If your current branch is set up to track a remote branch, you can simply use `$ git pull` to automatically fetch and merge that remote branch into your current branch. By default, the `git clone` command automatically set up your master branch to track the remote branch.

#### Pushing to Your Remotes

    $ git push rm-name your_branch

This command works only if you have write access to the server and if nobody has pushed in the meantime.

#### Inspecting a Remote

To see information of a particular remote, use the following command:

    $ git remote show rm-name

#### Renaming and Removing Remotes

    $ git remote rename old-name new-name
    $ git remote remove rm-name

### Tagging

Git has the ability to tag specific points in history as being important. Typically people use this functionality to mark release points (v1.0 and so on).

#### Listing Your Tags

    $ git tag

#### Creating Tags

Git supports two types of tags: _lightweight_ and _annotated_.

A lightweight tag is very much like a branch that doesn't change - it's just a pointer to a specific commit.

Annotated tags, however, are stored as full objects in the Git database. They are check-summed; contain the tagger name, email, and date; have a tagging message; and can be signed and verified with GNU Privacy Guard (GPG).

#### Annotated Tags

Create a annotated tag

    $ git tag -a tagname -m 'tagging message'

#### Lightweight Tags

Create a lightweight tag just by specifying its name, no other information required

    $ git tag lwtag

#### Tagging Later

You can also add a tag before a commit by specifying commit checksum at the end of the `git tag` command

    $ git tag -a tagname commit-checksum

#### Sharing Tags

By default, the `git push` command doesn't transfer tags to remote servers, you have to explicitly do it by yourself

    $ git push origin tagname

#### Checking out Tags

If you want to view the versions of files a tag is pointing to, you can do a git checkout

    $ git checkout tagname

### Git Aliases

You can create aliases to work with git faster and more conveniently

    $ git config --global alias.<new-command> <git-operation>

Whenever you type `new-command`, git will automatically execute `git-operation` instead.

## Git Branching

Branching means you diverge from the main line of development and continue to do work without messing with that main line. In many VCS tools, this is a somewhat expensive process, often requiring you to create a new copy of your source code directory, which can take a long time for large project.

Git is totally different. The way it branches is incredibly lightweight, making branching operations nearly instantaneous, and switching back and forth between branches generally just as fast.

Unlike many other VCSs, Git encourages workflows that branch and merge often, even multiple times in a day. Understanding and mastering this feature gives you a powerful and unique tool an can entirely change the way that you develop.

### Branches in a Nutshell

When you make a commit, Git stores a commit object that contains a pointer to the snapshot of the content you staged. Each commit object has a number of pointers that point to its parent: zero for initial commit, one for normal commit, and multiple for commit that results from a merge of two or more branches.

A branch in Git is simply a lightweight movable pointer to one of your commits. The default branch name in Git is `master`.

#### Creating a New Branch

Create a new branch by using `git branch` command.

    $ git branch testing

What happens here is you make a new pointer called testing point to the commit you're currently on.

How does Git know what branch you're currently on? It keeps a special pointer called HEAD to do so.

The `git branch` command only _created_ a new branch - it didn't switch to that branch.

#### Switching Branches

To switch to an existing branch, run the `git checkout` command

   $ git checkout testing

This move HEAD to point to the testing branch.

When you move to a branch that's still holding an older version of your project, your files will be reverted back to the snapshot of that branch. This also means the changes you make from this point forward will diverge from an older version of the project. The [Git book](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell) has a very nice visualization of this.

You can also see this with a `git log` command

    $ git log --oneline --decorate --graph --all

### Basic Branching and Merging

Here's an example of a workflow that you might use in the real world.

1.   Do some work on a website.
2.   Create a branch for a new story you're working on.
3.   Do some work on that branch.

At this stage, you'll receive a call that another issue is critical and you need a hotfix. You'll do the following:

1.   Switch to your production branch.
2.   Create a branch to add the hotfix.
3.   After it's tested, merge the hotfix branch, and push to production.
4.   Switch back to your original story and continue working.

#### Basic Branching

To create a new branch and switch to it the same time, use `git checkout` command with the `-b` option:

    $ git checkout -b new-branch

After you finish working on the `new-branch` and had some commit to it, you might want to merge it into the `master` branch. First, switch to the `master` branch:

   $ git checkout master

Then merge `master` with `new-branch`:

    $ git merge new-branch

Now, you don't need the `new-branch` anymore because your `master` has already pointed to its latest snapshot, delete that branch:

    $ git branch -d new-branch

#### Basic Merging

If you merge branch X that is currently pointing to snapshot A into branch Y pointing to snapshot B and B is an ancestor of A, Git just makes the Y pointer points to A and then done, pretty straight forward.

In the other case, if B is not an ancestor of A, Git does a simple three-way merge, using the two snapshots above and the common ancestor of the two, result in a new snapshot. Then Git automatically creates a commit to that snapshot on branch Y, this commit is referred to as a _merge commit_, and is special in that it has more than one parent.

#### Basic Merge Conflicts

Occasionally, merging two branch doesn't go smoothly. If you changed the same part of the same file differently in the two branches, Git won't be able to merge them cleanly.

To see which files are unmerged at any point after a merge conflict, you can run `git status`. Anything that has merge conflicts and hasn't been resolved is listed as _unmerged_.

Git adds standard conflict-resolution markers to the files that have conflicts, so you can open them manually and resolve those conflicts.

After resolving conflicts and you're happy with that, type `git commit` to finalize the merge commit. Please consider add some note about the merging operation to help other people understand how you resolved the merge.

### Branch Management

Some useful `git branch` option:

*   no args: List all branches, the branch you're in is marked with a '*' character.
*   -v: See the last commit of each branch
*   --merged: See all the branches are already merged to you're on.
*   --no-merged: See all the branches that contain work you haven't yet merged in.

### Branching Workflows

#### Long-Running Branches

This is the kind of workflow that's based on stablity.

*   Only entirely stable code is in the `master` branch - possibly only code that has been or will be released.
*   Some parallel branches named `develop` or `next` to work in or to test code. Whenever these branches are stable, it will be merged into `master`.
*   There can be several levels of stability, each corresponds to a parallel level. When ever a branch reach a more stable level, they're merged into the branch above them.

This approach is helpful when you're dealing with very latge or complex projects.

#### Topic Branches

Topic branches, however, are useful in projects of any size. A topic branch is a short-lived branch that you create and use for a single particular feature or related work. This requires alot of merging and it works well on Git because Git is excellent in merging files.

### Remote Branches

Remote references are pointers to your remote repositories, including branches, tags and so on.

Get a full list of remote references:

    git ls-remote [remote]

or

    git remote show [remote]

Remote-tracking branches take the form <remote>/<branch>, `origin/master` for example.

After cloning a repository, your master branch and the remote's master branch are pointing to the same commit. But this stage won't exist for so long, either you will make some new commit to your local master branch or someone else will push a new version to the remote's master branch later, so that the two branches will move forward differntly.

To synchronize your work, run a `git fetch origin` command. This command looks up which server 'origin' is, fetches any data from it that you don't have yet, and updates your local database, moving your origin/master pointer to its new position. Of course this won't effect your local master branch you're in.

#### Pushing

When you want to share a branch with the world, you need to push it up to a remote that you have write access to:

    $ git push <remote> <branch>

Note that after fetching a remote branch, you don't have an equivalent local branch of it, just a remote-name/branch-name that you cannot change. You can use `git checkout` command to do so:

    $ git checkout -b new-branch remote/new-branch

#### Tracking Branches

Tracking branch should have the same name as the remote branch, such as master - origin/master when you clone a repository.

Tracking branches have a direct relationship with the "upstream branch", if you type `git pull` in a tracking branch, it knows which server to fetch from and which branch to merge in.

You can also set up tracking branches by the `git checkout` command above, or set a upstream branch to an existed branch by run this command from that branch:

    $ git branch -u remote/branch

To see which tracking branch is tracking which remote branch, use this command:

    $ git branch -vv

#### Pulling

Fetching only update remote/branch, not the tracking branch. If you want to update your tracking branch automatically, use `git pull` instead.

#### Deleting Remote Branches

If you're done with a remote branch, say your collaborators have finished their job and you successfully merge it to your remote's master branch, you can delete it by the following command:

    $ git push origin --delete remote-branch

### Rebasing

`rebase` is an other way to integrate changes from one branch into another, beside `merge`.

#### The Basic Rebase

Say you want to integrate two branch: `experiment` pointing to C3 and `master` pointing to C4, and you're in `master` branch.

`merge` does a three-way merge that results in a new commit C5 which points to both C3 and C4, `master` then points to C5.

`rebase` apply changes in C4 on top of C3, result in a new C4' commit. This operations include these steps:

*   Checkout to the branch `experiment`

    $ git checkout experiment

*   Rebase `experiment` to `master`: Replay `master` for the most common ancestor of these two, then apply changes that `experiment` made on top of that result

    $ git rebase master

*   Merge `experiment` into `master` to move the pointer to the new position

    $ git checkout master; git merge experiment

These two procedure works exactly the same but rebasing keeps your history looks cleaner (more linear).

#### More about Rebases

See the example at page 92, the [Git book](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell)

#### The Perils of Rebasing

**Do not rebase commits that exist outside your repository**
#### Rebase vs. Merge



## Git on Server

## Distributed Workflows

## GitHub Hosting Service

## Advanced Git Commands

## Customizing Git

## Git and Other VCSs

## Git Internals
