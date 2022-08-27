# a method to manage snapshots for a block chain.

Create one commit for a block.

Store balance for a account in a file like this.

```shell
> cat balance/95150b71976fbf282260f8005fb67082573e4cc0
10
```

In the same commit, record transactions in a file like this. The record contains debit or credit, amount, relate account and relate block.

```shell
> cat transactions/95150b71976fbf282260f8005fb67082573e4cc0 
debit:10:{THE_HASH_OF_BLOCK_0}:{THE_HASH_OF_BLOCK_0}
```

When create commit, commit message include block hash and block height. After that tag the commit with these two value.

At the next commit/block. Account `95150b71976fbf282260f8005fb67082573e4cc0` send `5` to `fb996db6c0fadbea26bd50c2345c360a303ac040`. The repo will looks like this.

```shell
> cat balance/95150b71976fbf282260f8005fb67082573e4cc0 
5
> cat balance/fb996db6c0fadbea26bd50c2345c360a303ac040 
5
> cat transactions/95150b71976fbf282260f8005fb67082573e4cc0 
debit:10:{THE_HASH_OF_BLOCK_0}:{THE_HASH_OF_BLOCK_0}
credit:5:fb996db6c0fadbea26bd50c2345c360a303ac040:{THE_HASH_OF_BLOCK_1}
> cat transactions/fb996db6c0fadbea26bd50c2345c360a303ac040 
debit:5:95150b71976fbf282260f8005fb67082573e4cc0:{THE_HASH_OF_BLOCK_1}
```

And the git log will looks like this.

```log
> git log
commit f34025ec49d1b1dbef4e3026d1dcff4bb2088250 (HEAD -> master, tag: THE_HASH_OF_BLOCK_1, tag: 1)
Author: snap <snapshot>
Date:   Sat Aug 27 23:31:53 2022 +1200

    THE_HASH_OF_BLOCK_1
    1

commit 28d49ccd9ecbdb8040e38d027a0f2369d79711dd (tag: THE_HASH_OF_BLOCK_0, tag: 0)
Author: snap <snapshot>
Date:   Sat Aug 27 23:29:57 2022 +1200

    THE_HASH_OF_BLOCK_0
    0
```

Then, if we want to know the balance of `95150b71976fbf282260f8005fb67082573e4cc0` at block `0`/`THE_HASH_OF_BLOCK_0`.
- `git show THE_HASH_OF_BLOCK_0:balance/95150b71976fbf282260f8005fb67082573e4cc0`
- `git show 0:balance/95150b71976fbf282260f8005fb67082573e4cc0`
