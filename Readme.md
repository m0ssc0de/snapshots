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

## some thinking

Git is a great tools to manage snapshot. All balances on a block could be regard as a kind of snapshot. so we store the balnce, relate transactions or other things if needed in a snapshot. We can get a snapshot extremely fast(need confirm on a large data set), benefited by the nice algorithm of git.

Create commits in parallel. If a high I/O method calling be used in getting block details, we can create multi-branchs for every parallel callback. Then base on the git hooks to merge/rebase all branchs without any gaps. THe git hooks also could make different repo for reading and writing.

Using git repo as data repo has another benefit, there are a lot lib to interact with it. Even load it into memory to speed up.

### about performance

objects database
objects in pack file

performance issues, solution: separate into 2 letters folder like https://github.com/rust-lang/crates.io-index
30长度 实际40
同一级，一次 O(n/2) n是 account 数量 204m?
每个字符一级，30次 O((24+10)/2) : `30 * ( (24+10)/2 )` O(510) O(1), O(680)
每两个词组一级,15次 O((24+10)*(24+10)/2) : `15 * ( (34*34)/2 )` O(8670) O(1)

```shell
moss@potato /tmp % cat cat.sh 
#!/bin/bash
for i in {1..680}
do
   cat /tmp/a > /dev/null
done
moss@potato /tmp % time bash cat.sh
bash cat.sh  0.15s user 0.38s system 78% cpu 0.673 total
```

ext2
Maximum number of files: 10^18

other unknown limit of git or file system

30

names database

reftables make name resolving performance like hash lookup
https://github.com/google/reftable



Scan (read 866k refs), by reference name lookup (single ref from 866k refs), and by SHA-1 lookup (refs with that SHA-1, from 866k refs):
|format 	|cache  |scan 	|by name 	|by SHA-1|
|-|-|-|-|-|
|packed-refs |cold |402 ms |409,660.1 usec |412,535.8 usec|
|packed-refs |hot ||6,844.6 usec |20,110.1 usec|
|reftable |cold |112 ms |33.9 usec |323.2 usec|
|reftable |hot ||20.2 usec |320.8 usec|

multi-pack-index

customize commit hash with block hash. x

compare with just put in different folders named by blockheight