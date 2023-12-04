+++
title = 'Basic Git Jargon and Commands'
date = 2023-11-23T17:36:18+01:00
draft = false
slug = 'git-commands' 
tags = [ 'git', 'summary']
+++

{{< lead >}}
When I asked Juju "What are your favourite parts of programming?" **Git** was one of the answers I received. It is so **powerful** yet **concise** that leads me to write this blog. From here you can find the very plain explanations of Git jargon and the most commonly used commands. 

Most of the concepts come from [Learn Git Branching](https://learngitbranching.js.org/), which is also recommended by Juju. It is a very fun website that visualises Git operations and helps beginners understand Git in a highly interactive and effective way. Most importantly, it does not require an internet connection. It became my good pal during travels in Switzerland when I had no internet on the train! :P

This blog will be regularly updated with fresh insights and newfound knowledge about Git. 
{{< /lead >}}

## 1. What is Git

Before delving into the details, let's first have a general understanding of Git.

### 1.1. The Definition and Usage of Git
Git is a free and open-source distributed **version control system** designed to handle everything from a small file to very large projects with speed and efficiency [^1]. To understand it better, for now we can simply think about the "Version history" function in the Google Document but with more add-ons. 

Git sounds very simple according to its definition, but it is also very versatile. The most common use case is to be used for collaborative software development. Its poweful version control functions allow multiple contributors to work on the same project simultaneously, without interfering with the main project until the changes are ready to be merged.
 
For personal usage, it can be used to make the work more flexible by tracing back to different versions or developing different features on a project at the same time. Generally, it is always useful if you want to contribute to or try some repositories on GitHub.

### 1.2. Basic Elements of Git
[Table 1.2.1.](#table1) provides definitions for some pieces of common Git jargon. Developing a general understanding of these terms before diving into the commands will make the subsequent sections more friendly.

<a name="table1"></a>
<p align="center">  
<em>Table 1.2.1. Brief Introductions of Git Jargon</em>
</p>

|    **Jagon**    | **Brief Explanation**                                                                                                                                                                                                                                             |
|:---------------:|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|  **Repository** | (aka. repo) a storage space with all files and version history related to the project                                                                                                                                                                                         |
|    **Remote**   | a storage space on a remote server (like GitHub) that is connected to another repository                                                                                              |
|  **Workspace**  | (aka. Working Directory) it holds the actual files on the disk of our laptop                                                                                                                                                                                      |
|    **Commit**   | a snapshot of the project at the time you submit your changes                                                                                                                                                                                                     |
|     **Add**     | stages changes that are ready for the commit (tell Git what changes you want to include)                                                                                                                                                                          |
| **Index stage** | an intermediate area between the workspace and the repository for holding the "added" changes                                                                                                                                                                     |
|    **Clone**    | creates a copy of a repository to local (like our laptops)                                                                                                                                                                                                        |
|    **Branch**   | a parallel line of development within a repository                                                                                                                                                                                                                |
|    **Merge**    | combine changes from different branches by createing a new commit that has both branch histories                                                                                                                                                                  |
|    **Rebase**   | combine a sequence of commits (a branch) to a new base commit (branch) by rewriting the commit history                                                                                                                                                            |
|   **Checkout**  | switch between branches or commits                                                                                                                                                                                                                                |
|     **HEAD**    | a pointer to the latest commit in the currently checked-out branch, and can be redirected manually                                                                                                                                                                |
|    **Fetch**    | retrieves changes from a remote repository to local but doesn't automatically merge them into your local branch                                                                                                                                                   |
|     **Pull**    | fetch remote changes and then merge them (pull request (PR) is a very common word on GitHub. It is used to propose the changes that have been made to a repository and request the repository owner to review the changes and integrate them into the repository) |
|     **Push**    | uploads the local changes to a remote repository                                                                                                                                                                                                                  |

[Figure 1.2.1.](git-relat.png) explains the basic relationships between each basic Git element. The interplay between them will be more clear as we explore deeper into Git.
<p align="center">  
<em>Figure 1.2.1. Relationships between Git Elements</em>
</p>

![git-relat](git-relat.png)

## 2. Basic Local Git Commands

Let's first look at what local Git can do. Without using the remote repository, Git can still provide a range of version control and collaboration features at the local level.

### 2.1. Creating New Things: Commit and Branch 
<u>***Concepts:***</u><br>
**Commit**: Snapshots of the project.<br>
**Stage**: a holding area, use **stage** to select the changes you want to commit (after making many changes to your current file, you might just want to commit some of them).<br>
**Branch**: Pointers to a specific commit. "I want to include the work of this commit and all parent commits".<br>


<u>***Commands:***</u><br>
`git commit`: make a new commit <br>
`git branch bugFix`: create a new branch named `bugFix` <br>
`git checkout bugFix`: put us on the branch `bugFix`<br>
`git checkout -b bugFix`: create a new branch `bugFix` and check out `bugFix`<br>

### 2.2. Moving Around: HEAD, ^ and ~

<u>***Concepts:***</u><br>
**HEAD**: Currently checked out commit (always points to the most recent commit on this branch by default).<br>
**Detach HEAD**: Attach it to a commit instead of a branch.<br>
**Relative refs**: Start with somewhere memorable and work from there.<br>

<u>***Commands:***</u><br>
`git checkout C1`: Detach HEAD, from `HEAD -> main -> C1` to `HEAD -> C1`<br>
{{< alert "bell" >}}
In "detached HEAD" state, you are not on a branch but at a specific commit.
{{< /alert >}}<br>
`~<num>`: move the pointer from current location upwards a \<number> of times.<br>
`git checkout main~2` : move upwards 2 commits (the grandparent of `main`).<br>
`^`: move upwards 1 commit (find the parent of the specified commit).<br>
`git checkout main^^` : move upwards 2 commits (the grandparent of `main`).<br>

{{< alert "bell" >}}
The `^` modifier also accepts an optional number after it, which specifies which parent reference to follow from a merge commit.
{{< /alert >}}<br>
`git checkout main^2`: checkout the second parent from a merged commit point<br>
we can even chain them together `git checkout HEAD~^2~2`, but this will only move around `HEAD`, if we want to move branch around: `git branch bugFix main~^2~1`<br>
`git log`: a history record, showing all the changes or commits made in your project (starting from the most recent to the earliest).<br>

### 2.3. Redesigning Branches: Merge, Squash, Rebase, ans Cherry-pick
<u>***Concepts:***</u><br>
**Merge**: Create a special commit that has two unique parents. "I want to include all the work from this parent over here and this one over here, *and* the set of all their parents".<br>
**Squash**: Combine multiple commits into a single commit.<br>
**Rebase**: Combine work between branches by making a nice linear sequence of commits. Copy a set of commits and plop them down somewhere else.<br>
**Interactive rebasing**: Open up a UI to show you which commits (with their commit hashes and messages) are about to be copied to rebase. Before rebasing, you can:
- reorder commits by changing their order in the UI
- toggle of the `pick` button of the specific commits to drop them<br>
  
**Cherry-pick**: Select a series of commits according to your demands below the `HEAD`.<br>

<u>***Commands:***</u><br>
`git merge bugFix`: merge `bugFix` to `main`(checked out)<br>
`git rebase main`: move the work from `bugFix`(checked out) directly onto the `main`<br>
`git rebase main bugFix`: rebase `bugFix` onto `main`<br>
{{< alert "bell" >}}
Please pay attention to the difference between *merge* and *rebase*. Assume we are currently checked out on `main`:<br>
`git merge A`: merge `A` to `main`<br>
`git rebase A`: rebase `main` onto `A`
{{< /alert >}}

`git branch -f main HEAD~3`: move (by force) `main` to 3 parents behind current `HEAD`<br>
`git branch -f main C2`: move (by force) `main` to `C2` (a commit)<br>
`git rebase -i HEAD~4`: rebase the commits until 3 beforehand the `HEAD` (4 in total including `HEAD`) on to `main`<br>
`git rebase -i bugFix`: rebase the selected/reordered commits to `bugFix`<br>
`git cherry-pick C1 C2`: copy C1, C2 below current location (`HEAD`)<br>

### 2.4. Undoing the Changes: Reset and Revert
Undo the previous steps on local or remote branch. 

`git reset HEAD~1`: reverses changes by moving a branch reference backwards in time to one older commit before HEAD.<br>
`git revert HEAD`: reverse changes by creating a new commit that introduces changes that exactly reverse the commit.<br>
{{< alert "bell" >}}
The ways of undoing changes using *reset* and *revert* are different.<br>
By moving the branch pointer to the previous commit, *reset* rewrites the history.<br>
While *revert* creates a new commit that undoes the changes introduced by a previous commit (safer for shared branches in team work).
{{< /alert >}}<br>

### 2.5. Git tags
**Permanently** mark certain **commits** as "milestones" (major releases and big merges) that you can then reference like a branch. You can't commit directly onto the `v1` tag.

`git tag v1 C1`: name the commit `C1` with the tag `v1` (if you leave the C1 off, git will just use whatever `HEAD` is at).<br>

{{< alert "bell" >}}
`git checkout v1` is equal to `git checkout C1`<br>
{{< /alert >}}

`git describe`: describe where you are relative to the closest "anchor" (aka tag). Git describe takes the form of: `git describe <ref>`, where `<ref>` is anything Git can resolve into a commit. If you don't specify a ref, git just uses where you're checked out right now (`HEAD`).

The output of the command looks like: `<tag>_<numCommits>_g<hash>`, where `tag` is the closest ancestor tag in history, `numCommits` is how many commits away that tag is, `g` stands for "git", and `<hash>` is the hash of the commit being described.<br>

For example, if your current commit is 3 commits away from the nearest tag named v1, and the hash of your current commit is abcdefg, the output might look like `v1_3_gabcdefg`.

<!--
# 4. Some suggestions

I find that visualizing Git activities, either mentally or with tools, and even drawing them out on a page, can be the most effective approach.
- Branch more and branch early;
- Squash and rebase will make the Git graph less hairy


This concept was inspired by a [post](https://datasci.social/@b0rk@jvns.ca/111375097434853441) by Julia Evans.

<div style="display:flex; gap:6px">

{{< badge >}} git {{< /badge >}}

{{< badge >}} summary {{< /badge >}}
</div>
-->
[^1]: https://git-scm.com/
