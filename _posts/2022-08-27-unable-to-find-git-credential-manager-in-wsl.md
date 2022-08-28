---
layout: post
title:  "Unable to find git credential manager in WSL"
categories: coding git
---


Some time ago, I was working with Git on WSL using VSCode and was unable to push/pull/fetch. I kept being asked for my GitHub credentials. Furthermore, the git log showed the following:

`/mnt/c/Program\ Files/Git/mingw64/libexec/git-core/git-credential-manager-core.exe get: 1: /mnt/c/Program Files/Git/mingw64/libexec/git-core/git-credential-manager-core.exe: not found`

## The solution

On WSL2, execute the following command:

```bash
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/libexec/git-core/git-credential-wincred.exe"
```

## Probable cause

Apprarently, `git-credential-manager-core.exe` was renamed to `git-credential-wincred.exe`
I did do an update that day on my Windows machine with Chocolatey to update git to *git 2.36.1*, so I think this caused this.
