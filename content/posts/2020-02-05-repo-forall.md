---
title: "Google git-repo: forall"
description: ""
date: 2020-02-05T00:10:09+01:00
draft: false
---

Google's [repo tool][gerrit-git-repo] tool can run a command in all of its
tracked repositories.

Use `-p` to also show the current repository when running.

Example usage: See all the changes a security patch bump introduced:
```
repo forall -p -c \
  "/usr/bin/git --no-pager \
    diff android-10.0.0_r18..android-10.0.0_r25 \
    2>/dev/null"
```

Another variant, suggested by [Jean-Baptiste Queru][jbq] back in the day of 4.x:
```
repo forall -c '
if git rev-parse android-4.0.3_r1 >/dev/null 2>&1
then
git log --oneline --no-merges android-4.0.3_r1..android-4.0.4_r1.1
else
git log --oneline --no-merges android-4.0.4_r1.1
fi
' | cat
```

Find out all commits that you cherry-picked during bringup:
```
repo forall -p -c \
    "/usr/bin/git \
    --no-pager log HEAD...android-r-preview-2 \
    2>/dev/null"
```
Note: The `...` triple dot.

Explanation, see [stackoverflow][so]:
> `git log a..b` means give me all commits that were made since a, until and
> including b (or, like the man page puts it "Include commits that are reachable
> from b but exclude those that are reachable from a"), the three-dot variant
>
> `git log a...b ` means "Include commits that are reachable from either a or b
> but exclude those that are reachable from both", which is a totally different
> thing.

[gerrit-git-repo]: https://gerrit.googlesource.com/git-repo
[jbq]: https://groups.google.com/forum/#!msg/android-building/0DtsHawjs4k/And8o3Dni_UJ
[so]: https://stackoverflow.com/questions/18595305/git-log-outputs-in-a-specific-revision-range#18595347
