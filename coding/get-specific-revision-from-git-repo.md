# HOWTO get specific revision from Git repo without getting history

Sometimes you need to get the only revision without history and need get it as
fast as possible. For example it is useful for CI jobs. `git clone` doesn't
allow doing this, but you can use `get fetch` instead.

Code below assumes the following variables contain the parameters:
```
export PROJECT=<dir-name>
export PROJECT_URL=<git-remote-url>
export PROJECT_REV=<git-revision>
```

The recipe initializes new repo, fetches the only required revision without
history and resets `HEAD` to the fetched revision:
```
mkdir -p ${PROJECT}
cd ${PROJECT}
git init
git remote add origin $PROJECT_URL
git fetch --depth=1 origin $PROJECT_REV
git reset --hard FETCH_HEAD
```
