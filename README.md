# Git pull request tools

Everyone abstractly likes pull requests, but they can be a lot of keystrokes:
making new branches, pushing them, and especially keeping track of
what can be deleted.  This is an attempt to reduce the number of
keystrokes to a bare minimum.:

* `git pr branch BRANCH_NAME` - make a new branch based on inferred
  upstream HEAD.
* `git pr push -r` - push current branch to inferred origin,
  automatically make Github pull request.

Features:

* Create new PR branch: `git pr branch $name`
* Push PR branch: `git pr push $name`
* Automatically detect personal and upstream remotes names - when you
  need to start PRs, just add a remote if you need: best option based
  on names `local`, `upstream`, `origin` (see below).
* Anonymous PR branches (no local branch, only operating detached)
* Fetch existing PR branches by number
* Automatic diffs against upstream HEAD and PR branch: `git pr diff`
* Delete local and remote branches at same time: `git pr rm`
* Support for Gitlab and Github
* Support for automatically making PRs/MRs (Github and gitlab>11.10).
* Single shell script


## Installation

In this repository, you find a shell script `git-pr` which, when
placed anywhere on your `$PATH` and made executable (`chmod a+rx
git-pr`), provides the following commands.
There are no dependencies except POSIX shell (we hope - it's developed
in `dash`).

To automatically open Github pull requests, you must install the
[hub](https://github.com/github/hub) command line tool (Recent
Debian-based: `apt install hub`).  Gitlab pull requests require
git>=2.10 and Gitlab>=11.10.



## PR workflows

Here is our current short PR workflow (1):

1. `git pr branch $brname`: create a new branch based on inferred
   `upstream/HEAD`.  (note: fetch first if you want to be sure to be
   up to date)

2. Do work, commit, etc.

3. `git pr push [-r]`: Infer upstream remote automatically and push to
   branch matching local branch name.  The `-r` option automatically
   opens a MR/PR (Gitlab/Github).

5. Once you are done, `git pr rm $brname` to remove both local and
   remote branches (again inferring upstream remote).

There is actually an even shorter way (2):

1. `git pr branch`: create a detached head, don't even name it locally.

2. Do work, commit, etc.  If you change your mind, no need to remove
   anything.

3. `git pr push [-r] $brname`: push to inferred upstream.  Note you need to
   give a name since we don't have a local branch name.  The `[-r]`
   option again creates a pull request.

4. You don't need to remove anything - remove branch remotely (via
   Github web) or delete your fork if this was really a one-time
   thing.



## Common assumptions (repository setup)

