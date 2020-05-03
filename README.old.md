# Aliases (obsolete)

These are old aliases which predated and somewhat reproduce the
functionality of the `git-pr` script.  If you can use the `git-pr`
script, it is more powerful, but these old one-liners may still be
useful to someone.

```shell
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
