---
title: 'InIt CTF: Leaky History'
author: AmcaQt
categories: [InIt CTF]
tags: [Git, Forensics]
render_with_liquid: false
media_subpath: /images/init-ctf-leaky-history/
---

## Description

A developer on the Tanvoro deploy team realised they'd committed a secrets file to the tanvoro-deploy repo. In a panic they "cleaned it up" — dropped the commit, scrubbed their tracks. The repo looks spotless now: `git log` shows nothing out of place.

We recovered a copy of the repository. The secret isn't in the history anymore… but git rarely throws anything away. Get it back.

Handout: a copy of the `tanvoro-deploy` repo.

## Question & Flag

### First look

Dropped into the repo and ran the obvious thing first.

```
$ git log
commit ddc4e9b92ac2075c22f9e6244a561dde411b269b (HEAD -> main)
Author: Dana Okoro <dana@tanvoro.example>
Date:   Mon Sep 21 11:05:33 2026 +0000

    docs: note secret handling

commit da390f04d219c65c0a515c0111f1ed58da83d5a5
Author: Dana Okoro <dana@tanvoro.example>
Date:   Sun Sep 20 09:40:12 2026 +0000

    deploy: add .env.example, gitignore secrets.env

commit 13e088d11afcad79c3aa42bdffa7397c8673a645
Author: Dana Okoro <dana@tanvoro.example>
Date:   Sat Sep 19 14:22:40 2026 +0000

    deploy: add deploy.sh

commit 5d726f152cbfe5057ed5da1732141d32546b1716
Author: Dana Okoro <dana@tanvoro.example>
Date:   Fri Sep 18 09:03:11 2026 +0000

    init: flask skeleton + readme
```

Four commits, nothing suspicious in any of the messages. Ran `git show` against each of these four hashes individually, nothing relevant turned up, no secrets, no weird diffs. Exactly what the description promised: the history looks clean.

### The commit that isn't in the history

`git log` only walks commits reachable from a ref (branches, tags, HEAD). If a commit gets dropped from a branch, say with `git reset --hard` to an earlier commit, or an interactive rebase that drops it, the commit object itself doesn't disappear immediately. It just becomes unreachable, nothing points at it anymore, so `git log` will never show it. But the object is still sitting in `.git/objects` until git actually garbage-collects it, and that doesn't happen automatically very often.

Checked `git show-ref` first to see what refs actually exist:

```
$ git show-ref
ddc4e9b92ac2075c22f9e6244a561dde411b269b refs/heads/main
```

Just the one ref, pointing at the same HEAD commit from `git log`. Confirms there's nothing else being tracked, no stray branches or tags hiding the dropped commit.

So the real tool for this is `git fsck`, which walks every object in `.git/objects` regardless of whether anything points to it, and flags anything unreachable:

```
$ git fsck --full
Checking object directories: 100% (1/1), done.
Checking object directories: 100% (256/256), done.
dangling commit 6dfa863c4a9a1700c6f09a608761f82ef9c241a2
```

There it is. A dangling commit sitting in the object store that no branch or tag points to, exactly the kind of leftover a `reset --hard` or rewritten history leaves behind.

### Recovering it

`git show` works on any object hash you give it, reachable or not, since the data's still physically there:

```
$ git show 6dfa863c4a9a1700c6f09a608761f82ef9c241a2
commit 6dfa863c4a9a1700c6f09a608761f82ef9c241a2
Author: Dana Okoro <dana@tanvoro.example>
Date:   Sun Sep 20 02:11:57 2026 +0000

    ci: wire up deploy secrets

diff --git a/deploy/secrets.env b/deploy/secrets.env
new file mode 100644
index 0000000..4d080a7
--- /dev/null
+++ b/deploy/secrets.env
@@ -0,0 +1,5 @@
+# DO NOT COMMIT -- loaded by CI
+AWS_ACCESS_KEY_ID=AKIAZZ7FAKE4EXAMPLE1
+AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
+DEPLOY_TOKEN=InIt{d4ngl1ng_c0mm1t_l34k}
+DB_PASSWORD=pr0d-db-9f3a21
```

The exact commit that got "cleaned up." `deploy/secrets.env`, added and then presumably scrubbed from the branch, but the commit object was never actually deleted, just unlinked from any ref.

```
InIt{d4ngl1ng_c0mm1t_l34k}
```

## Conclusion

`git reset`, rebase, and friends only move refs around, they don't touch the underlying object store. The data behind a "deleted" commit sticks around until something actually runs garbage collection (`git gc`, or enough time/commits passing for it to trigger automatically) and prunes anything unreachable. Until that happens, `git fsck --full` will dig it straight back out. Moral of the story: if you accidentally commit a secret, rotating the secret is the only real fix, dropping the commit is not deleting it.

# Happy Hacking