- The upstream remote is `upstream`, `origin`, or the first remote in
  the `git remote` output, whichever is found first.  The idea is that
  if you add a remote named `upstream`, it will use that by default
  (if `origin` isn't it already the upstream).

- You push to the `local`, `origin`, `upstream`, or first remote in
  the `git remote` output, whatever comes first.  The idea is that if
  you cloned the upstream, if you
  add a remote called `local`, it will push to that by default.

Taken together, no matter if you originally clone the upstream or personal
fork, you can at most add one more remote and get started (without
having to rename any remotes).



## Usage

For each command, you can run `-h` to get help (with no arguments).
Only a brief description is shown here.

* `git pr branch`: create a new PR based on the current
  `(inferred_upstream)/HEAD`.  See `-h` for some considerations.  Also
  at the top of the script is a configuration option to force a
  `fetch` before to make sure you are up to date.  With one argument,
  create a branch of this name, otherwise create a detached head.

* `git pr push [-d] [-o] [[remote] branchname]`: Push a PR.  With no
  arguments, send to inferred
  origin automatically with a name the same as the current branch.
  With one argument, send to a branch of that name.  With two
  arguments, the first is the remote name to use, and the second is
  the branch name to push to.

  * `-f`: Force push
  * `-o`: Create a pull request at the same time (Github/Gitlab).
    Gitlab merge requests work with git>=2.10 and Gitlab>=11.10.

  When opening a pull request, these options are available:

  * `-n`: skip the "edit pull request message" step and instead use
    the message from the (first) commit. (Github)
  * `-d`: Will open as a draft pull request.  (Github)
  * `-b {branch_name}`: Target the named branch as the base branch to
    merge into (Github/Gitlab)

  Gitlab merge requests are only opened on invocations that actually
  push something, since this uses [git push
  options](https://docs.gitlab.com/ce/user/project/push_options.html).

* `git pr open`: Push and open a pull request.  This is completely
  equivalent to `git pr push -o`, see above for documentation.

* `git pr diff`: Diff between current working dir and merge-base of
  inferred_upstream.

* `git pr rm $branch_name ...`: Remove named branches, both locally
  and on inferred_origin.

* `git pr merged`: show local and remote branches which can be
  removed.  A small wrapper around `git branch --merged` that always
  checks relative to `$inferred_upstream/HEAD`.  (I welcome UI
  suggestions for this and the two following commands, how to properly
  do things automatically.)

* `git pr prune`: Remove remote tracking references already deleted
  upstream, and {local,remote} branches which are now merged to
  `$inferred_upstream/HEAD`.

* `git pr fetch $pr_number`: Fetch the given upstream PR to a new
  local remote branch `inferred_upstream/pr/$pr_number`. (all fetch
  commands support github.com and gitlab.com at least)

* `git pr fetchall`: Fetch all remote upstream PRs to local
  repository.  Warning: this includes all PRs, open and closed.

* `git pr unfetchall`: Remove all `$inferred_upstream/pr/[0-9]+$`
  branches (remove remote tracking branch is opposite of fetch).
  Warning: *all*.

* `git pr unfetchmerged`: Remove all PR branches (see above) branches
  which are merged (according to `--git branch --merged
  $inferred_origin/HEAD`).  Since you can't fetch just unmerged PRs,
  normally you would do `fetchall` followed by `unfetchmerged`.

* `git pr checkout $pr_number`: Locally check out a PR by number:
  simply a `git pr fetch $pr_number` followed by `git checkout
  pr/$pr_number`.

* `git pr main` (also aliased to `master`) will checkout the inferred
  main base branch (your local branch, not the remote tracking
  branch).  If it can't infer it from the remote's upstream, tries the
  order `main`, `master`, `gh-pages` and checks out the first that
  exists.

* `git pr info` prints the autodetected remotes and base branches.

* `git pr set-head` checks the remote and sets the remote default branch.

## Configuration

* **branch name prefix:** You may want to always prefix your branches
  with your name, e.g. `rkdarst/auto-prefix`.  You can set the a git
  variable using `git config --global git-pr.branchprefix PREFIX/`, and
  any branch you try to create will have this prefixed to it.  Note:
  include a trailing `/` or whatever character with your config
  option.  This is only applied if your `$inferred_upstream` is the
  same as your `$inferred_origin` (in other words, if you have a
  separate personal repository, this won't be added).



## Possible problems

We use the remote HEAD to infer what the upstream branch is.  There
are some problems with this:

- it is only set when first cloned, if default branch changes it won't
  be updated later.  However, you can manually set this with `git
  remote set-head ${inferred_upstream} $branch_name`

- If multiple branches had the same HEAD as the default branch, the
  remote default branch can't be inferred automatically.

- Setting the option `NEW_ALWAYS_FETCH=1` in the file solves this, at
  the cost of network access for `git pr branch`.


## Feedback

Please send feedback.  By its very nature, this makes some choices
about how workflows work.  If these can be improved to suit other
workflows, or you have ideas, please send pull requests.


## Other resources

* This is seems nice for automatically fetching PRs: https://gist.github.com/piscisaureus/3342247
* Related, similar but slightly less features: https://gist.github.com/gnarf/5406589
* Alias to fetch a single PR by ID: https://davidwalsh.name/pull-down-pr
* Has some super complicated aliases which I haven't examined yet:
  https://gist.github.com/metlos/9368527
* Big library of aliases (not read yet, no obvious relevant ones): https://github.com/Ajedi32/git_aliases
* Other projects of the same name
  * Fetches PRs using a PHP script: https://github.com/ozh/git-pr
  * May use Github CLI to make PRs: https://github.com/cladmi/git-pr/
* [Github CLI](https://cli.github.com/) (new April 2020) might take
  place of git-pr (but probably won't support Gitlab).


## License

All content is released under the MIT license.  In addition, code and
aliases are released into the public domain (CC0).  See LICENSE.

Note: the repository was called git-pr-tools until 2019-04-08.

`git-pr` is hosted under the NordicHPC umbrella.  Current primary
developer is Richard Darst, Aalto University (Finland) Science-IT.
