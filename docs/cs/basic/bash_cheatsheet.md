# Bash Cheatsheet

## Git

### Basis

```shell
> git init: to create `.git` in the folder
> git status: what is the status of the current repository  # see your changes
> git add <files> / git add .
> git commit -m "commit-message"
> git log --all --graph --decorate: visualizes history as a DAG
> git checkout address: enter different nodes (change pointer `HEAD`)
> git checkout <file>: throw away the changes in the working directory and set it back as the version of `HEAD` 
> git diff <file>: find differences with the last snapshot (known as `HEAD`)
> git diff pointer/address <file>: find differences with specific snapshot
> git diff address1 address2 <file>: find differences between two snapshot
```

### Branching and Merging

```shell
> git branch: list all the branches that are presented in the local directory
> git branch <name>: creates a branch (its parent is `HEAD`)
> git checkout -b <name>: creates a branch and switches to it
# same as git branch <name>; git checkout <name>
> git merge <revision>: merges into current branch
> git merge --abort: undo the last "git merge"
> git mergetool: use a fancy tool to help resolve merge conflicts
> git rebase: rebase set of patches onto a new base
> git branch -vv: tell all the branches
```

### Git Reset

```shell
> git reset HEAD <files>: undo the operation `git add`
> git reset --soft HEAD^: undo the operation `git commit`
> git reset --hard HEAD: throw away all the changes, back to the HEAD commit
> git commit --amend: just modify the commit message(entering vim)
```

### Git Remote

```shell
> git remote: list remotes
> git remote add <name> <url>: add a remote and name it.
> git push <remote> <local branch>:<remote branch>: send objects to remote, and update remote reference
> git branch --set-upstream-to=<remote>/<remote branch>: set up correspondence between local and remote branch
# then I can simply use `git push`(In github, it has already be done)

> git fetch: retrieve objects/references from a remote
> git pull: same as git fetch; git merge
# git is very smart, it will always keep the latest version, do you don't have to worry that this command will cover your local changes.
> git clone: download repository from remote
# git clone will clone the whole history from remote, and use `git clone --shallow` can just clone the latest snapshot
```

### Git Stash

> When using `git pull`, you might encounter the error `Your local changes would be overwritten by merge` because you have make some changes to the last commit. If you don't want to commit your current changes, you can temporarily push your changes to a stack.

> With reference to: https://blog.csdn.net/a112626290/article/details/106981153

```bash
> git stash # back up the changes to the stack; use `git status` after this command you will find no changes
> git stash list [<options>] # list all the backups
> git stash show [<options>] [<stash>] 
> git stash drop [-q|--quiet] [<stash>] # drop the backup at the top
> git stash ( pop | apply ) [--index] [-q|--quiet] [<stash>] 
# apply: merge the top backup with current workspace
# apply + drop, files that have conflicts won't be deleted
> git stash branch <branchname> [<stash>]
> git stash [push [-p|--patch] [-k|--[no-]keep-index] [-q|--quiet]
	     [-u|--include-untracked] [-a|--all] [-m|--message <message>]
	     [--] [<pathspec>…]]
> git stash clear # clear the stack
> git stash create [<message>]
> git stash store [-m|--message <message>] [-q|--quiet] <commit>
```

### Others

```shell
> git add -p <file>: interact with the terminal to upload the changes
> git diff --cached: show the changes in the staged area(compared with last snapshot)
> git blame: who edit each line of the file
> git show: get information of specific commit
> git stash: temporarily remove modifications to working directory
# git stash pop: undo
> git bisect: binary search history (e.g. for regressions)
```



## Tmux Cheatsheet

```bash
tmux new -s <session-name> # set up a new session
<prefix> d || tmux detach # hang up the session and exit
tmux ls # list all sessions
tmux attach -t <name> # attach to specific session
tmux kill-session -t <name> # kill specific session
<prefix> $ || tmux rename-session -t <name> <newname> # rename current session
<prefix> m # Mouse mode toggle
<prefix> - # splits the current pane vertically
<prefix> _ # splits the current pane horizontally
```

For more information: https://github.com/gpakosz/.tmux

