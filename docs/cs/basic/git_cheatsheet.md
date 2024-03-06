# Git Cheatsheet

## Commands

### Basis

```shell
> git init: to create `.git` in the folder
> git status: what is the status of the current repository
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
> git reset <hash>: switch to certain commit
	git reset --hard HEAD: throw away all the changes, back to the HEAD commit
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

## [Write Good Git Commits](https://cbea.ms/git-commit/#capitalize)

### Why good commit messages matter

- Keep a healthy commit history
- Make the git log more beautiful
- Uniform commit rules, make it easier for others to understand

### How to write great Git commit message

In order to create a useful revision history, teams should first agree on a commit message convention that defines at least the following three things:

#### Style. 

Markup syntax, wrap margins, grammar, capitalization, punctuation. Spell these things out, remove the guesswork, and make it all as simple as possible. The end result will be a remarkably consistent log that’s not only a pleasure to read but that actually does get read on a regular basis.

#### Content. 

What kind of information should the body of the commit message (if any) contain? What should it not contain?

#### Metadata. 

How should issue tracking IDs, pull request numbers, etc. be referenced?

#### Seven Basic Rules

1. Separate subject from body with a blank line
2. Limit the subject line to 50 characters
3. Capitalize the subject line
4. Do not end the subject line with a period
5. Use the imperative mood in the subject line
6. Wrap the body at 72 characters
7. Use the body to explain **what** and **why** vs. **how**

Example:
```
Summarize changes in around 50 characters or less

More detailed explanatory text, if necessary. Wrap it to about 72
characters or so. In some contexts, the first line is treated as the
subject of the commit and the rest of the text as the body. The
blank line separating the summary from the body is critical (unless
you omit the body entirely); various tools like `log`, `shortlog`
and `rebase` can get confused if you run the two together.

Explain the problem that this commit is solving. Focus on why you
are making this change as opposed to how (the code explains that).
Are there side effects or other unintuitive consequences of this
change? Here's the place to explain them.

Further paragraphs come after blank lines.

 - Bullet points are okay, too

 - Typically a hyphen or asterisk is used for the bullet, preceded
   by a single space, with blank lines in between, but conventions
   vary here

If you use an issue tracker, put references to them at the bottom,
like this:

Resolves: #123
See also: #456, #789
```