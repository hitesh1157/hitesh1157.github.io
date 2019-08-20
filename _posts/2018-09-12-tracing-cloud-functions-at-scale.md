---
layout: post
title: Tracing Cloud Functions Calls at Scale
published: true
---

Consider a client connected to Firebase Realtime Database. Then consider a scenario where I have `onCreate`, `onUpdate` Cloud Function triggers on some arbitrary paths/nodes.

Now I set some data via client on an arbitrarily chosen path `p1` which invokes `onCreate` trigger on `p1`. Let's say `onCreate` on `p1` updates some key-value pairs on path `p2` which invokes `onUpdate` trigger on `p2` and updates some key-value pairs on exact same path `p1` which invokes `onUpdate` trigger on `p1` as well. After that, all executions end.

Assuming all recursive triggers are taken care of, my questions are:

1. I want to print an execution graph that according to above example prints:
```
        p1(onCreate)
        /         \
p2(onUpdate)   p2(onUpdate)
```
2. Is it even possible to do something like this with Cloud Functions or in general to track execution order?
