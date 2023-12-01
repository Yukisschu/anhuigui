+++
title = 'Basic Jargons in Git'
date = 2023-11-23T17:36:18+01:00
draft = false
slug = 'git-commands' 
tags = [ 'git', 'summary']
+++

{{< lead >}}
When I asked Juju, "What are your favourite programming languages?" Git was one of the answers I received. Afterward, I embarked on the study of git, and I was deeply impressed by its power and conciseness.

This led me to write a blog documenting some common Git usages I studied, and it will be updated whenever new thoughts and knowledge are added about Git. Most of the concepts come from [Learn](https://learngitbranching.js.org/) Git Branching](https://learngitbranching.js.org/) (recommended by Juju, too), which visualizes Git operations. Most importantly, it does not require an internet connection (It became my good pal during travels in Switzerland when I had no internet on the train! :P).
{{< /lead >}}

## 1. What is Git

Before delving into the commands, let's first get familiar with our new friend Git.

### 1.1. Easy Understanding of Git
Git is a free and open-source distributed **version** control system*** designed to handle everything from a small file to very large projects with speed and efficiency [^1]. For personal usage, it can be an easy and powerful tool for version control. For beginners, we can simply think about the "Version history" function in the Google Document but with more add-ons. For using for cooperating in a team, the light yet powerful design of Git enables a big development project to move forward in parallel effectively. Generally, it is always useful if you want to contribute to or try some repositories on GitHub.


### 1.2. Basic Elements of Git
Table 1.2.1 shows the definitions of some common jagons of Git. Having a general understanding of them before delving into the commands will make the following sections more friendly (so far I find the easiest way is to visualise all the Git activities, either in your mind, use tools or draw them down on a page). 


<a name="Table 1.2.1."></a>

<p align="center">  
<em>Table 1.2.1. Brief Introductions of Git Elements</em>
</p>

|    **Jagon**    | **Brief Explanation**                                                                                                                                                                                                                                             |
|:---------------:|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|  **Repository** | a storage space with all files and version history related to the project                                                                                                                                                                                         |
|    **Remote**   | a storage space on a remote server (like GitHub) that is connected to another repository, enabling project sharing and team collaboration                                                                                                                         |
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




Besides, [Figure 1.2.1.](git-relat.png) explains the basic relationships between each basic Git element. The interplay between them will be more clear as we explore deeper into Git.
<p align="center">  
<em>Figure 1.2.1. Relationships between Git Elements</em>
</p>

![git-relat](git-relat.png)

## 2. Let's Start with the Basic Local Commands

### 2.1. Creating New Things: Commit and Branch 
<u>***Concepts:***</u><br>
**Commit**: Snapshots of the project.<br>
**Branch**: Pointers to a specific commit. "I want to include the work of this commit and all parent commits".

<u>***Commands:***</u><br>
`git commit`: make a new commit <br>
`git branch newpost`: create a new branch named `newpost` <br>
`git checkout newpost`: put us on the branch `newpost`<br>
`git checkout -b newpost`: create a new branch `newpost` and check out `newpost`<br>

### 2.2. Redesigning Branches: Merge, Cherry-pick and Rebase
***Concepts:***<br>
**Merge**: Create a special commit that has two unique parents. "I want to include all the work from this parent over here and this one over here, *and* the set of all their parents."<br>
**Rebase**: Combine work between branches by making a nice linear sequence of commits. Copy a set of commits and plop them down somewhere else.<br>
**Cherry-pick**: Copy a series of commits below the current location.<br>
**Interactive rebasing**: Open up a UI to show you which commits are about to be copied below the target of the rebase. shows their commit hashes and messages.
- reorder commits by changing their order in the UI
- toggle of the `pick` button of the specific commits to drop them

<u>***Commands:***</u><br>
`git merge bugFix`: merge `bugFix` to `main`(checked out)<br>
`git rebase main`: move the work from `bugFix`(checked out) directly onto the `main` (add at the end)<br>
`git rebase main side`: rebase `side` onto `main`<br>
`git branch -f main HEAD~3`: move (by force) the main branch to three parents behind `HEAD`<br>
`git branch -f main C2`: move (by force) the main branch to `C2`<br>
`git rebase -i HEAD~4`: rebase the commit until 3 beforehand the `HEAD` (4 in total including `HEAD`) on to `main`<br>
`git rebase -i overHere`: over the UI of currently checked out branch and rebase the selected/reordered commits to `overHere`<br>
`git cherry-pick C1 C2`: copy C1, C2 below current location (`HEAD`)<br>

{{< alert "bell" >}}
Assume we are currently checked out on main branch.
`git merge A`: merge A to main;
`git rebase A`: rebase main onto A;
{{< /alert >}}



### 2.3. HEAD ^ and ~
***Concepts:***
HEAD: Currently checked out commit, always points to the most recent commit on this branch by default.

Detach HEAD: Attach it to a commit instead of tracing a branch.








# 4. Some suggestions
- Branch more and branch early;
- Squash and rebase will make the Git graph less hairy


This concept was inspired by a [post](https://datasci.social/@b0rk@jvns.ca/111375097434853441) by Julia Evans.

<div style="display:flex; gap:6px">

{{< badge >}} git {{< /badge >}}

{{< badge >}} summary {{< /badge >}}
</div>

[^1]: https://git-scm.com/
