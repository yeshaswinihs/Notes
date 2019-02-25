https://stackoverflow.com/questions/16638092/copying-a-rsa-public-key-to-clipboard
cat ~/.ssh/id_rsa.pub
then you can copy your ssh key
ssh-keygen -t rsa -C "your_email@example.com"

https://stackoverflow.com/questions/2452226/master-branch-and-origin-master-have-diverged-how-to-undiverge-branches

You can review the differences with a:
git log HEAD..origin/master
before pulling it (fetch + merge) (see also "How do you get git to always pull from a specific branch?")

When you have a message like:
"Your branch and 'origin/master' have diverged, # and have 1 and 1 different commit(s) each, respectively."
, check if you need to update origin. If origin is up-to-date, then some commits have been pushed to origin from another repo while you made your own commits locally.
... o ---- o ---- A ---- B  origin/master (upstream work)
                   \
                    C  master (your work)
You based commit C on commit A because that was the latest work you had fetched from upstream at the time.
However, before you tried to push back to origin, someone else pushed commit B.
Development history has diverged into separate paths. 
You can then merge or rebase. See Pro Git: Git Branching - Rebasing for details.
Merge
Use the git merge command:
$ git merge origin/master
This tells Git to integrate the changes from origin/master into your work and create a merge commit.
The graph of history now looks like this: 
... o ---- o ---- A ---- B  origin/master (upstream work)
                   \      \
                    C ---- M  master (your work)
The new merge, commit M, has two parents, each representing one path of development that led to the content stored in that commit.
Note that the history behind M is now non-linear.
Rebase
Use the git rebase command:
$ git rebase origin/master
This tells Git to replay commit C (your work) as if you had based it on commit B instead of A.
CVS and Subversion users routinely rebase their local changes on top of upstream work when they update before commit.
Git just adds explicit separation between the commit and rebase steps.
The graph of history now looks like this:
... o ---- o ---- A ---- B  origin/master (upstream work)
                          \
                           C'  master (your work)
Commit C' is a new commit created by the git rebase command.
It is different from C in two ways:
It has a different history: B instead of A.
Its content accounts for changes in both B and C; it is the same as M from the merge example. 
Note that the history behind C' is still linear.
We have chosen (for now) to allow only linear history in cmake.org/cmake.git.
This approach preserves the CVS-based workflow used previously and may ease the transition.
An attempt to push C' into our repository will work (assuming you have permissions and no one has pushed while you were rebasing).
The git pull command provides a shorthand way to fetch from origin and rebase local work on it:
$ git pull --rebase
This combines the above fetch and rebase steps into one command. 

560

I had this and am mystified as to what has caused it, even after reading the above responses. My solution was to do
git reset --hard origin/master
Then that just resets my (local) copy of master (which I assume is screwed up) to the correct point, as represented by (remote) origin/master.
WARNING: You will lose all changes not yet pushed to origin/master.
git pull --rebase origin/master 
is a single command that can help you most of the time.
Edit: Pulls the commits from the origin/master and applies your changes upon the newly pulled branch history.

27

I found myself in this situation when I tried to rebase a branch that was tracking a remote branch, and I was trying to rebase it on master. In this scenario if you try to rebase, you'll most likely find your branch diverged and it can create a mess that isn't for git nubees!
Let's say you are on branch my_remote_tracking_branch, which was branched from master
$ git status
# On branch my_remote_tracking_branch
nothing to commit (working directory clean)
And now you are trying to rebase from master as:
git rebase master
STOP NOW and save yourself some trouble! Instead, use merge as:
git merge master
Yes, you'll end up with extra commits on your branch. But unless you are up for "un-diverging" branches, this will be a much smoother workflow than rebasing. See this blog for a much more detailed explanation.
On the other hand, if your branch is only a local branch (i.e. not yet pushed to any remote) you should 
definitely do a rebase (and your branch will not diverge in this case).
Now if you are reading this because you already are in a "diverged" scenario due to such rebase, you can get back to the last commit from origin (i.e. in an un-diverged state) by using:
git reset --hard origin/my_remote_tracking_branch

20

In my case here is what I did to cause the diverged message: I did git push but then did git commit --amend to add something to the commit message. Then I also did another commit.
So in my case that simply meant origin/master was out of date. Because I knew no-one else was touching origin/master, the fix was trivial: git push -f (where -f means force)


6

In my case I have pushed changes to origin/master and then realised I should not have done so :-( This was complicated by the fact that the local changes were in a subtree. So I went back to the last good commit before the "bad" local changes (using SourceTree) and then I got the "divergence message".
After fixing my mess locally (the details are not important here) I wanted to "move back in time" the remote origin/master branch so that it would be in sync with the local master again. The solution in my case was:
git push origin master -f

To view the differences:
git difftool --dir-diff master origin/master
This will display the changes or differences between the two branches. In araxis (My favorite) it displays it in a folder diff style. Showing each of the changed files. I can then click on a file to see the details of the changes in the file.


4

I know there are plenty of answers here, but I think git reset --soft HEAD~1 deserves some attention, because it let you keep changes in the last local (not pushed) commit while solving the diverged state. I think this is a more versatile solution than pull with rebase, because the local commit can be reviewed and even moved to another branch.
The key is using --soft, instead of the harsh --hard. If there is more than 1 commit, a variation of HEAD~x should work. So here are all the steps that solved my situation (I had 1 local commit and 8 commits in the remote):
1) git reset --soft HEAD~1 to undo local commit. For the next steps, I've used the interface in SourceTree, but I think the following commands should also work:
2) git stash to stash changes from 1). Now all the changes are safe and there's no divergence anymore.
3) git pull to get the remote changes.
4) git stash pop or git stash apply to apply the last stashed changes, followed by a new commit, if wanted. This step is optional, along with 2), when want to trash the changes in local commit. Also, when want to commit to another branch, this step should be done after switching to the desired one. 

In my case this was caused by not committing my conflict resolution.
The problem was caused by running the git pull command. Changes in the origin led to conflicts with my local repo, which I resolved. However, I did not commit them. The solution at this point is to commit the changes (git commit the resolved file)
If you have also modified some files since resolving the conflict, the git status command will show the local modifications as unstaged local modifications and merge resolution as staged local modifications. This can be properly resolved by committing changes from the merge first by git commit, then adding and committing the unstaged changes as usual (e.g. by git commit -a). 


0

I had same message when I was trying to edit last commit message, of already pushed commit, using: git commit --amend -m "New message" When I pushed the changes using git push --force-with-lease repo_name branch_name there were no issues.



https://lifehacker.com/how-the-heck-do-i-use-github-5983680
git config --global user.name "Your Name Here"
git config --global user.email "your_email@youremail.com"
git init
git remote add origin https://github.com/gittest1040/Hello-World.git

