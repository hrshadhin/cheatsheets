# Git Cheatsheet
## Index
- [How to set up a global ignore file](#how-to-set-up-a-global-ignore-file)
- [Disable status advice](#disable-status-advice)
- [Automatically create upstream branches](#automatically-create-upstream-branches)
- [How to enable autocorrect](#how-to-enable-autocorrect)
- [Cleaning up local git branches deleted on a remote](#cleaning-up-local-git-branches-deleted-on-a-remote)
- [How to alias “master” as “main”](#how-to-alias-master-as-main)
- [How to undo commits](#how-to-undo-commits)
- [How to show and copy commit SHAs](#how-to-show-and-copy-commit-shas)
- [How to automatically stash while rebasing or merging](#how-to-automatically-stash-while-rebasing-or-merging)
- [Check for missing commits from a remote](#check-for-missing-commits-from-a-remote)
- [Comparing generated files before and after changes with git diff](#comparing-generated-files-before-and-after-ahanges-with-git-diff)
- [How to run a command on many files in a git repository](#how-to-run-a-rommand-on-many-files-in-a-git-repository)
- [Find usage for specific codes](#find-usage-for-specific-codes)
- [Check history with `git blame`](#check-history-with-git-blame)
- [Rebase stacked branches](#rebase-stacked-branches)
- [How to add changes to stacked branches](#how-to-add-changes-to-stacked-branches)
- [How to skip hooks](#how-to-skip-hooks)
- [Hooks](#hooks)

## How to set up a global ignore file
Make your global ignore file
```bash
mkdir -p $HOME/.config/git
touch $HOME/.config/git/ignore
```
contents of `$HOME/.config/git/ignore`
```ini
# OS
*~
.fuse_hidden*
.directory
.Trash-*
.nfs*
Thumbs.db
.DS_Store
.AppleDB

# Sublime Text
*.sublime-project
*.sublime-workspace

# vscode
.vscode/*
.history/
*.vsix

# Python
*.pyc
venv
.venv
env
.env
.tox

# Vim
# Swap
[._]*.s[a-v][a-z]
[._]*.sw[a-p]
[._]s[a-rt-v][a-z]
[._]ss[a-gi-z]
[._]sw[a-p]

Session.vim
Sessionx.vim
.netrwhist
tags
[._]*.un~
```

## Disable status advice
Many Git commands output “advice”, with hints about which commands you could run next. Most notably, git status gives you advice for what to do about files in each state:
```
git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
  new file:   cocobolo.txt

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
  deleted:    oak.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
  pine.txt
```

If you’re comfortable with Git, you might start to find these hints to be unnecessary “visual noise”. Thankfully, if you’d like the screen space back, you can disable the messages with the `advice.statusHints` option:
```
git config --global advice.statusHints false
```
`~/.gitconfig`:
```
[advice]
    statusHints = false
```
From then on, running git status will show only information:
```
git status
On branch main
Changes to be committed:
  new file:   cocobolo.txt

Changes not staged for commit:
  deleted:    oak.txt

Untracked files:
  pine.txt
````

## Automatically create upstream branches
started a new branch, worked hard on initial commits, and you’re ready to send it for review. You try to push and:
```
git push
fatal: The current branch cheese has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin cheese

To have this happen automatically for branches without a tracking
upstream, see 'push.autoSetupRemote' in 'git help config'.
```

Git requires the `--set-upstream` option to create an upstream branch

The `push.autoSetupRemote` option and the corresponding hint text were added in Git 2.37. To enable the option, run:

```
git config --global push.autoSetupRemote true
```

That command will add to your global configuration file (`~/.git/config/git` or `~/.gitconfig`):
```
[push]
    autoSetupRemote = true
```
From then on, git push on new branches will automatically create the branch.

## How to enable autocorrect

By default, if you mistype a Git command, it will list similar commands that you might have meant:
```
git comit -m "Some awesome work"
git: 'comit' is not a git command. See 'git --help'.

The most similar command is
  commit
```
To enable autocorrect, set the `help.autocorrect`:
```
git config --global help.autocorrect immediate
```

This will add to your `~/.gitconfig` file:
```
[help]
    autocorrect = immediate
```

With this in place, Git will automatically correct typo’d commands instantly:
```
git comit -m "Some awesome work"
WARNING: You called a Git command named 'comit', which does not exist.
Continuing under the assumption that you meant 'commit'.
[main 25112f085] Some awesome work
```

**Prompt to run**

If automatically proceeding to run the closest matched command makes you a little nervous, you can instead use the “prompt” mode:
```
git config --global help.autocorrect prompt
```
Typos will then show a `yes/no` prompt to correct:

```
git comit -m "More awesome work"
WARNING: You called a Git command named 'comit', which does not exist.
Run 'commit' instead [y/N]?
```
Press “y” then “enter” to accept the correction:
```
git comit -m "More awesome work"
WARNING: You called a Git command named 'comit', which does not exist.
Run 'commit' instead [y/N]? y
[main 2183b892e] More awesome work
```

## Cleaning up local git branches deleted on a remote

clean up squash-merging or branches that not in remote but have in locally.

- check history log:
  ```
  git log --graph --oneline
  git log --graph --oneline --all
  ```
- list branch with upstream status:
  ```
  git branch --format '%(refname:short) %(upstream:track)'
  ```
- list only upstream deleted branches
  ```
  git branch --format '%(refname:short) %(upstream:track)' | awk '$2 == "[gone]" { print $1 }'
  ```
- delete untrack remote branches on locally
  ```
  git branch --format '%(refname:short) %(upstream:track)' | awk '$2 == "[gone]" { print $1 }' | xargs -r git branch -D
  ```
- Alias to delete untrack branches
  ```
  git config --global alias.rm-merged '!git branch --format '\''%(refname:short) %(upstream:track)'\'' | awk '\''$2 == "[gone]" { print $1 }'\'' | xargs -r git branch -D'
  ```
  That will add an alias to your `~/.gitconfig`:
  ```
  [alias]
      rm-merged = !git branch --format '%(refname:short) %(upstream:track)' | awk '$2 == \"[gone]\" { print $1 }' | xargs -r git branch -D
  ```

- You can then run the `rm-merged` alias as part of your workflow like so:
  ```
  git switch main
  git pull --prune
  git rm-merged
  ```
- what about an alias to do this `main, pull, rm-merged` dance in one?
  ```bash
  git config --global alias.sync '!git switch main && git pull --prune && git rm-merged'

  # in action:
  git sync
  ```

## How to alias “master” as “main”
Git 2.28 (2020-07-27) introduced the init.defaultBranch option, which controlls the default branch name for repos created with git init:

```
git config --global init.defaultBranch main
```

**Aliasing**
It can be annoying to work on legacy repos that still have a “master” branch. Your muscle memory or command aliases might use “main”, causing your commands to fail:
```
git switch main
fatal: invalid reference: main
```
Luckily, Git offers a solution: local aliases, called `symbolic refs`, which you can configure with `git symbolic-ref`. You can make main an alias for master like so:
```
git symbolic-ref refs/heads/main refs/heads/master
````

And `origin/main` an alias for `origin/master` with:
```
git symbolic-ref refs/remotes/origin/main refs/remotes/origin/master
```
You can then switch to the aliased main branch:
```
git switch main
Switched to branch 'main'
```
…nd use main anywhere you’d previously need master:
```
git rebase -i main
...
git log origin/main..example
...
```
Git will show both main and master in some views, like `git log`:
```
git log --oneline
0e78b70 (HEAD -> master, main) Make the damn thing
275a9fd (origin/master, origin/main) Initial commit
```
**An alias to alias**

You can add such a command alias with this one-liner:
```bash
git config --global alias.alias-master-as-main '!git symbolic-ref refs/heads/main refs/heads/master && git symbolic-ref refs/remotes/origin/main refs/remotes/origin/master && git switch main'
```

This will add an entry in your `~/.gitconfig`:
```bash
[alias]
    alias-master-as-main = !git symbolic-ref refs/heads/main refs/heads/master && git symbolic-ref refs/remotes/origin/main refs/remotes/origin/master && git switch main
```

Now, when you need to add a main alias to a new legacy repo, you can run:
```
git alias-master-as-main
Switched to branch 'main'
```
**If the remote updates**

First, remove your symbolic refs, with `-d` for delete:
```
git switch master
git symbolic-ref -d refs/heads/main
git symbolic-ref -d refs/remotes/origin/main
```

Second, update to use the remote repository:
```
git branch -m master main
git fetch origin
git branch -u origin/main main
git remote set-head origin -a
git fetch --prune
```

## How to undo commits
- **How to undo the last commit**
  ```bash
  git reset @~

  # Use reset --soft to keep files staged. so you don’t need to git add them again
  git reset --soft @~

  # Use reset --hard to discard changes
  git reset --hard @~

  # Undo several commits with `@~<n>` `n` is number of commits
  # Combine with `--soft` or `--hard` as appropriate
  git reset @~3
  ```
- **How to undo previous commits**

  To undo a previous commit with a new `revert commit`, use `git revert ` with the commit’s SHA:
  ```
  git revert <commit>
  ```
  This will create a new commit that undoes the changes made in that previous commit.

  Use `revert -n / --no-commit` to revert without committing:
  ```
  git revert -n <commit>
  ```
  Undo several previous commits. You can pass several commit references to git revert to undo them all:
  ```
  git revert <commit1> <commit2> ...
  git revert 5c87687 1c13e39
  ```
  Git will revert each one at a time, potentially pausing for a merge conflict at each step. Git will open your editor for each commit message in turn.

  If you want to end up with a single revert commit, you can use -n:
  ```
  git revert -n 5c87687 1c13e39
  git revert --continue
  ```

- **How to rewrite history to remove previous commits**

  To undo a previous commit by removing it from history, you’ll need to use an interactive rebase. You’d normally only want to this on a feature branch that hasn’t been merged to your main branch.

  In short, start an interactive rebase with:
  ```
  git rebase -i main
  ```
  Git pops open your editor with the rebase plan:
  ```
  pick 5c87687 Loosen soil
  pick 1c13e39 Add compost

  # Rebase 07a4556..1c13e39 onto 07a4556 (2 commands)
  #
  # Commands:
  ...
  ```
  To remove a commit from history, you can simply delete its line. (Or replace “pick” with the “d” (“drop”) command, if you want to be explicit.)

  After you save and close the file, Git will continue. 

  > Note that its SHA has changed (from 1c13e39 to bd86d96). This is because it has been recreated. Git’s history is immutable, so rebasing recreates commits, reusing the same message, authorship, and changes, but new parents.


## How to show and copy commit SHAs
You can use `git rev-parse` to show the SHA of the current Git commit:
```
git rev-parse @
44a4ec1ffe634759fd25a9e87a6c555fe83f7d64
```
`@` is short for `HEAD`, Git’s name for the current commit.

Use `--short` for the abbreviated SHA:
```
git rev-parse --short @
44a4ec1
```
To show the SHA of a branch or tag, use its name:
```
git rev-parse main
44a4ec1ffe634759fd25a9e87a6c555fe83f7d64
```
Remote branch and tag references also work:
```
git rev-parse origin/main
385bb2ff3d2ce74690a1583eefa216ed61e8d28a
```
Copy to clipboard
```bash
# Mac OS
git rev-parse @ | tr -d '\n' | pbcopy

# GNU/Linux
git rev-parse @ | tr -d '\n' | xclip

# Within shell commands
# When templating the SHA into another command, the `--sq` option may come in handy. This option adds shell quoting to the output, and removes the trailing newline:
git rev-parse --sq @
'44a4ec1ffe634759fd25a9e87a6c555fe83f7d64'
```

## How to automatically stash while rebasing or merging
Rebase your branch onto main. To do so with automatic stashing and unstashing, use the `--autostash` option:
```
git rebase -i main --autostash
```
Enable `rebase.autoStash` in your global Git config to enable rebase autostashing by default:
```
git config --global rebase.autoStash true
```
This will add to your `~/.gitconfig`:
```
[rebase]
    autoStash = true
```
No need to pass `--autostash` anymore. If you want to run a rebase without autostashing, you can pass `--no-autostash`:
```
git rebase -i main --no-autostash
```

**How to merge with `autostash`**

git merge also has an `--autostash` option, which acts similarly:
```
git merge --autostash main
```

You can also enable this globally with merge.autoStash:

```
git config --global merge.autoStash true
```

`~/.gitconfig`:
```
[merge]
    autoStash = true
```
Equally, use `--no-autostash` to disable the behaviour for a particular git merge invocation.

## Check for missing commits from a remote
```bash
git rev-list HEAD..origin/master
```

## Comparing generated files before and after ahanges with git diff

```bash
# move to previous commit from HEAD
git switch --detach master~1
bundle exec jekyll build
mv _site _site_old

# back to latest commit
git switch master
bundle exec jekyll build

git diff --no-index _site_old _site
```

## How to run a rommand on many files in a git repository
```bash
git ls-files -- '*.py' | xargs wc -l
```

## Find usage for specific codes
```bash
git grep splash
# limit the search
git greap splah -- '.html' '.js'

# also use ripgrep(rg) for faster search in files

# check history with git log
git log -p --stat -S splash
git log -p --stat -S splash -- '*.html' '*.js'
git log -p --stat --pickaxe-regex -S '\bsplash\b'
```

## Check history with `git blame`
```bash
git blame -- example/static/main.css
git show 1234abcdef
# `~1` mean the commit before
git blame 1234abcdef~1 -- example/static/main.css
```

## Rebase stacked branches
Imagine you have this situation, most recent commits first:
```
* e67fe90 Add deployment (main)
|
| * 73145a7 Add Poll views (HEAD -> poll_views)
| |
| * 3345a24 Add Poll database models (poll_models)
|/
|
* 86e3722 Set up Django
```
First, switch to the final feature branch:
```bash
git switch poll_views
```
Second, rebase onto the `main` branch with `--update-refs`:
```bash
git rebase --update-refs main
```
Git tells about poll_views being rebased, and that it update poll_models in the process.

Now both branches live on top of the latest commit on `main`:
```
* c6ac1a3 Add Poll views (HEAD -> poll_views)
|
* 9f9622b Add Poll database models (poll_models)
|
* e67fe90 Add deployment (main)
|
* 86e3722 Set up Django
```

## How to add changes to stacked branches
Imagine you had that same initial situation:
```
bc63397 Add deployment (HEAD -> main)
|
| * 94d92fd Add Poll views (poll_views)
| |
| * 229c030 Add Poll database models (poll_models)
|/
|
 * 86e3722 Set up Django
```
After submitting `poll_models` and `poll_views` for code review, you have some changes to make to both. How can you make those changes, and ensure they end up in the right branches?
- First, check out the last of the stacked branches:
  ```
  git switch poll_views
  ```
-  Second, make changes applicable to both branches:
    ```
    ...
    git commit -m "Add database constraint to Poll model"
    ...
    git commit -m "Add rate-limiting to view"
    ...
    ```
-  Third, start an interactive rebase of the stack:
    ```
    git rebase -i --update-refs main
    ```
- Fourth, change the rebase file to move the commits to the appropriate branches. When the file opens, you’ll see it starts like:
    ```
    pick 229c030 Add Poll database models
    update-ref refs/heads/poll_models

    pick 94d92fd Add Poll views
    pick 31d5fc9 Add database constraint to Poll model
    pick af05cde Add rate-limiting to view

    # Rebase bc63397..af05cde onto bc63397 (5 commands)
    #
    # Commands:
    ...
    ```
The pick lines list commits, in the order they’ll be rebased. The update-ref line controls the commit that the poll_models branch will point to after the rebase.
In this case, the “Add database constraint to Poll model” commits should be part of poll_models. Move its line above the update-ref line:
  ```
  pick 229c030 Add Poll database models
  pick 31d5fc9 Add database constraint to Poll model
  update-ref refs/heads/poll_models

  pick 94d92fd Add Poll views
  pick af05cde Add rate-limiting to view

  # Rebase bc63397..af05cde onto bc63397 (5 commands)
  ...
  ```
- Fourth, save and close the file, and Git will perform the instructed rebase:
  ```
  git rebase -i --update-refs main
  Successfully rebased and updated refs/heads/poll_views.
  Updated the following refs with --update-refs:
    refs/heads/poll_models
  ```

Now the two branches are on top of the latest main, with their respective commits:
  ```
  * 9182779 Add rate-limiting to view (HEAD -> poll_views)
  |
  * df9ae26 Add Poll views
  |
  * d0cfd78 Add database constraint to Poll model (poll_models)
  |
  * e7ee0b7 Add Poll database models
  |
  * bc63397 Add deployment (main)
  |
  * 86e3722 Set up Django
  ```

To enable the flag by default, for all repos, add the `rebase.updateRefs` boolean option to your global config:
```bash
git config --global --add --bool rebase.updateRefs true
```
This will add the option in your `~/.gitconfig` file like so:
```
[rebase]
    updateRefs = true
```

## How to skip hooks
**Skip most hooks with `--no-verify`**

Git lets you bypass most hooks with a `--no-verify` option on triggering commands. For example, you can use `commit --no-verify` to skip the pre-commit hook:
```
git commit --no-verify -m "hack da prototype"
```
Uniquely among such commands, git commit supports `-n` as a short option for `--no-verify`, so you can shorten the above:
```
git commit -nm "double hack da prototype"
```

## Hooks
Path: `<repo>/.git/hooks/`

**Pre-commit**
- `<repo>/.git/hooks/pre-commit`
  ```bash
  #!/bin/bash

  cd "$(dirname "$0")/pre-commit.d"

  for hook in *; do
      ./$hook
      RESULT=$?
      if [ $RESULT != 0 ]; then
          echo "pre-commit.d/$hook returned non-zero: $RESULT, abort commit"
          exit $RESULT
      fi
  done

  exit 0
  ```
- `<repo>/.git/hooks/pre-commit.d/missing-commit.py`
  ```python
  #!/usr/bin/env python

  import subprocess
  import sys


  def check_git_includes_origin_master():
      subprocess.run(["git", "fetch", "origin"])
      rev_list_run = subprocess.run(
          ["git", "rev-list", "HEAD..origin/master"],
          stdout=subprocess.PIPE,
          stderr=subprocess.PIPE,
          universal_newlines=True,
      )
      missing_commits = rev_list_run.stdout.splitlines()
      if missing_commits:
          print(
              (
                  f"Current git status is missing {len(missing_commits)}"
                  + f" commit{'s' if len(missing_commits) > 1 else ''} from"
                  + f" origin/master. Please pull master and if necessary "
                  + f"rebase the current branch on top."
              ),
              file=sys.stderr,
          )
          sys.exit(1)


  def check_git_includes_origin_develop():
      subprocess.run(["git", "fetch", "origin"])
      rev_list_run = subprocess.run(
          ["git", "rev-list", "HEAD..origin/develop"],
          stdout=subprocess.PIPE,
          stderr=subprocess.PIPE,
          universal_newlines=True,
      )
      missing_commits = rev_list_run.stdout.splitlines()
      if missing_commits:
          print(
              (
                  f"Current git status is missing {len(missing_commits)}"
                  + f" commit{'s' if len(missing_commits) > 1 else ''} from"
                  + f" origin/develop. Please pull develop and if necessary "
                  + f"rebase the current branch on top."
              ),
              file=sys.stderr,
          )
          sys.exit(1)



  if __name__ == "__main__":
      check_git_includes_origin_master()
      #check_git_includes_origin_develop()
  ```
