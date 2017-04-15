---
title: Git - Moving changes to different branches
date: 2015-09-05 5:45 PM
category: coding
from: stephanieliu.net
tags:
- git
- coding
---
I recently created a pull request with 92 changed files... 92!? Who wants to review 92 files? (To be fair, almost half of those changed files were deleted files, but still). So, I was asked to break up the changes into different branches to make it easier to review. Unfortunately, I'd made all the changes in one single commit (probably not my greatest idea), so I couldn't just cherry-pick commits into different branches.

Which brought me to my Git journey of resetting, stashing, tracking, and committing. And here's what I learned.

My problem was that I wasn't just changing files. I was removing files and adding files. So my first thought was to check out a branch from my changed branch, reset the head to point to the commit before all the changes, and then selectively add and commit specific changes... which, actually, would've worked just fine--what tripped me up, was the language `git add`.

So after checking out a branch from my modified branch and resetting the head to the previous commit, running `git status` informed me that all the files I had deleted were now not staged for commit, and all the files I had created were untracked. What's the difference? Honestly, I can't say that I know exactly--but I do know this: untracked files are files git is kind of ignoring for now. So, for example, if you ran `git stash` in this kind of state, all untracked changes would remain unchanged in your directory, but all "unstaged" changes would be stashed. And then of course, you could run `git stash pop` to bring those changes back.

So what happens if you have a bunch of deleted files that are unstaged? It turns out, if you run `git stash`, those files will reappear! And if you run `git checkout -- some-file-that-was-deleted-but-not-staged`, the file will reappear! Which, for me, was kind of hard to understand. (P.S. that `--` means "filename" so git knows you're not talking about something else, like a branch name). It was a weird concept for me because when you checkout a file that no longer exists, terminal won't autocomplete the filename and it seems like that file shouldn't exist, and yet it comes back to life! Neat, eh?

Anyway, what I actually ended up doing was resetting my head to one commit prior, stashing my changes (which effectively kept all my new files because they were untracked and brought the deleted files back to life), and then selectively running `git add` and `git rm` to add and remove the files I wanted. (I also found out that running `git rm` is kind of like running `rm` and then `git add the-file-you-removed`).

If I had made modifications to files, this wouldn't have worked, since `git stash` would've reverted all my changes! What I should've done was just checked out a new branch, reset the head to the previous commit, and then ran `git add` on each file I wanted to add as well as each file I wanted to remove. (That's the weird part--running `git add the-file-to-remove` when that file doesn't even exist in your directory as is).

Oh well, lesson learned. I'd love to understand git front to back... such a mysterious beast, git.

$$ \sum_{i=1}^n 1 = \infty $$
