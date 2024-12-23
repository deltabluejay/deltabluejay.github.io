---
title: Super CTF Land
description: >-
  Gameboy
categories: [CTF Writeup]
tags: [misc, BYU CTF]
pin: false
media_subpath: '/assets/img/posts/Super_CTF_Land'
---

## Description
This challenge is from the internal BYU End-of-Semester CTF (Fall 2022).

```
Welcome to Super CTF Land, the first custom BYU CTF retro game! It looks like someone may have hacked the ROM and left behind a secret. Give it a play and see if you can find that secret!
```

## Writeup

This challenge provides you with a file named `Super_CTF_Land.gb`. A quick web search or prior knowledge of video game emulation reveals that this is the file extension for a GameBoy game. To run it, download any GameBoy emulator software- I used [SameBoy](https://sameboy.github.io/downloads/). Then, using your emulator software, load the `Super_CTF_Land.gb` file and run it.

![](start.png)

Once you are presented with the game’s title screen, hit the ENTER key or whatever key your emulator's A or START button is mapped to, and you’ll be put in the game’s first level. At the top of the screen, you’ll see the flag. 

![](flag.png)

**FLAG:** `byuctf{h@xertym}`
