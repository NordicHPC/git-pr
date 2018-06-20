# Git pull request tools

Everyone likes pull requests, but they can be a lot of keystrokes:
making new branches, pushing them, and especially keeping track of
what can be deleted.

## PR workflows

Here is our current short PR workflow (1):

1. `git prnew $brname`: create a new branch based on inferred
   `upstream/HEAD`.  (note: we don't currently automatically
   re-fetch).

2. Do work, commit, etc.

3. `git prpush`: Infer current branch name and push to inferred
   remote.  You can give a different name to push to a different
   branch.

4. Once you are done, `git prrm $brname` to remove both local and
   remote branches (again inferring upstream remote).

There is actually an even shorter way (2):

1. `git prnew`: create a detached head, don't even name it locally.

2. Do work, commit, etc.

2. `git prpush $brname`: push to inferred upstream.  Note you need to
   give a name since we don't have a local name.

3. You don't need to remove anything - remove branch remotely (via
   Github web) or delete your fork if this was really a one-time
   thing.



Common assumptions:

- The upstream remote is `upstream`, `origin`, or the first remote in
  the `git remote` output, whichever is found first.

- You always push to the `origin` remote.


## Aliases

Currently commands are distributed as aliases.

```
# Create a new HEAD suitable for a pull request.  Use
# upstream/HEAD, origin/HEAD, or (the first remote)/HEAD as
# the base.  If a argument is given, create a branch of this
# name, otherwise create a detached head here.
prnew = !sh -x -c 'git checkout $(test $1 && echo "-b" $1) $(git remote | grep upstream || git remote | grep origin || git remote | head -1)/HEAD' -

# Push a PR branch.  If one argument given, push to that
# branch on guessed remote (same logic as above).  This is
# suitable for the local detached heads workflow.  If two
# arguments given, the first is a remote and the second is
# branch name.  If no arguments given, use the current branch
# name as the remote branch name (only works if you have a
# local branch).
prpush = !sh -x -c 'git push $(test $2 && echo $1 || echo origin) HEAD:refs/heads/${2:-${1:-$(git symbolic-ref HEAD | cut -d/ -f3)}}' -

# Compare current working tree to upstream merge-base.  Uses
# same logic to find upstream merge base as "prnew"
prdi = !git diff $(git merge-base $(git remote | grep upstream || git remote | grep origin || git remote | head -1)/HEAD HEAD)

# Delete a branch (argument 1) both locally and remotely.
prrm = !sh -x -c "'git branch | grep $1 && git branch -d $1 ; git branch -a | grep origin/$1 && git push origin :$1'" -


# Fetch PR refs to local.  Give one PR numbers, and they will
# be pulled as $remote/pr/$number.  (This should be modified
# to take multiple PR numbers)
prfetch = !sh -x -c 'git fetch $(git remote | grep upstream || git remote | grep origin || git remote | head -1) --refmap="+refs/pull/*/head:refs/remotes/$(git remote | grep upstream || git remote | grep origin || git remote | head -1)/pr/*" refs/pull/$1/head' -

# Fetch all PRs from a certain remote
prfetchall = !sh -x -c 'git fetch upstream "+refs/pull/*/head:refs/remotes/$(git remote | grep upstream || git remote | grep origin || git remote | head -1)/pr/*"' -

# Delete everything fetched by prfetchall.  Note, this deletes
# these refs from *all* remotes, not just the default
# upstream.  It will delete anything matching '/pr/[0-9]+'
prunfetchall = !sh -x -c 'git branch --remote -d `git branch --remote | grep -E '/pr/[0-9]+$'`' -
```

## Other resources

* This is seems nice for automatically fetching PRs: https://gist.github.com/piscisaureus/3342247
* Related, similar but slightly less features: https://gist.github.com/gnarf/5406589
* Alias to fetch a single PR by ID: https://davidwalsh.name/pull-down-pr
* Has some super complicated aliases which I haven't examined yet:
  https://gist.github.com/metlos/9368527
* Big library of aliases (not read yet, no obvious relevant ones): https://github.com/Ajedi32/git_aliases

