---
title: "Interactive git rebase : how to create nice PR"
seoTitle: "How to use interactive git rebase to improve our PR"
seoDescription: "This guide will show you how to use interactive git rebase in IntelliJ to improve your pull requests. Learn how to quickly and easily make changes to your c"
datePublished: Mon Mar 13 2023 08:38:05 GMT+0000 (Coordinated Universal Time)
cuid: clf6knwbi001009judgr21c04
slug: interactive-git-rebase-how-to-create-nice-pr
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/842ofHC6MaI/upload/b19b5cdc94e2450da9ebab9fdd583b2a.jpeg
tags: intellij, git, pr, git-rebase

---

Do you ever feel overwhelmed when looking at your list of commits before sending a pull request (PR)? Is it difficult for the reviewer to keep track of your different changes? Do you have chains of fixup commits in your PR? If so, Git has a great feature that can help you keep your commits in an orderly fashion. In this blog, I'll be demonstrating this feature and how it can help you maintain a clear and intelligible list of commits for easy review.

### How my work can looks like

When I start working on a task, I create a new branch and push commits on this branch. I may push on Friday a commit to save my work in case something happens during the weekend (to me or my laptop). I can also push a commit to trigger a pipeline to see how it will react to my changes. I often push a commit based on Sonarqube review (or any other tools like WhiteSource,...)

In the end, my tree history can look like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678694077077/4595f9d8-b74a-46b5-af48-ac4a6503c1f8.png align="center")

I feel sorry for the person doing the review... There is too much noise on the list of commits, hard to point out what was done, what might be reverted, etc, etc...

Even from my point of view. Let's say I have to stop working on it for few days (or weeks), when I go back on it, this won't help me!

### Rewrite the story

When I end up in this situation, I try to rewrite the story: rewrite the git history. To do so, I use git rebase in the interactive mode. Thanks to it, I will be able to do [several actions](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History) on this list of commits:

```console
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
```

To clean my PR, I only use:

* pick to keep the commits that have a meaning
    
* reword to fix typos, include more details, rephrase my commit message
    
* squash to merge several commits and rewrite the message in one action. This can happen when 2 commits are on the same feature and the message should be changed
    
* fixup to merge several commits and keep the message of the first commit. This happens for commits with code typos, commits with back-and-forth push / revert to try things on the pipeline, etc...
    

### How I do it technically

As for every git feature, you can use git-cli for that. You can find all the details [here](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History).

I want to show you how to do it in IntelliJ (I am sure your favorite idea is also capable to do it).

IntelliJ has a git plugin. When you open it, you will see the git history of your project (local and remote)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678694898846/0292c941-e0b2-4646-80ba-f26da7cac775.png align="center")

I have selected my branch GIT\_REBASE and I will see the all the commits (from Create a feature to Update test)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678694953245/9aed4926-0760-4b04-ae9a-63d94726f4ae.png align="center")

When I right click on the first commit (Create a feature), I see a menu with all the options:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678695079752/5df18d0f-7a59-4964-9c87-5a9d9b9c7e3c.png align="center")

I click on Interactive Rebase from Here and IntelliJ opens the wizard to apply my rebase options

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678695175733/61d87109-416e-47b6-a106-3641dd09adc1.png align="center")

On top, there is the list of options I can apply to the commits:

* Up / Down to change the commit orders
    
* Reword to change the commit message
    
* Squash / Fixup (only available if it is not the first commit of the tree)
    
* Drop
    

I will select the first Fixup commit:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678695319154/9336943a-8c1c-4c9f-8813-a9547924b6ca.png align="center")

Now my tree looks like this.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678695349424/c5fdcda0-f512-42e3-9751-8a4623f864ce.png align="center")

Let's keep going by doing all the fixup first. We can "stack" several fixup:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678695431163/7271f32d-5f4f-440c-8ecd-a26620afe659.png align="center")

So all the fix commits have been marked as fixup. I still have 3 commits:

* Create a feature (the first commit that I want to keep like that)
    
* Apply review: I want to keep it but I want to rewrite the message
    
* Update feature 2: I want to keep it like that also.
    

I select Apply review commit, click Reword and now I can see this input field

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678695517781/c73dd80a-2a9d-4d09-a7fd-1c0a07442211.png align="center")

Better no?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678695607975/664f524c-7032-4d2f-b430-e67ffd923337.png align="center")

When all my changes are done, I just need to click Start Rebasing:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678695648105/7c0414d2-1d19-465a-a9d7-f433403b4092.png align="center")

Now I have my nice tree and can create a PR with 3 commits with lots of value.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678696058584/1aca459b-3f8e-4f5f-9f27-f9b68f9873a3.png align="center")

**If at any point, you are blocked, you can always cancel the rebase:**

```plaintext
git rebase --abort
```

So now only thing that you have to do is to push your PR. If you already pushed it, you will have to do a push force:

```plaintext
git push -f
```

### Conclusion

Git is a powerful tool and can help us in our work. When we submit a PR, we have to consider that someone will look at it. They will look at the code, but the commit history can also be a great source of information.

So clean it before you submit it, you will look more professional, and it will make the process smother!