---
layout: post
title: Seamless pushing to multiple remote git repositories.
---

A couple of years ago we had a need in the team to keep 2 remote repositories in sync. 
A couple of services, devop scripts and other tools were evaluated, 
but we ended up using the following little hack to treat both remotes as one.

This hack will not work with proper git commands, hence the hack. Open up `.git/config` file in an editor of your choice.
You will see something similar to the following contents:

```
...
[remote "origin"]
    url = https://github.com/<username>/<repo-name>.git
    fetch = +refs/heads/*:refs/remotes/origin/*
...
```

Simply add another `url` entry to that remote's configuration like so:

```
...
[remote "origin"]
    url = https://github.com/<username>/<repo-name>.git
    url = https://gitlab.com/<username>/<repo-name>.git
    fetch = +refs/heads/*:refs/remotes/origin/*
...
```

Now, try to push to that remote with `git push origin master`.

This will push the changes to the `master` branch on the remote repositories configured by their urls.
So, basically tou treat the 2 repositories as if it was one.