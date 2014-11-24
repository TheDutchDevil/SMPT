#Git + Gerrit workflow

Every time the local repo layout changes an ASCII illustration is included to clarify how the last action affected the local repo.

`(x)` represents a commit where x is the commit hash/name/number  
`[master]` represents the branch

###1. Clone the repo

**Using ssh**

Go in Gerrit to Projects > List and select the project you want to clone.
There you select 'Clone with commit-msg hook' and 'ssh'.
You can clone with the generated string.

**Local repo status**  

    (1)---(2) [HEAD][master]

###2. Make a new branch

Lets say we are going to work on the `Users` class. First, create a new branch

`git branch users`

**Local repo status**  

    (1)---(2) [HEAD][master][users]

Switch to the new branch so we can do some work

`git checkout users`

This can also be done in 1 line

`git checkout -b users`

###3. Do some work in the branch

As example we will just create a new empty file

`touch testfile.md`

Add it to the git index

`git add testfile`

Now commit it with a message describing what we are committing

`git commit -m "Add a file for testing purposes."`

**Local repo status**  

    (1)---(2) [master]
            \
             (3) [HEAD][users]

Now we can push the commit to gerrit

`git push origin HEAD:refs/for/master`


The `HEAD:refs/for/master` tells gerrit that this commit is to be merged into the remote master branch.

###4. Wait for the review

While you wait for your work to be reviewed you can continue to work (in a new branch).  
Make - and switch to - a new branch that is branched off of the master branch.  
Lets make a `cars` branch

`git checkout -b cars master`

**Local repo status**  

    (1)---(2) [HEAD][master][cars]
            \
             (3) [users]

Do some work

`touch another_test_file.md`

`git add another_test_file.md`

`git commit -m "Add a new test file."`

**Local repo status**  

             (4) [HEAD][cars]
            /
    (1)---(2) [master]
            \
             (3) [users]
 
 Do some more work
 
 `touch yet_another_file.md`
 
 and add some content to it in an editor.

###5. The review on commit 3 is done and it turns out we have to make some changes to it- but wait! We are still working on a 5th commit in the cars branch! When we try to you checkout the users branch we get an error saying "Please, commit your changes or stash them before you can switch branches.  Aborting" Omg now what?!

No worries - git can temporarily store your uncommitted work

`git stash`

Now the working index is clean and we can safely change to other branches.  
Change to the users branch to make that edit.

`git checkout users`

**Local repo status**  

             (4) [cars]
            /
    (1)---(2) [master]
            \
             (3) [HEAD][users]

Edit the files that need editing in an IDE or something and save the changes.  
Now we need to add & commit these files again

`git commit -a --amend`

The `-a` tells git to commit all changed files and the `--amend` tells git not to make a new commit, but to dump the new changes onto the last existing commit. This is needed so gerrit can tell which files and commits belong where. A new commit would create a new Change-Id and we dont want that, so we amend the old commit.

**Local repo status (notice there are no new commits! We merely edited the existing one)**

             (4) [cars]
            /
    (1)---(2) [master]
            \
             (3) [HEAD][users]

Dont forget to push the amended commit

`git push origin HEAD:refs/for/master`

gerrit will put this commit (the one we just amended) on the same page as the original commit. We can compare these so called patch sets to view what changes were made in the amended commit, based on the original commit.

###6. We went home to sleep and when we got back at work we got notified that our amended commit was approved and merged into the remote master branch. Yay - now what?!

Because the remote master branch was updated we should sync our local master with the remote

`git checkout master`

`git pull`

This will put the commit we made in the users branch (that was merged into the remote master branch) in our local master branch.

**Local repo status**  

             (4) [cars]
            /
    (1)---(2)---(3) [HEAD][master]
            \
             (3) [users]

Because the work we did in the users branch is now merged into the master branch (local as well as remote) we can safely remove the users branch

`git branch -d users`

**Local repo status**  

             (4) [cars]
            /
    (1)---(2)---(3) [HEAD][master]

###7. We can now continue to work in our cars branch.

**- If the cars and users branches' files are all different (not worked on the same file in these 2 branches)**  
There is no risk of conflicts so we can just change to branch cars and continue our work there

`git checkout cars`

To get back the work we stashed earlier we use

`git stash pop`

The index will now be as it was right before step 5. We can now continue to work normally.

**Local repo status**  

             (4) [HEAD][cars]
            /
    (1)---(2)---(3) [master]


**- If we are working on files in the cars branche that were also changed in the users branch**  
Because one file was changed on different branches, we will get conflicts if we try to push the work from branch cars. To avoid conflicts we do a so called **rebase**

`git checkout cars`

`git rebase master`

If there were indeed no conflicts the local repo will now be as illustrated below.  

**Local repo status**  

                  (4) [HEAD][cars]
                  /
    (1)---(2)---(3) [master]

To get back the work we stashed earlier we use

`git stash pop`

This will put the changes we made in this branch before step 5 back on top of it. Because some of these files were also changed in the users branch, it is likely that one or more conflicts will arise. We will have to manually edit the files to tell git which changes we need and where. Google 'git resolve conflicts' to find out how that works.