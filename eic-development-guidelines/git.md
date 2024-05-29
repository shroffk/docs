
# Git Basics

This guide serves as a summary of [Atlassian's Git Tutorial](https://www.atlassian.com/git/glossary#commands)

For more tutorials:
* [https://www.jetbrains.com/help/clion/using-git-integration.html CLion Git Tutorial]
* [https://code.visualstudio.com/docs/sourcecontrol/intro-to-git VSCode Git Tutorial]
* [https://www.jetbrains.com/help/pycharm/using-git-integration.html PyCharm Git Tutorial]
* [https://www.jetbrains.com/help/idea/using-git-integration.html IntelliJ Git Tutorial]
* [https://ohmygit.org OhMyGit]
* [https://learngitbranching.js.org Learn Git Branches]


### git clone
- Git clone is used to replicate a repository from Github
- A clone copies an existing repo this is not a file-based interaction
- A cloned repository has it's own history, manages, it's own files, completely isolated environment from the original repository
``` bash
git clone git@github.com:organization/
cd my-project 
# Start working on the project
```

### git branch
**Reasons to use branches**
1. Collaboration: avoid merge conflicts
2. Long Term Development: 
    - If a feature takes 1-2 weeks to develop you should be pushing changes as you work
    - Pushing this developement to master ruins the opportunity for someone else to work on a new feature or revert to an old version in case of issues
3. Multiple Feature Development 
    - If you need to work on 2 features at once, maybe one feature takes less time than expected and is ready to be released
    - If both are on your local master you must go back and remove code for feature 2 or wait to release until both features are completed
        - what if one feature had an issue? Both would need to be reverted
**how to use:**
- Branches are used to make a local copy of cloned respository where changes can be made without affecting master
- This helps keep code safe!
[Examples and more information](https://www.atlassian.com/git/tutorials/using-branches)
``` bash
git branch
* master

git checkout -b demo-branch
Switched to a new branch 'demo-branch'

git branch 
* demo-branch
  master

git checkout master
Your branch is up to date with 'origin/master'.

git branch
  demo-branch
* master

git branch -d demo-branch
Deleted branch demo-branch (was 216f16e).

git branch
* master
```

### git add
Git add moves a file change from the working directory to the staging area. Git now knows these updates to a file are to be added to the next commit. This does not affect the repository until you run git commit.
**Common Usage:**
- git add <file>
- git add <directory>
- git add . 

### git commit
Git commit captures the state of a project at that point in time and prepares all changes to be added to the repository after a git push.
**Common Usage:**
- git commit
- git commit -a 
- git commit -m "commit message"
- git commit -am "commit message"
- git commit --amend

### git push
Git push is used to upload local repository content to a remote respository. Commits are transferred from your local repository to the remote repository during a push.
**Common Usage:**
- git push <remote> <branch>
- git push <remote> --force

### git pull
Git pull is used to fetch and download content from a remote repository and immediately update the local repository to match that content. Merging remote upstream changes into your local repository is a common and important task in Git. The git pull command first runs git fetch which downloads content from the specified remote repository. Then a git merge is executed to merge the remote content refs and heads into a new local merge commit.
**Common Usage:**
- git pull
- git pull --no-commit <remote>
- git pull --rebase <remote>
 

### git diff
Used for taking 2 git data sources and comparing them. These data sources include: commits, branches, files, and more. By default git diff will show you any uncommitted changes since the last commit. 
**Common Usage:**
- git diff branch1..branch2
- git diff branch1 branch2 <file name>

### git status

### merge requests

### merge conflicts

### pull requests

### rebase

### reset

### revert

