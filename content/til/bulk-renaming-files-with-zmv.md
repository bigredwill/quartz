---
title: Bulk renaming files with zmv
---

zmv - zshell move

Apple Source for zmv [https://opensource.apple.com/source/zsh/zsh-65/zsh/Functions/Misc/zmv.auto.html]()


Really nice utility to bulk rename files matching a glob pattern. No messing with for loops and sed.

Don't forget to escape your parentheses.

Given a file name like `Artist Name (Flowers) - song title.mp3`:
```sh
zmv 'Artist Name \(Flowers\) - (*)' '$1';
```
Will result in `song title.mp3`.

Good overview blog article [https://blog.smittytone.net/2021/04/03/how-to-use-zmv-z-shell-super-smart-file-renamer/]()
