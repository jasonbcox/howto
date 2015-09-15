
# howto/git

Assuming you already know the basics of source control, this guide should be useful for learning how to use git.
This document covers all of the current processes, best practices, and tips/tricks that I use personally.
These methods may not apply for everyone.


### Table of Contents
 * [Step 0: Setup Your .gitconfig and .bash_profile](#step0)
 * [Step 1: Create or Clone a Repository](#step1)
 * [Guide to Pulling Latest Changes](#guide-pull)
 * [Guide to Pushing Changes](#guide-push)
 * [Guide to Git Diff](#guide-diff)
 * [Guide to Stashing Changes](#guide-stash)
 * [Guide to Branching (Locally)](#guide-local-branch)
 * [Guide to Branching (Remotely)](#guide-remote-branch)
 * [Guide to Reverting Changes](#guide-revert)
 * [Tips/Tricks](#tips-tricks)


## Step 0: Setup Your .gitconfig and .bash_profile <a id="step0"></a>
Your global .gitconfig houses all of your own settings for all git repositories on your machine.
This project contains a template you can build on that provides some nice features that don't ship with git alone (see *.gitconfig* above)

In your .bash_profile, you can append the current branch to your PS1 with the example found in this project (see *.bash_profile.partial* above)


## Step 1: Create or Clone a Repository <a id="step1"></a>

If you're starting from scratch, create a new directory and turn it into a git repository:
```
mkdir myproject
git init            # that's it!
```
If you're working on an existing project, just clone the repository (this will create a new directory for you, named after the project):
`git clone myuser@mydomain.com:path/to/repo/myproject.git`


## Guide to Pulling Latest Changes <a id="guide-pull"></a>

If you came from SVN, this is similar to 'svn up':
```
git stash           # only if you have unstaged changes
git pull --rebase   # git pl   # shortcut
git stash pop       # only if you did 'git stash'
```

**Important:** Except in extreme circumstances, do not leave off --rebase.
A normal 'git pull' will produce a merge instead of a rebase, and this may cause problems for maintaining good change history.
Read this for more info: (http://stackoverflow.com/questions/16666089/whats-the-difference-between-git-merge-and-git-rebase/16666418#16666418) and for a more comprehensive explanation of rebase: (https://www.atlassian.com/git/tutorials/rewriting-history/git-rebase-i)
Note that some teams may choose to use merging over rebasing.  This is fine as long as you are choosing to do so for the right reasons.  I personally advocate rebasing in most circumstances because linear change history is easier to read and rollback (if necessary).

## Guide to Pushing Changes <a id="guide-push"></a>

1. Make your changes, then add them to git's tracking to be committed.  There are a number of ways to add files, some of which are listed here:
  ```
  git add file/to/add    # add a single file
  git add -u             # add all modified and deleted files
  git add .              # add all new and modified files
  git add -A             # git add .; git add -u
  ```

  * Make sure to check what files you added with 'git status'.  Did you accidentally add a bunch of files that you didn't mean to? Try these:
  ```
  git reset          # This will undo all of your staged changes.  Don't worry, your changes won't be reverted
  git reset <file>   # Unstage a single file
  ```

1. Commit your changes to your **local** repository.  If you're unfamiliar with how git works, keep in mind that you have a full repository on your local box.  When you do 'git commit' you are not making a commit like you would with 'svn commit' (where your changes get pushed to the central repository).  Your changes are simply tracked on your local repository.
  ```
  git commit -m "Your commit message"
  ```

1. What's really nice about having a local repository is that you can commit as often as you like.  It's generally recommended to commit as often as you can so that you can revert to older local states, if need be.
  However, if you're working in a team, you don't want to clutter your team's git history with commit messages like the following:
  ```
  * 123456 - Fixed the bug for real this time (1 hour ago) <Jason B. Cox>
  * 789012 - Fixed a bug (6 hours ago) <Jason B. Cox>
  * 123456 - First half of the feature (2 days ago) <Jason B. Cox>
  ```

  So before you push your changes upstream, be sure to squash your commits:
  `git rebase -i HEAD~X`
  Where X is the number of commits to squash from HEAD.  Say you committed 3 times and want to squash those 3 commits into one: git rebase -i HEAD~3.

  *When you run this command, you must enter the commits to squash.

  <table>
    <tr>
      <td>For example this:</td>
      <td>Would be changed to this:</td>
    </tr>
    <tr>
      <td><pre>$ git rebase -i HEAD~4</pre></td>
      <td><pre>$ git rebase -i HEAD~4</pre></td>
    </tr>
    <tr>
      <td>
        <pre>
  pick 01d1124 Adding license
  pick 6340aaa Moving license into its own file
  pick ebfd367 Jekyll has become self-aware.
  pick 30e0ccb Changed the tagline in the binary, too.
        </pre>
      </td>
      <td>
        <pre>
  pick 01d1124 Adding license
  squash 6340aaa Moving license into its own file
  squash ebfd367 Jekyll has become self-aware.
  squash 30e0ccb Changed the tagline in the binary, too.
        </pre>
      </td>
    </tr>
  </table>

  **Look at this quick tutorial on git sqashing:** <a href="http://gitready.com/advanced/2009/02/10/squashing-commits-with-rebase.html">http://gitready.com/advanced/2009/02/10/squashing-commits-with-rebase.html</a> **(No, seriously, read it.  99% of your squashing problems can be fixed by reading this!)**

1. Push your changes:
  `git push`


## Guide to Git Diff <a id="guide-diff"></a>

Diff commands will differ depending on the situation. These are some of the most common situations you will encounter:
Shows the diff between **unstaged changes** and HEAD: `git diff`
Shows the diff between **staged changes** and HEAD (changes that you have added with 'git add'): `git diff --cached`
Shows the diff between the given commit and the one before it: `git diff 123abc^ 123abc`
Shows the diff from the older commit (left) and newer commit (right):
```
git diff 123abc..HEAD
git diff 123abc..456def
```
Shows all files changed in a commit: `git diff 123abc^ --name-only`

## Guide to Stashing Changes <a id="guide-stash"></a>

If you want to stash your changes without committing them to a new branch, you can use git stash.  Your stash exists outside of branches, so they will always be accessible no matter which branch you are on.

### Using the Stash Stack
Stash your changes on the top of the stash stack: `git stash`

Retrieve your changes from the top of the stash stack:
`git stash apply`

Or Using Stash Numbers (if you have multiple stashes):
Stash your changes
`git stash save "stash description"`

Retrieve your changes later
```
git stash list               # find the stash number for your changes
git stash apply stash@{X}    # where X is the stash number
```
Delete your old stash later `git stash drop stash@{X}`
Or delete all of your stashes, if you like: `git stash clear`


## Guide to Branching (Locally) <a id="guide-local-branch"></a>

If you're working on multiple bugs/features at the same time, consider branching.  Branching in git is easy and light weight.
Any branch you create will remain on your local box, so don't worry about creating clutter for your team.
If you want to save a branch externally or share your local branch, you can push it upstream (just like you do with master).
For more info on remote branching, see the next section: Guide to Branching (Remotely).
Create a new local branch:
```
git checkout -b branchName
```

**Note: If you want to branch from a particular commit, find its hash in the git logs and branch as such where ###### is the commit hash:**
`git checkout -b branchName ######`
Note: git checkout -b is simply shorthand for:
```
git branch branchName     # creates the new branch
git checkout branchName   # switches to the new branch
```

Make your changes and commit, then rebase your changes into the master branch
```
git rebase -i HEAD~X     # squash your commits into a single commit (note: you may want to 'git review dcommit' after this step)
git checkout master      # switch to master branch
git pull --rebase        # pull the latest changes from upstream

git checkout branchName  # switch back to your branch
git rebase master        # take the upstream changes on master, and apply them to your local branch

git checkout master      # switch back to master so you can push your changes
git rebase branchName    # rebase your local branch into master
git push                 # push your changes to master
git branch -d branchName # delete this topic branch after merging into master (to be safe, do this step only after you have pushed your changes)
```


## Guide to Branching (Remotely) <a id="guide-remote-branch"></a>

If you're working on a feature with your coworkers, but don't want to interfere with work on the master branch, create another upstream branch for your group to work on!
Create a new local branch (see above for details):
```
git checkout -b branchName
```

Push the branch upstream so your coworkers can begin working on it:
`git push -u origin branchName`

Your other coworkers can now pull the new branch:
```
git fetch
git checkout branchName
```

Instead of pushing to master, push your changes to this new remote (git pull --rebase and git push will work on this branch just like if you were working on master)
When you are done working on the feature, rebase all of the changes onto master (same as above in Guide to Branching (Locally))
Lastly, once all of the changes have been confirmed and properly rebased onto the remote master branch, delete the remote feature branch to keep your repository clean:
```
git push origin --delete branchName    # delete the upstream branch
git branch -d branchName               # delete the local branch copy
```

## Guide to Reverting Changes <a id="guide-revert"></a>

There are different ways to revert different types of changes.  Listed below are the safest methods for reverting.
**WARNING: DO NOT USE 'git reset' ON COMMITS THAT HAVE BEEN PUSHED UPSTREAM.**
You could end up breaking your master branch and your neighboring engineers will hate you.
YOU HAVE BEEN WARNED.

1. Undo local commits, but keep the changes as unstaged changes:
  ```
  # Undo the latest commit
  git reset --soft HEAD~1

  # Undo all of your local commits
  git reset --soft origin/HEAD..HEAD
  ```
  Once you have done a reset soft on local commits, you can remove files from staging (without reverting the changes) by doing the following:
  ```
  git reset <path/to/the/file>
  ```
  Doing a plain git reset is useful when you want to remove an accidentally added a file and committed it along with other changes.


1. Revert a single file (your changes will be lost):
  ```
  # Revert the file to latest:
  git checkout origin/HEAD <filename>

  # Revert the file to a specific commit:
  git checkout <commit hash> <filename>
  ```

1. Revert uncommitted changes (locally committed changes will be intact):
  `git reset --hard HEAD`

1. Revert locally committed changes (your committed changes will be lost):
  `git reset --hard origin/HEAD`

1. Revert changes that have already been pushed upstream (note: this creates a set of reverse commits that undo the changes applied by previous commits.  Use with caution):
  ```
  git revert <oldest commit to revert>..<latest commit to revert>
  # Don't forget the two dots         ^
  ```

## Tips/Tricks <a id="tips-tricks"></a>

Applying a single commit from one branch onto another
```
git checkout theBranchToApplyTo
git cherry-pick ######
```
Where ###### is the partial hash of the commit you want to apply to your current branch.
Moving commits from master to another branch<br />You may want to do this when another issue comes and you no longer want to be working directly on master.
```
git checkout master
git branch newBranchName
git reset --hard HEAD~X    # WARNING: This will obliterate any unstaged changes.  Make sure to commit or stash changes that you want to save!
                           # HEAD~X will reset back X number of commits (ex: HEAD~3 will roll back to the commit before the most recent 3 commits)
git checkout newBranchName
```

Differences between deleting branches
  * Delete a local branch cautiously (it will notify you if your changes haven't been pushed upstream):
  ```
  git branch -d <branchname>
  ```

  * Forcefully delete a local branch.  If you have squashed your changes and then pushed them upstream, -d may through an error (CAREFUL! Make sure you actually want to delete the branch! There is no prompt!):
  ```
  git branch -D <branchname>
  ```

  * Delete the branch remotely (requires permissions):
  ```
  git push origin --delete <branchname>
  ```

Applying a patch.
  * If you're using IntelliJ, the easiest way to apply this is to use VCS -> Apply Patch.  For command line users:
  ```
  patch -p1 < filename.patch
  ```

  * If the above doesn't work, try this (the p level depends on the patch format):
  ```
  patch -p0 < filename.patch
  ```

Immediately apply diff as a patch:
```
git diff <commit-from> <commit-to> | git apply
```

