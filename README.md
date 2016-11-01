Don't be a git
==============

This is a git repository that is to be used to along with my presentation "Don't be a git" to learn a few things about git.

Topics
------

The following will be covered.

* `git add/reset -p`
* `git rebase -i`
* `git bisect`
* `git rerere`

-p flag (or the power flag)
---------------------------

The -p flag can be used on a lot of git commands but I recommend using it on `git add` and `git reset`.
From the [git docs](https://git-scm.com/docs/git-add) `-p` or `--patch`does the following:

```
Interactively choose hunks of patch between the index and the work tree and add them to the index.
This gives the user a chance to review the difference before adding modified contents to the index.
```

For our example I would recommend checking out the master branch. Making a could of changes to the file `meet-magento-romania.txt` on lines 30 and 83 so that git will see them as two different patches.
Then after that run `git add -p`.

When running this command you will have the following options for each patch

```
y - stage this hunk
n - do not stage this hunk
q - quit; do not stage this hunk nor any of the remaining ones
a - stage this hunk and all later hunks in the file
d - do not stage this hunk nor any of the later hunks in the file
g - select a hunk to go to
/ - search for a hunk matching the given regex
j - leave this hunk undecided, see next undecided hunk
J - leave this hunk undecided, see next hunk
k - leave this hunk undecided, see previous undecided hunk
K - leave this hunk undecided, see previous hunk
s - split the current hunk into smaller hunks
e - manually edit the current hunk
? - print help
```

You can also try this with `git reset -p` which should do the reverse of the add and unstage the changes patch at a time.

Keeping your history
--------------------

For working with the rebase please checkout the branch `diverged-branch`.
This branch should have a different history than the current master.
To see the difference in history run `git log --oneline --decorate --all --graph`

So to update the diverged branch to be onto of the current master we should call `git rebase -i origin/master`.
This will prompt you with the commits that will be moved onto the master head.
For our example we should simply save and exit this prompt.
There will be a conflict here and if you run `git status` you should see:

```
rebase in progress; onto 4d701d4
You are currently rebasing branch 'diverged-branch' on '4d701d4'.
  (fix conflicts and then run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Unmerged paths:
  (use "git reset HEAD <file>..." to unstage)
  (use "git add <file>..." to mark resolution)

        both modified:      meet-magento-romania.txt
```

Use your faviourate text editor to fix the conflicts in `meet-magento-romania.txt` run `git add meet-magento-romania.txt` and then `git rebase --continue`.
It will ask you for a commit message and then will finish the rebase.

After the rebase is finished running `git log --oneline --decorate --all --graph` will show you that the branch `diverged-branch` is now 1 commit ahead of `master`.

Find the bug with bisect
------------------------

So if we checkout the branch `bisect-branch` and open the file `meet-magento-romania.txt` we will see the following line:

```
* Anna Völkl (Looking for a security talk come see mine! - Love Anna),
```

To find the commit that added this change we can run `git bisect`.
For this we will use the initial bad commit of `c199955` and good commit of `abf1494`.

```
git bisect start
git bisect bad c199955
git bisect good abf1494
```

This will start the bisect process and move the head pointer to the first testing position.
To check if this is good or bad simple open the `meet-magento-romania.txt` file and look for the change.
In this case the change is still there so we call `git bisect bad`
It will then move the head pointer again and we can check the file for the change.
Again it is there so we call `git bisect bad`.
Once more it moves the head pointer and we can check the file.
This time the change is not there so we call `git bisect good`

We will then get a message about the found "bad" commit.

```
b77c7a8b466c1711cbd66e9d0b3093adc325d526 is the first bad commit
commit b77c7a8b466c1711cbd66e9d0b3093adc325d526
Author: Anna Völkl <miss-magento-security@annavoelkl.at>
Date:   Tue Oct 25 08:34:59 2016 +0000

    This is not a security issue nothing to see here

:100644 100644 5d320792604905fb0eccbfb8e81ecdc8534920a2 df928694bb4a8b2ea1ba7043884c93953105ed83 M      meet-magento-romania.txt
```

To finish the bisect run `git bisect reset`.

Reuse recorded resolution
-------------------------

To demo the recorded resolutions we will firstly checkout the correct branch `rerere-branch`.
And then enable rerere with the command:

* `git config --global rerere.enabled true`

Now we will merge in the master branch with the command `git merge master`.
You should see the following message if rerere was enabled correctly.

```
Auto-merging meet-magento-romania.txt
CONFLICT (content): Merge conflict in meet-magento-romania.txt
Recorded preimage for 'meet-magento-romania.txt'
Automatic merge failed; fix conflicts and then commit the resul
```

So we simply fix the conflict and commit

* `vim meet-magento-romania.txt`
* `git add meet-magento-romania.txt`
* `git commit`

This will have saved the resolved resolution for use again later.

Now we can run this on a second branch `rerere-branch2`.
Again call `git merge master` but this time we should see the following:

```
Auto-merging meet-magento-romania.txt
CONFLICT (content): Merge conflict in meet-magento-romania.txt
Resolved 'meet-magento-romania.txt' using previous resolution.
Automatic merge failed; fix conflicts and then commit the result.
```

And on opening the file `meet-magento-romania.txt` you should see your resolution has been used.
Again adding and commit the file should finish the merge.
