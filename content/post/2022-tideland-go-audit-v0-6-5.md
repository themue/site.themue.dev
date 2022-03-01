---
title: 'Tideland Go Audit v0.6.5'
date: '2022-02-28T22:00:00+01:00'
draft: false
categories: ['tideland']
tags: ['golang', 'tideland', 'library', 'release', 'testing']
---

There are times when you find a bug in your software. Today it had been in my testing library **TIdeland Go Audit**. Here the assertion `ErrorContains()` reacted with a panic in case of a `nil` error. So I fixed it like I already had done it earlier in `ErrorMatch()`. Interestingly I found in testing that I didn't verified it there. So this test is now also changed.
