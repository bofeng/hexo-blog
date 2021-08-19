---
layout: post
title: Install pinyin input method in ubuntu
date: 2021-08-18 23:36:55
tags: ["ubuntu", "pinyin"]
---

## Problem

You want to input Chinese pinyin input method in Ubuntu.

## Solution


### Step 1

```bash
sudo apt install fcitx fcitx-sunpinyin
```

### Step 2

Go to system settings, then "Region & Language", click "Manage Installed Languages", install Chinese language and change "Keyboard input method system" to "fcitx".

Then in the "Input sources", click the "+" button, select "Chinese" then "Chinese Pinyin".

