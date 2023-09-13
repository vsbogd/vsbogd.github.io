# HOWTO get specific revision from Git repo without getting history

Sometimes you need to get the only revision without history and need get it as
fast as possible. For example it is useful for CI jobs. `git clone` doesn't
allow doing this, but you can use `get fetch` instead.

The shell function initializes new repo, fetches the only required revision
without history and resets `HEAD` to the fetched revision:
```
git_clone_rev() {
    dir=$1
    url=$2
    rev=$3

    mkdir -p ${dir}
    cd ${dir}
    git init
    git remote add origin $url
    git fetch --depth=1 origin $rev
    git reset --hard FETCH_HEAD
}

git_clone_rev <dir> <git-remote-url> <git-revision>
```
