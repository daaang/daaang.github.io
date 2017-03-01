---
layout: post
title:  "Just wrote a git-vain implementation in rust"
date:   2017-03-01 18:45:23 -0500
---
There are currently two versions of [git-vain][1], in C and in ruby. The
ruby version is understandably slow, and the C version depends on
XCode's SHA1 hasher.

I could have broken that dependency, but I decided it was a fun excuse
to write a rust program, so that's what I did. Now my `git-vain` runs
search for five bytes instead of four.

Pretty cocky.

[1]: https://github.com/will/git-vain
