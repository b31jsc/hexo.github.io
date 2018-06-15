---
title: Git中的objects数据管理
date: 2018-06-14 13:49:08
tags: [Git]
description: 介绍 Git 如何通过 objects 管理数据
---
### ﻿0. 前言
学习Git的时候见到如下描述：
> Git直接记录快照，而非差异比较。  
> Git 和其他版本控制系统的主要差别在于，Git 只关心文件数据的整体是否发生变化，而大多数其他系统则只关心文件内容的具体差异。这类系统（CVS，Subversion，Perforce，Bazaar 等等）每次记录有哪些文件作了更新，以及都更新了哪些行的什么内容。如下图：
![SVN](https://raw.githubusercontent.com/b31jsc/img/master/Git%E4%B8%AD%E7%9A%84objects/Git%E4%B8%AD%E7%9A%84objects1-SVN.png)
> 
> Git 并不保存这些前后变化的差异数据。实际上，Git 更像是把变化的文件作快照后，记录在一个微型的文件系统中。每次提交更新时，它会纵览一遍所有文件的指纹信息并对文件作一快照，然后保存一个指向这次快照的索引。为提高性能，若文件没有变化，Git 不会再次保存，而只对上次保存的快照作一链接。如下图：
![GIT](https://raw.githubusercontent.com/b31jsc/img/master/Git%E4%B8%AD%E7%9A%84objects/Git%E4%B8%AD%E7%9A%84objects2-Git.png)

一直没能理解快照snapshot的真正含义，快照是否就是文件的复制？如果是复制，一个大文件只修改了一小部分也会完全复制一份么？这样是否造成了资源的浪费？如果不是复制，那么快照又是什么呢？带着这些问题做了如下实验。

### ﻿1. 新建Git仓库
`mkdir test` 新建目录 test  
`cd test/` 打开目录 test  
`git init` 初始化 Git 仓库
```
$ mkdir test
$ d test/
$ git init
已初始化空的 Git 仓库于 /home/jiang/git/test/.git/
```
完成仓库建立后，目录 test 下新建了目录 .git，目录结构如下：
```
$ tree .git/
.git/
├── branches
├── config
├── description
├── HEAD
├── hooks
│   ├── applypatch-msg.sample
│   ├── commit-msg.sample
│   ├── fsmonitor-watchman.sample
│   ├── post-update.sample
│   ├── pre-applypatch.sample
│   ├── pre-commit.sample
│   ├── prepare-commit-msg.sample
│   ├── pre-push.sample
│   ├── pre-rebase.sample
│   ├── pre-receive.sample
│   └── update.sample
├── info
│   └── exclude
├── objects
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags

9 directories, 15 files
$ 
```
其中目录 .git/objects/ 存放所有的 Git 对象

### ﻿2. 新建文件
新建文件 hello.txt，输入命令 `git add hello.txt`，查看目录 .git，发现目录 objects 下新增了一个文件
```
$ echo hello > hello.txt
$ git add hello.txt
$ tree .git/objects/
.git/objects/
├── ce
│   └── 013625030ba8dba906f756967f9e9ca394464a
├── info
└── pack

3 directories, 1 file
$ 
```
输入命令 `git commit`，再次查看，发现目录 objects 下又新增了两个文件，目录 refs 下也新增了一个文件 master
```
$ git commit -m "commit 1"
[master （根提交） c84bbf5] commit 1
 1 file changed, 1 insertion(+)
 create mode 100644 hello.txt
$ 
$ tree .git/objects/
.git/objects/
├── aa
│   └── a96ced2d9a1c8e72c56b253a0e2fe78393feb7
├── c8
│   └── 4bbf54ec2858a684d6ccabe0a92292b0e8a163
├── ce
│   └── 013625030ba8dba906f756967f9e9ca394464a
├── info
└── pack

5 directories, 3 files
$ 
$ tree .git/refs/
.git/refs/
├── heads
│   └── master
└── tags

2 directories, 1 file
$ 
```
依次查看新增的3个文件  
文件 master 中的内容是一个随机字符串，内容与目录 objects 中的一个文件名高度类似。查看该随机字符串对应的文件
```
$ cat .git/refs/heads/master
c84bbf54ec2858a684d6ccabe0a92292b0e8a163
$
```
文件 4bbf54ec2858a684d6ccabe0a92292b0e8a163 的内容为乱码
```
$ cat .git/objects/c8/4bbf54ec2858a684d6ccabe0a92292b0e8a163       
x��M
� @��=��
         EG��z�q2�
&$��o�@�>o��Ƨ�8D���g�qNd8J@v>���	�&�H�>�ldk���l���Y*Ys{����>�qc��"�u�Z]�_�(���6�%       

$
```
目录 objects 中的文件是经过压缩的，需要使用命令 `git cat-file` 查看  
> `git cat-file`命令介绍：
```
 $ git cat-file
 用法：git cat-file (-t [--allow-unknown-type] | -s
 [--allow-unknown-type] | -e | -p | <类型> | --textconv |
 --filters) [--path=<路径>] <对象>
    或：git cat-file (--batch | --batch-check)
    [--follow-symlinks] [--textconv | --filters]
 
 <类型> 可以是其中之一：blob、tree、commit、tag
     -t                    显示对象类型
     -s                    显示对象大小
     -e                    当没有错误时退出并返回零
     -p                    美观地打印对象的内容
     --textconv            对于数据对象，对其内容做文本转换
     --filters             对于数据对象，对其内容做过滤
     --path <数据对象>     对于 --textconv/--filters 使用一个特定的路径
     --allow-unknown-type  允许 -s 和 -t 对损坏的对象生效
     --buffer              缓冲 --batch 的输出
     --batch[=<格式>]      显示从标准输入提供的对象的信息和内容
     --batch-check[=<格式>]
                           显示从标准输入提供的对象的信息
     --follow-symlinks     跟随树内符号链接（和 --batch 或 --batch-check 共用）
     --batch-all-objects   使用 --batch 或 --batch-check 参数显示所有对象

$
```

输入命令 `git cat-file -p c84bbf54ec2858a684d6ccabe0a92292b0e8a163`，查看文件内容，第一行为 "tree + 随机字符串"，随机字符串与目录 objects 下的另一个文件名高度类似，第二行为 "author + 本文作者"，第三行为 "commiter + 上传人员"，最后一行是 commit message
```
$ git cat-file -p c84bbf54ec2858a684d6ccabe0a92292b0e8a163
tree aaa96ced2d9a1c8e72c56b253a0e2fe78393feb7
author b31jsc <jiangshichenbj@qq.com> 1528880332 +0800
committer b31jsc <jiangshichenbj@qq.com> 1528880332 +0800

commit 1
$
```
继续查看随机字符串对应的文件，输入命令 `git cat-file -p aaa96ced2d9a1c8e72c56b253a0e2fe78393feb7`，查看文件内容，第一行为 "数字 + blob + 随机字符串 + 文件名"，随机字符串与目录 objects 下的最后一个文件名高度类似，文件名与保存文件的名称一致
```
$ git cat-file -p aaa96ced2d9a1c8e72c56b253a0e2fe78393feb7 
100644 blob ce013625030ba8dba906f756967f9e9ca394464a	hello.txt
$
```
继续查看随机字符串对应的文件，输入命令 `git cat-file -p ce013625030ba8dba906f756967f9e9ca394464a`，查看文件内容，文件内容为 “hello”，与我们记录在文件 hello.txt 中的内容一致
```
$ git cat-file -p ce013625030ba8dba906f756967f9e9ca394464a
hello
$
```
至此，一个 master 文件，三个 objects 文件都已查看，梳理一下四个文件的内容与关系（文件名只记录前七个字符，方便查看）
* 文件 `.git/refs/heads/master` 的内容指向了记录 commit 信息的文件
* 文件 `.git/objects/c8/4bbf54e` 的内容为 commit 信息，包括 tree、author、commiter 和 commit message，其中 tree 指向了记录目录信息的文件
* 文件 `.git/objects/aa/a96ced2` 的内容为目录信息，包括 blob 和该目录下的文件名，其中 blob 指向了记录文件内容的文件
* 文件 `.git/objects/ce/0136250` 的内容为内容信息，只包含文件的具体内容，文件名称记录在 tree 中

那么带来了一些新问题：
1. **猜测** 每次提交都会新建一个 commit 文件，master 的内容会及时更新，指向最新提交的 commit 信息
2. **猜测** commit 信息只需要包含根目录的 tree 信息
3. tree 如果目录下又包含子目录，tree 中的内容该如何记录？
4. 如果 blob 只记录文件内容，假设两个文件名称和路径都不一致，但是内容相同，是否只有一个 blob ？
5. blob 修改某个文件之后，会生成新的blob，还是通过其他方式记录？

带着上述疑问，我们继续验证

### 3. 新建目录
根目录 test 下新建子目录 dir2，子目录 dir2 下新建文件 hello2.txt，输入命令 `git add dir2/hello2.txt`，查看目录 objects，发现目录下新增了文件 `.git/objects/14/be0d41c`，文件内容与 hello2.txt 相同
```
$ mkdir dir2
$ echo "hello2" > dir2/hello2.txt
$ git add dir2/hello2.txt  
$ tree .git/objects/
.git/objects/
├── 14
│   └── be0d41c639d701e0fe23e835b5fe9524b4459d
├── aa
│   └── a96ced2d9a1c8e72c56b253a0e2fe78393feb7
├── c8
│   └── 4bbf54ec2858a684d6ccabe0a92292b0e8a163
├── ce
│   └── 013625030ba8dba906f756967f9e9ca394464a
├── info
└── pack

6 directories, 4 files
$
$ git cat-file -p 14be0d41c639d701e0fe23e835b5fe9524b4459d
hello2
$ 
```
输入命令 `git commit`，继续查看目录 objects，发现目录下新增了三个文件
```
$ git commit -m "hello2 msg"
[master 16f146f] hello2 msg
 1 file changed, 1 insertion(+)
 create mode 100644 dir2/hello2.txt
$ tree .git/objects/
.git/objects/
├── 13
│   └── 6a5a4969ce4ad5b5836df2f369b22ccce3f055
├── 14
│   └── be0d41c639d701e0fe23e835b5fe9524b4459d
├── 16
│   └── f146fb4380737991f6bb6e27eca549c4d2be90
├── aa
│   └── a96ced2d9a1c8e72c56b253a0e2fe78393feb7
├── c8
│   └── 4bbf54ec2858a684d6ccabe0a92292b0e8a163
├── ce
│   └── 013625030ba8dba906f756967f9e9ca394464a
├── fe
│   └── ea7c8f0b7b6bcbc1960875e7c47798576855d2
├── info
└── pack

9 directories, 7 files
$ 
```
我们继续从文件 master 开始查看，文件 `.git/refs/heads/master` 中的内容指向了新的 commit 信息
```
$ cat .git/refs/heads/master                              
16f146fb4380737991f6bb6e27eca549c4d2be90
$ 
```
文件 `.git/objects/16/f146fb4` 记录了新的 commit 信息，仍然包含 tree、author、commiter 和 commit message 等信息，其中 tree 有更新，指向了一个新的 tree 信息，新增了一个 parent，指向上一次提交的 commit 信息
```
$ git cat-file -p 16f146fb4380737991f6bb6e27eca549c4d2be90
tree 136a5a4969ce4ad5b5836df2f369b22ccce3f055
parent c84bbf54ec2858a684d6ccabe0a92292b0e8a163
author b31jsc <jiangshichenbj@qq.com> 1528964834 +0800
committer b31jsc <jiangshichenbj@qq.com> 1528964834 +0800

hello2 msg
$ 
```
文件 `.git/objects/13/6a5a496` 记录了新的 tree 信息，内容包含一个 tree 和一个 blob，根目录下正好有一个子目录 dir2 和一个文件 hello.txt
```
$ git cat-file -p 136a5a4969ce4ad5b5836df2f369b22ccce3f055
040000 tree feea7c8f0b7b6bcbc1960875e7c47798576855d2	dir2
100644 blob ce013625030ba8dba906f756967f9e9ca394464a	hello.txt
$ 
```
文件 `.git/objects/fe/ea7c8f0` 记录了子目录 dir2 的 tree 信息，内容包含一个 blob，指向文件 hello2.txt
```
$ git cat-file -p feea7c8f0b7b6bcbc1960875e7c47798576855d2
100644 blob 14be0d41c639d701e0fe23e835b5fe9524b4459d	hello2.txt
$ 
```
指向关系如下：
```
master  
└── commit 16f146              第二次 commit
   ├── tree 136a5a4              目录 test  
   │   ├──  tree feea7c8         目录 dir2  
   │   │    └──  blob 14be0d4    文件 hello2.txt  
   │   └──  blob ce01362         文件 hello.txt  与第一版为同一个文件
   └── parent c84bbf5          第一次 commit
       └──  tree aaa96ce         目录 test  
            └──  blob ce01362    文件 hello.txt

```
上文提出的部分问题我们也就有答案了：
1. **猜测** 每次提交都会新建一个 commit 文件，master 的内容会及时更新，指向最新提交的 commit 信息  **是的**
2. **猜测** commit 信息只需要包含根目录的 tree 信息 **是的，有了根目录的 tree 信息，就可以还原成整个版本**
3. tree 如果目录下又包含子目录，tree 中的内容该如何记录？**分别记录目录和文件，目录通过 tree 记录，文件通过 blob 记录**

### 4. 修改文件
修改文件 hello.txt，输入命令 `git add`，查看目录 objects，发现目录下新增了一个文件 `.git/objects/15/b8f2a8f`，内容与文件 hello.txt 修改后的内容一致
```
$ echo "hello1" > hello.txt
$ git add hello.txt
$ tree .git/objects/
.git/objects/
├── 13
│   └── 6a5a4969ce4ad5b5836df2f369b22ccce3f055
├── 14
│   └── be0d41c639d701e0fe23e835b5fe9524b4459d
├── 15
│   └── b8f2a8ffc8a7789b65fdcf2505f23ea9e4dde0
├── 16
│   └── f146fb4380737991f6bb6e27eca549c4d2be90
├── aa
│   └── a96ced2d9a1c8e72c56b253a0e2fe78393feb7
├── c8
│   └── 4bbf54ec2858a684d6ccabe0a92292b0e8a163
├── ce
│   └── 013625030ba8dba906f756967f9e9ca394464a
├── fe
│   └── ea7c8f0b7b6bcbc1960875e7c47798576855d2
├── info
└── pack

10 directories, 8 files
$ 
$ git cat-file -p 15b8f2a8ffc8a7789b65fdcf2505f23ea9e4dde0
hello1
$
```
输入命令 `git commit`，继续查看目录 objects，目录下又新增了两个文件，一个是 commit，记录本次 commit 信息，一个是 tree，将 blob 指向修改后的 hello.txt 文件  
此处，不再赘述，指向关系如下：
```
master  
└── commit f0f28c9                 第三次 commit
    ├── tree 4deb88d                 目录 test  
    │   ├──  tree feea7c8            目录 dir2       与第二版一致
    │   │    └──  blob 14be0d4       文件 hello2.txt 与第二版一致
    │   └──  blob 15b8f2a            文件 hello.txt
    └── parent 16f146f             第二次 commit
        ├── tree 136a5a4             目录 test  
        │   ├──  tree feea7c8        目录 dir2  
        │   │    └──  blob 14be0d4   文件 hello2.txt  
        │   └──  blob ce01362        文件 hello.txt  与第一版一致
        └── parent c84bbf5         第一次 commit
            └──  tree aaa96ce        目录 test  
                 └──  blob ce01362   文件 hello.txt

```
接下来尝试将两个文件的内容修改为完全相同  
输入命令 `cat dir2/hello2.txt > hello.txt`，输入命令 `git add`，查看目录 objects，发现目录下文件数量没有变化
```
$ cat dir2/hello2.txt > hello.txt
$
```
输入命令 `git commit`，继续查看目录 objects，发现目录下新增了两个文件，一个是 commit，记录本次 commit 信息，一个是 tree，将 blob 指向 hello2.txt 文件
此处，不再赘述，指向关系如下：
```
master  
└── commit d16778c                     第四次 commit
    ├── tree ea48e71                     目录 test  
    │   ├──  tree feea7c8                目录 dir2       与第二版一致
    │   │    └──  blob 14be0d4           文件 hello2.txt 与第二版一致
    │   └──  blob 14be0d4                文件 hello.txt  与 hello2.txt 一致
    └── parent f0f28c9                 第三次 commit
        ├── tree 4deb88d                 目录 test  
        │   ├──  tree feea7c8            目录 dir2       与第二版一致
        │   │    └──  blob 14be0d4       文件 hello2.txt 与第二版一致
        │   └──  blob 15b8f2a            文件 hello.txt
        └── parent 16f146f             第二次 commit
            ├── tree 136a5a4             目录 test  
            │   ├──  tree feea7c8        目录 dir2  
            │   │    └──  blob 14be0d4   文件 hello2.txt  
            │   └──  blob ce01362        文件 hello.txt  与第一版一致
            └── parent c84bbf5         第一次 commit
                └──  tree aaa96ce        目录 test  
                     └──  blob ce01362   文件 hello.txt
```

至此，完成上文提出的最后两个问题验证：
4. 如果 blob 只记录文件内容，假设两个文件名称和路径都不一致，但是内容相同，是否只有一个 blob ？ **是的，blob 只记录内容，内容一致就是同一个**
5. 修改某个文件之后，会生成新的blob，还是通过其他方式记录？ **生成新的 blob**


### 汇总：
git 将数据保存在目录 objects 下，称为对象。
* 对象有四种类型：blob、tree、commit、tag
  * blob: 每个 blob 对应一份文件内容，只记录文件的具体内容，不包含文件名称，文件内容通过 zlib 格式压缩，当文件内容被修改时会生成一个新 blob 文件，原 blob 文件依然保留。即使是不同路径和不同名称的两个文件，只要内容相同，会使用同一个对象 blob 
  * tree: 每个 tree 对应一个目录，记录该目录下的文件名和子目录名
  * commit: 每个 commit 对应一次提交，记录本次提交message、根目录和父提交
  * tag: 待补充

* 目录 objects 中的文件命名格式：  
生成 objects 对象时，git 通过源文件计算出一个40位 SHA1 值，将 SHA1 值的前2位作为目录名、后38位作为文件名进行保存，通过 zlib 算法压缩文件内容生成对象文件。blob 只记录文件的具体内容，不包含文件名称，即使是不同路径和不同名称的两个文件，只要内容相同，会使用同一个 blob 文件  
源文件对应的 SHA1 值可通过命令 `git hash-object` 查看  
例如 hello.txt 文件
  ```
  $ git hash-object hello.txt
  ce013625030ba8dba906f756967f9e9ca394464a
  $
  $ ls .git/objects/ce/013625030ba8dba906f756967f9e9ca394464a
  .git/objects/ce/013625030ba8dba906f756967f9e9ca394464a
  $
  ```
* 目录 refs 下的文件 master 内容对应最后一次提交的对象 commit。如果通过 `git reset` 等方式进行版本回溯，只需要修改 master 内容为对应版本的对象 commit

* 回答最初的疑问  
一直没能理解快照snapshot的真正含义，快照是否就是文件的复制？如果是复制，一个大文件只修改了一小部分也会完全复制一份么？这样是否造成了资源的浪费？如果不是复制，那么快照又是什么呢？带着这些问题做了如下实验。  
**Git 将源文件的内容通过 zlib 压缩、以 SHA1 值命名，保存为对象 blob。文件内容的任何修改都会创建一个新的 blob 对象。同一个文件的当前版本和历史版本，如果内容相同，则对应同一个对象 blob；如果内容不同，则对应不同的对象 blob。对象 blob 与文件内容一一对应。通过对象 commit 记录版本信息，通过对象 tree 记录目录结构。对象 commit 和对象 tree 可以快速还原出指定版本的目录结构，这种还原方式被称为快照**
