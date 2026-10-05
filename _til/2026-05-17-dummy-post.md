---
layout: til
title: "Replace text across files with rg + sd"
date: 2026-05-17
lang: bash
tags: [shell, cli, ripgrep, sd]
published: false
---

`rg` finds matches fast; `sd` (a modern `sed` replacement with sane regex syntax) handles the substitution. Together they're faster and less error-prone than `grep -r` + `sed -i`, especially across large trees.

```bash
# replace OldName with NewName across all .go files
rg -l 'OldName' --glob '*.go' | xargs sd 'OldName' 'NewName'
```
