+++
title="Python cannot resolve local computer name in non-english"
date=2022-02-22

[taxonomies]
categories=["Programming"]
tags=["python", "host", "dns"]
+++

Encountered an interesting issue: python cannot resolve dns name if it's not english.

```
gethostbyaddrname
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xca in position 52: invalid continuation byte
```