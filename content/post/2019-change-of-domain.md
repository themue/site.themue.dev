+++
date = "2019-03-02T20:00:00+01:00"
draft = false
title = "Change of top level domain to dev"
tags = ["blogging", "tideland"]
categories = ["blogging", "tideland"]
+++

On Feb 28th the top level domain **.dev** has been available for all. So for me as
a developer it's the best possible TLD to get. One day later I reserved two different ones:

1. **themue.dev** as a replacement for my former personal domain **themue.name** and
2. **tideland.dev** as a replacement of all my different Tideland domains into one.

The personal domain is now changed, the Tideland domains will follow these days. Then
I also have to change the new Go mono repository I'm migrating my packages to. They'll
then be importable like

```
import (
    "context"
    
    "tideland.dev/go/together/loop"
    "tideland.dev/go/together/notifier"
)
```

I hope you like these changes.