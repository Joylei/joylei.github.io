+++
title="got atom-shell 404 issue when updating atom"
date=2016-03-31

[taxonomies]
categories=["Programming"]
tags=["atom"]
+++
Recent days I tried several times to update Atom packages from both Atom and apm update command, but failed finally.

So I decided to look what happened by running command:
```sh
apm update --verbose
```

It seems it failed to download atom-shell.

Googled with 'atom-shell 404' and found this link https://github.com/atom/atom/issues/11080.

The solution mentioned did the trick.
