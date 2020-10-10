# HOWTO move files between Git repositories preserving history

Some links on topic how to move files from one repository to another preserving
history:
- [GitHub article about moving files into fresh
  repo](https://help.github.com/articles/splitting-a-subfolder-out-into-a-new-repository/)
- [article about moving files into existing
  repository](http://gbayer.com/development/moving-files-from-one-git-repository-to-another-preserving-history/)

In few words:
- make a local copy of source repository
- use `git filter-branch` to filter history for files you need
- move filtering results under proper folder in source repository copy if
  required
- use `get remote add` or `git remote set-url` to set target repository as
  remote URL
- use `git pull --allow-unrelated-histories` to sync local and remote target
  repository
- do `git push`
