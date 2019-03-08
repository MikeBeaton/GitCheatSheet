# Git Cheat Sheet

To create a brand new repository from existing code (also includes how to make SVN and Git co-exist peacefully):

    cd project/root/dir (path to just inside pre-existing root of project)
    git init (sets Git up to track this directory and all subdirectories, but with nothing added yet)
    (Copy in the .gitignore file,  to avoid checking in the junk that you don't want)
    (Add .svn/ to .gitignore to make Git ignore SVN files, if not already present and you are co-existing with SVN)
    (If you have Unix .js files and Windows .cs files, then you will want to set: `git config --global core.autocrlf false` to make Git not try to normalise these)
    git add * (add everything to staging)
    git add .gitignore (add this file to staging too)
    git commit -m 'Initial commit' (Put everything from staging into your local Git, so it's no longer empty)
    (Now create the remote repository where you want the code to live, if you haven't already)
    (Make sure that the user you are committing as has write permissions on the repository, otherwise it will look like it is just not there)
    git remote add origin [git-repo-url]
    git push --set-upstream|-u origin master

You can make a git repository co-exist with an SVN repository by adding `.git` and `.gitignore` the the `svn:ignore` properties, to make SVN ignore Git files. But this is not what you want to do!

Use:

	git svn clone --stdlayout --authors-file=[authors-file] [repo-url] [git-folder]

to create a git version of an svn repository. Get the author name and emaill address right in `[authors-file]`, if you want to link to real Git users. After which:

	git svn fetch

will fetch any further changes made in SVN.

Get a copy of a remote repository:

    cd /c/Projects/
    git clone https://github.com/Bmju/Massive
    cd Massive/

Show my branches:

    git branch
    git branch -a
    git branch -r
    git branch -vv
    git branch -r -vv

Put all pending changes on a new branch ("If you haven't already committed your changes, just use git checkout to move to the new branch and then commit them normally - changes to files are not tied to a particular branch until you commit them.")

    git checkout -b MyTest

Then once your new branch is ready, to put it up on the server:

    git push -u origin MyTest

Add an upstream repository:

    git remote add upstream https://github.com/FransBouma/Massive
    git fetch upstream

Show contents of a remote:

    git remote show origin
    git remote show upstream

(For more/different detail use `git ls-remote origin`, `git ls-remote upstream`.)

Show pending changes; for a given file; show the actual diff:

    git status
    git status tests/Oracle/App.config
    git diff tests/Oracle/App.config

Add a branch (but see below about tracking branches):

    git branch piggle

Remove a branch:

    git branch -d piggle

Interactively rebase my changes from BranchA onto a NEW BranchB:

    [create BranchB - as above]
    git checkout BranchA
    git rebase -i --onto BranchB

NB Nothing in git branches has to relate any local branch to any remote branch (they can have the same name, and NOT be linked, or different names and be linked). However...

https://git-scm.com/book/en/v2/Git-Branching-Remote-Branches

"Checking out a local branch from a remote-tracking branch automatically creates what is called a “tracking branch” (and the branch it tracks is called an “upstream branch”)". Thus, ASSUMING YOU ALREADY HAVE A REMOTE-TRACKING BRANCH ON THE REMOTE BRANCH, then

    git checkout -b serverfix origin/serverfix
    git checkout --track origin/serverfix
    git checkout serverfix
    
all do the same, and create a tracking branch. (NB the local tracking branch DOES NOT have to have the same name as the remote branch which it is tracking - in the first syntax you could name the local branch sf, for example.) (Also, the last syntax only creates a tracking branch "If the branch name you’re trying to checkout (a) doesn’t exist and (b) exactly matches a name on only one remote".)

When you fetch a remote, you automatically get and update it's remote-tracking branches, except that if you want to prune any which have been deleted, you need to add -p:

    git fetch -p
    git fetch -p origin
    git fetch -p upstream
    
NB remote-tracking branches ARE NOT tracking branches.
You DO NOT automatically get a local branch for every branch in a remote which you clone!

It sounds like it should be difficult, but it is very easy to change the upstream of a (the current) local branch:

    git branch -u origin/serverfix

It is also ridiculously easy, even though it sounds like it should be impossible, to rebase a branch onto basically anything:

    git rebase upstream/v2.0

NB This works just fine, even on a local branch which is tracking origin, and NOT upstream!

Push changes on current branch back to what it is tracking (usually you will be tracking origin, and will not even have permissions to do this, if you are tracking upstream):

    git push

Force push the changes back if you've been doing rebasing, etc:

    git push -f

May also be useful to see what your branches are pointing at (but does NOT include everything visible from the above commands):

    notepad++ .git/config

The two built-in git gui programs
(use & at then end to start a program from the command line and return straight to the command line):

    gitk &
    git-gui &

gitk is useful once you start to understand what git is doing; it shows the branch and revision graph for the project
NB You can show the same graph (even coloured) from the command line:

    git log --graph --oneline --decorate --all

User name and email:

    git config --global user.name
    git config --global user.email

Save pending changes where I can get them back again:

    git stash show
    git stash
    git stash show
    
Switch to tracking new (renamed) origin:

    git remote set-url origin https://github.com/MikeBeaton/Massive

Make a branch from the upstream and cherry pick some commits onto it:
(This is not quite right yet, if you look in .git/config then the branch this makes has upstream as its remote, which makes git not keen to
push it, and not sure where to push it. I want to have origin as the remote, so I need to work out the sequence to do this.
I could rebase all my changes onto the upstream, then make a branch starting at the point just before my changes and cherry pick on to
this - but I don't really want to, I just want to create a new branch in my Github which has a name I choose, and is the current state
of the master.)

    git checkout -b wherevarfix upstream/v2.0
    git cherry-pick 88be108
    git push origin wherevarfix

Abandon all local changes
(this does not (always) work! it does not fail silently, but still leaves unsaved changes which prevent commits;
git stash; git stash drop; works):

    git checkout -- .

(I think it should be `git checkout -- *`, shouldn't it?)

Amend the last commit:

    git commit --amend

Interactively squash the last n changes:

    git rebase -i HEAD~2

Rebase all the way to the first commit (which can't be done with the above) (https://stackoverflow.com/a/2309391/795690):

    git rebase -i --root

Split a commit:

    git rebase -i HEAD~1 (or as far back as needed to include the one to split)
    [Mark the commit you want to split with the action "edit"]
    [When it comes to editing that commit, execute git reset HEAD^. The effect is that the HEAD is rewound by one, and the index follows suit. However, the working tree stays the same. Meaning that the outstanding changes in that commit are listed as outstanding, ready for you to commit in pieces.]
    [Now add the changes to the index that you want to have in the first commit. This may just be a git stage.]
    [Commit the now-current index with whatever  message is appropriate.]
    [Repeat the last two steps until you have committed all the changes.]
    git rebase --continue.

Set editor:

    git config --global core.editor nano

Show what you've been doing:

    git reflog

Just in case it's not obvious this isn't re-flog - a dead horse, for instance - but reference log.

Split a commit:

    git rebase -i HEAD~1
    [Mark it as 'edit']
    git reset HEAD~
    [Commit the pieces individually in the usual way, producing as many commits as you need]
    git rebase --continue

To push local repository to new remote, first add empty remote repository, then:

    git remote add origin https://github.com/GRP-IT/TigerMVC

Then to push to a different, named upstream repository and set as new upstream (will work on new empty upstream):

    git push -u origin master

(There's another way to stuff everything into an empty repository, isn't there?)

Push to a new remote branch:

    git push public HEAD:test

Create a new local branch:

    git checkout -b test

And push it to the upstream:

    git push --set-upstream origin test

Clean/delete unnecessary directories and files and git ignored files:

    git clean -dfx

Move a branch back to an earlier commit:

    git reset --hard 2f005bc

Move a branch back to an earlier commit but keep your changes as unstaged commits:

    git reset b47b3c4

Reset/restore/abandon local changes to a single file:

    git checkout -- tests/SqlServer/AsyncReadTests.cs

http://stackoverflow.com/a/34455483/795690
To stage all manually deleted files you can use:

    git rm $(git ls-files --deleted)

To add an alias to this command as git rm-deleted, run:

    git config --global alias.rm-deleted '!git rm $(git ls-files --deleted)'

To make a (default type) tag:

    git tag <tagname>

To push the tag to the server:

    git push <remote> <tagname>

To push one branch to another (https://stackoverflow.com/a/13897766/795690):

	git push <dest-remote> <local-branch>:<dest-branch>
    git push origin develop:release/test

(https://stackoverflow.com/a/6591218/795690)  
If you want to rename a branch while pointed to any branch, do:

    git branch -m <oldname> <newname>

If you want to rename the current branch, you can do:

    git branch -m <newname>
