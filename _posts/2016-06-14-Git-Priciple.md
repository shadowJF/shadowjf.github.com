---
layout: post_layout
title: Git原理详解
time: 2016年06月14日 星期二
location: 北京
pulished: true
tag: git
excerpt_separator: "``"
---


虽然一直在用Git，但是对Git的原理一直都不太理解，最近看了官网的原理讲解，觉得大有收获，下面贴出官网关于Git原理的原文。

[官方介绍地址](https://git-scm.com/book/zh/v2)

---

不过在贴出原文之前，我先谈谈自己看完之后的理解：

Git中所有对象都以key value形式保存在文件系统中，key是一个对存储内容+header进行散列后的值

Git保存了文件的每一个版本，也就是每一次commit中，对所有更新的文件，Git都会完整保存其新的副本，而不是保存和上个版本的差异。

每一次提交也是一个提交对象，也同样保存了下来，一个提交对象会指向一个树对象，而树对象则维护着你的项目的全部文件对象。树对象就像是目录。这样通过一个提交对象，就能找到树对象，通过树对象找到所有文件在该次提交时的副本对象，从而获得某次提交的版本。

不同commit下的树对象是不同的，但是树对象下的文件对象有可能相同，也就是对那些在两次commit下没有改变过的文件，那么他们的对象在两个树对象下是相同的，但是那些更新过的文件，在这两个树对象下他们的存储对象也是不一样的，分别对应着自己的版本的存储对象。

每次commit之所以要指定父commit，是因为有了父commit我们就能知道这次commit和上次commit之间的差异。通过commit对象下的树对象维护的所有文件对象，来判断有哪些文件是新增的，有哪些是删除的。

---

``

### **10.1 Git 内部原理 - 底层命令和高层命令** ###

无论是从之前的章节直接跳到本章，还是读完了其余章节一直到这——你都将在本章见识到 Git 的内部工作原理和实现方式。 我们发现学习这部分内容对于理解 Git 的用途和强大至关重要。不过也有人认为这些内容对于初学者而言可能难以理解且过于复杂。 因此我们把这部分内容放在最后一章，在学习过程中可以先阅读这部分，也可以晚点阅读这部分，这取决于你自己。

无论如何，既然已经读到了这里，就让我们开始吧。 首先要弄明白一点，从根本上来讲 Git 是一个内容寻址（content-addressable）文件系统，并在此之上提供了一个版本控制系统的用户界面。 马上你就会学到这意味着什么。

早期的 Git（主要是 1.5 之前的版本）的用户界面要比现在复杂的多，因为它更侧重于作为一个文件系统，而不是一个打磨过的版本控制系统。 不时会有一些陈词滥调抱怨早期那个晦涩复杂的 Git 用户界面；不过最近几年来，它已经被改进到不输于任何其他版本控制系统地清晰易用了。

内容寻址文件系统层是一套相当酷的东西，所以在本章我们会先讲解这部分内容。随后我们会学习传输机制和版本库管理任务——你迟早会和它们打交道。

##### **底层命令和高层命令** #####

本书旨在讨论如何通过 checkout、branch、remote 等大约 30 个诸如此类动词形式的命令来玩转 Git。 然而，由于 Git 最初是一套面向版本控制系统的工具集，而不是一个完整的、用户友好的版本控制系统，所以它还包含了一部分用于完成底层工作的命令。 这些命令被设计成能以 UNIX 命令行的风格连接在一起，抑或藉由脚本调用，来完成工作。 这部分命令一般被称作“底层（plumbing）”命令，而那些更友好的命令则被称作“高层（porcelain）”命令。

本书前九章专注于探讨高层命令。 然而在本章，我们将主要面对底层命令。 因为，底层命令得以让你窥探 Git 内部的工作机制，也有助于说明 Git 是如何完成工作的，以及它为何如此运作。 多数底层命令并不面向最终用户：它们更适合作为新命令和自定义脚本的组成部分。

当在一个新目录或已有目录执行 git init 时，Git 会创建一个 .git 目录。 这个目录包含了几乎所有 Git 存储和操作的对象。 如若想备份或复制一个版本库，只需把这个目录拷贝至另一处即可。 本章探讨的所有内容，均位于这个目录内。 该目录的结构如下所示：
    
    $ ls -F1
    HEAD
    config*
    description
    hooks/
    info/
    objects/
    refs/

该目录下可能还会包含其他文件，不过对于一个全新的 git init 版本库，这将是你看到的默认结构。 description 文件仅供 GitWeb 程序使用，我们无需关心。 config 文件包含项目特有的配置选项。 info 目录包含一个全局性排除（global exclude）文件，用以放置那些不希望被记录在 .gitignore 文件中的忽略模式（ignored patterns）。 hooks 目录包含客户端或服务端的钩子脚本（hook scripts），在 Git 钩子 中这部分话题已被详细探讨过。

剩下的四个条目很重要：HEAD 文件、（尚待创建的）index 文件，和 objects 目录、refs 目录。 这些条目是 Git 的核心组成部分。 objects 目录存储所有数据内容；refs 目录存储指向数据（分支）的提交对象的指针；HEAD 文件指示目前被检出的分支；index 文件保存暂存区信息。 我们将详细地逐一检视这四部分，以期理解 Git 是如何运转的。

### **10.2 Git 内部原理 - Git 对象** ###

##### **Git 对象**　#####

Git 是一个内容寻址文件系统。 看起来很酷， 但这是什么意思呢？ 这意味着，Git 的核心部分是一个简单的键值对数据库（key-value data store）。 你可以向该数据库插入任意类型的内容，它会返回一个键值，通过该键值可以在任意时刻再次检索（retrieve）该内容。 可以通过底层命令 hash-object 来演示上述效果——该命令可将任意数据保存于 .git 目录，并返回相应的键值。 首先，我们需要初始化一个新的 Git 版本库，并确认 objects 目录为空：

    $ git init test
    Initialized empty Git repository in /tmp/test/.git/
    $ cd test
    $ find .git/objects
    .git/objects
    .git/objects/info
    .git/objects/pack
    $ find .git/objects -type f

可以看到 Git 对 objects 目录进行了初始化，并创建了 pack 和 info 子目录，但均为空。 接着，往 Git 数据库存入一些文本：

    $ echo 'test content' | git hash-object -w --stdin
    d670460b4b4aece5915caf5c68d12f560a9fe3e4

-w 选项指示 hash-object 命令存储数据对象；若不指定此选项，则该命令仅返回对应的键值。 --stdin 选项则指示该命令从标准输入读取内容；若不指定此选项，则须在命令尾部给出待存储文件的路径。 该命令输出一个长度为 40 个字符的校验和。 这是一个 SHA-1 哈希值——一个将待存储的数据外加一个头部信息（header）一起做 SHA-1 校验运算而得的校验和。后文会简要讨论该头部信息。 现在我们可以查看 Git 是如何存储数据的：
    
    $ find .git/objects -type f
    .git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4

可以在 objects 目录下看到一个文件。 这就是开始时 Git 存储内容的方式——一个文件对应一条内容，以该内容加上特定头部信息一起的 SHA-1 校验和为文件命名。 校验和的前两个字符用于命名子目录，余下的 38 个字符则用作文件名。

可以通过 cat-file 命令从 Git 那里取回数据。 这个命令简直就是一把剖析 Git 对象的瑞士军刀。 为 cat-file 指定 -p 选项可指示该命令自动判断内容的类型，并为我们显示格式友好的内容：

    $ git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
    test content

至此，你已经掌握了如何向 Git 中存入内容，以及如何将它们取出。 我们同样可以将这些操作应用于文件中的内容。 例如，可以对一个文件进行简单的版本控制。 首先，创建一个新文件并将其内容存入数据库：

    $ echo 'version 1' > test.txt
    $ git hash-object -w test.txt
    83baae61804e65cc73a7201a7252750c76066a30

接着，向文件里写入新内容，并再次将其存入数据库：

    $ echo 'version 2' > test.txt
    $ git hash-object -w test.txt
    1f7a7a472abf3dd9643fd615f6da379c4acb3e3a

数据库记录下了该文件的两个不同版本，当然之前我们存入的第一条内容也还在：

    $ find .git/objects -type f
    .git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a
    .git/objects/83/baae61804e65cc73a7201a7252750c76066a30
    .git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4

现在可以把文件内容恢复到第一个版本：

    $ git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30 > test.txt
    $ cat test.txt
    version 1

或者第二个版本：

    $ git cat-file -p 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a > test.txt
    $ cat test.txt
    version 2

然而，记住文件的每一个版本所对应的 SHA-1 值并不现实；另一个问题是，在这个（简单的版本控制）系统中，文件名并没有被保存——我们仅保存了文件的内容。 上述类型的对象我们称之为数据对象（blob object）。 利用 cat-file -t 命令，可以让 Git 告诉我们其内部存储的任何对象类型，只要给定该对象的 SHA-1 值：

    $ git cat-file -t 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
    blob

##### **树对象** #####

接下来要探讨的对象类型是树对象（tree object），它能解决文件名保存的问题，也允许我们将多个文件组织到一起。 Git 以一种类似于 UNIX 文件系统的方式存储内容，但作了些许简化。 所有内容均以树对象和数据对象的形式存储，其中树对象对应了 UNIX 中的目录项，数据对象则大致上对应了 inodes 或文件内容。 一个树对象包含了一条或多条树对象记录（tree entry），每条记录含有一个指向数据对象或者子树对象的 SHA-1 指针，以及相应的模式、类型、文件名信息。 例如，某项目当前对应的最新树对象可能是这样的：

$ git cat-file -p master^{tree}
100644 blob a906cb2a4a904a152e80877d4088654daad0c859  README
100644 blob 8f94139338f9404f26296befa88755fc2598c289  Rakefile
040000 tree 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0  lib

master^{tree} 语法表示 master 分支上最新的提交所指向的树对象。 请注意，lib 子目录（所对应的那条树对象记录）并不是一个数据对象，而是一个指针，其指向的是另一个树对象：

    $ git cat-file -p 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0
    100644 blob 47c6340d6459e05787f644c2447d2595f5d3a54b  simplegit.rb

从概念上讲，Git 内部存储的数据有点像这样：

![](https://git-scm.com/book/en/v2/book/10-git-internals/images/data-model-1.png)

你可以轻松创建自己的树对象。 通常，Git 根据某一时刻暂存区（即 index 区域，下同）所表示的状态创建并记录一个对应的树对象，如此重复便可依次记录（某个时间段内）一系列的树对象。 因此，为创建一个树对象，首先需要通过暂存一些文件来创建一个暂存区。 可以通过底层命令 update-index 为一个单独文件——我们的 test.txt 文件的首个版本——创建一个暂存区。 利用该命令，可以把 test.txt 文件的首个版本人为地加入一个新的暂存区。 必须为上述命令指定 --add 选项，因为此前该文件并不在暂存区中（我们甚至都还没来得及创建一个暂存区呢）；同样必需的还有 --cacheinfo 选项，因为将要添加的文件位于 Git 数据库中，而不是位于当前目录下。 同时，需要指定文件模式、SHA-1 与文件名：

    $ git update-index --add --cacheinfo 100644 \
      83baae61804e65cc73a7201a7252750c76066a30 test.txt

本例中，我们指定的文件模式为 100644，表明这是一个普通文件。 其他选择包括：100755，表示一个可执行文件；120000，表示一个符号链接。 这里的文件模式参考了常见的 UNIX 文件模式，但远没那么灵活——上述三种模式即是 Git 文件（即数据对象）的所有合法模式（当然，还有其他一些模式，但用于目录项和子模块）。

现在，可以通过 write-tree 命令将暂存区内容写入一个树对象。 此处无需指定 -w 选项——如果某个树对象此前并不存在的话，当调用 write-tree 命令时，它会根据当前暂存区状态自动创建一个新的树对象：

    $ git write-tree
    d8329fc1cc938780ffdd9f94e0d364e0ea74f579
    $ git cat-file -p d8329fc1cc938780ffdd9f94e0d364e0ea74f579
    100644 blob 83baae61804e65cc73a7201a7252750c76066a30  test.txt

不妨验证一下它确实是一个树对象：

    $ git cat-file -t d8329fc1cc938780ffdd9f94e0d364e0ea74f579
    tree

接着我们来创建一个新的树对象，它包括 test.txt 文件的第二个版本，以及一个新的文件：

    $ echo 'new file' > new.txt
    $ git update-index test.txt
    $ git update-index --add new.txt

暂存区现在包含了 test.txt 文件的新版本，和一个新文件：new.txt。 记录下这个目录树（将当前暂存区的状态记录为一个树对象），然后观察它的结构：

    $ git write-tree
    0155eb4229851634a0f03eb265b69f5a2d56f341
    $ git cat-file -p 0155eb4229851634a0f03eb265b69f5a2d56f341
    100644 blob fa49b077972391ad58037050f2a75f74e3671e92  new.txt
    100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a  test.txt

我们注意到，新的树对象包含两条文件记录，同时 test.txt 的 SHA-1 值（1f7a7a）是先前值的“第二版”。 只是为了好玩：你可以将第一个树对象加入第二个树对象，使其成为新的树对象的一个子目录。 通过调用 read-tree 命令，可以把树对象读入暂存区。 本例中，可以通过对 read-tree 指定 --prefix 选项，将一个已有的树对象作为子树读入暂存区：

    $ git read-tree --prefix=bak d8329fc1cc938780ffdd9f94e0d364e0ea74f579
    $ git write-tree
    3c4e9cd789d88d8d89c1073707c3585e41b0e614
    $ git cat-file -p 3c4e9cd789d88d8d89c1073707c3585e41b0e614
    040000 tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579  bak
    100644 blob fa49b077972391ad58037050f2a75f74e3671e92  new.txt
    100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a  test.txt

如果基于这个新的树对象创建一个工作目录，你会发现工作目录的根目录包含两个文件以及一个名为 bak 的子目录，该子目录包含 test.txt 文件的第一个版本。 可以认为 Git 内部存储着的用于表示上述结构的数据是这样的：

![](https://git-scm.com/book/en/v2/book/10-git-internals/images/data-model-2.png)

##### **提交对象** #####

现在有三个树对象，分别代表了我们想要跟踪的不同项目快照。然而问题依旧：若想重用这些快照，你必须记住所有三个 SHA-1 哈希值。 并且，你也完全不知道是谁保存了这些快照，在什么时刻保存的，以及为什么保存这些快照。 而以上这些，正是提交对象（commit object）能为你保存的基本信息。

可以通过调用 commit-tree 命令创建一个提交对象，为此需要指定一个树对象的 SHA-1 值，以及该提交的父提交对象（如果有的话）。 我们从之前创建的第一个树对象开始：

    $ echo 'first commit' | git commit-tree d8329f
    fdf4fc3344e67ab068f836878b6c4951e3b15f3d

现在可以通过 cat-file 命令查看这个新提交对象：

    $ git cat-file -p fdf4fc3
    tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579
    author Scott Chacon <schacon@gmail.com> 1243040974 -0700
    committer Scott Chacon <schacon@gmail.com> 1243040974 -0700
    
    first commit

提交对象的格式很简单：它先指定一个顶层树对象，代表当前项目快照；然后是作者/提交者信息（依据你的 user.name 和 user.email 配置来设定，外加一个时间戳）；留空一行，最后是提交注释。

接着，我们将创建另两个提交对象，它们分别引用各自的上一个提交（作为其父提交对象）：

    $ echo 'second commit' | git commit-tree 0155eb -p fdf4fc3
    cac0cab538b970a37ea1e769cbbde608743bc96d
    $ echo 'third commit'  | git commit-tree 3c4e9c -p cac0cab
    1a410efbd13591db07496601ebc7a059dd55cfe9

这三个提交对象分别指向之前创建的三个树对象快照中的一个。 现在，如果对最后一个提交的 SHA-1 值运行 git log 命令，会出乎意料的发现，你已有一个货真价实的、可由 git log 查看的 Git 提交历史了：

    $ git log --stat 1a410e
    commit 1a410efbd13591db07496601ebc7a059dd55cfe9
    Author: Scott Chacon <schacon@gmail.com>
    Date:   Fri May 22 18:15:24 2009 -0700
    
    	third commit
    
     bak/test.txt | 1 +
     1 file changed, 1 insertion(+)
    
    commit cac0cab538b970a37ea1e769cbbde608743bc96d
    Author: Scott Chacon <schacon@gmail.com>
    Date:   Fri May 22 18:14:29 2009 -0700
    
    	second commit
    
     new.txt  | 1 +
     test.txt | 2 +-
     2 files changed, 2 insertions(+), 1 deletion(-)
    
    commit fdf4fc3344e67ab068f836878b6c4951e3b15f3d
    Author: Scott Chacon <schacon@gmail.com>
    Date:   Fri May 22 18:09:34 2009 -0700
    
    	first commit
    
     test.txt | 1 +
     1 file changed, 1 insertion(+)

太神奇了： 就在刚才，你没有借助任何上层命令，仅凭几个底层操作便完成了一个 Git 提交历史的创建。 这就是每次我们运行 git add 和 git commit 命令时， Git 所做的实质工作——将被改写的文件保存为数据对象，更新暂存区，记录树对象，最后创建一个指明了顶层树对象和父提交的提交对象。 这三种主要的 Git 对象——数据对象、树对象、提交对象——最初均以单独文件的形式保存在 .git/objects 目录下。 下面列出了目前示例目录内的所有对象，辅以各自所保存内容的注释：

    $ find .git/objects -type f
    .git/objects/01/55eb4229851634a0f03eb265b69f5a2d56f341 # tree 2
    .git/objects/1a/410efbd13591db07496601ebc7a059dd55cfe9 # commit 3
    .git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a # test.txt v2
    .git/objects/3c/4e9cd789d88d8d89c1073707c3585e41b0e614 # tree 3
    .git/objects/83/baae61804e65cc73a7201a7252750c76066a30 # test.txt v1
    .git/objects/ca/c0cab538b970a37ea1e769cbbde608743bc96d # commit 2
    .git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4 # 'test content'
    .git/objects/d8/329fc1cc938780ffdd9f94e0d364e0ea74f579 # tree 1
    .git/objects/fa/49b077972391ad58037050f2a75f74e3671e92 # new.txt
    .git/objects/fd/f4fc3344e67ab068f836878b6c4951e3b15f3d # commit 1

如果跟踪所有的内部指针，将得到一个类似下面的对象关系图：

![](https://git-scm.com/book/en/v2/book/10-git-internals/images/data-model-3.png)

##### **对象存储** #####

前文曾提及，在存储内容时，会有个头部信息一并被保存。 让我们略花些时间来看看 Git 是如何存储其对象的。 通过在 Ruby 脚本语言中交互式地演示，你将看到一个数据对象——本例中是字符串“what is up, doc?”——是如何被存储的。

可以通过 irb 命令启动 Ruby 的交互模式：

    $ irb
    >> content = "what is up, doc?"
    => "what is up, doc?"

Git 以对象类型作为开头来构造一个头部信息，本例中是一个“blob”字符串。 接着 Git 会添加一个空格，随后是数据内容的长度，最后是一个空字节（null byte）：
    
    >> header = "blob #{content.length}\0"
    => "blob 16\u0000"

Git 会将上述头部信息和原始数据拼接起来，并计算出这条新内容的 SHA-1 校验和。 在 Ruby 中可以这样计算 SHA-1 值——先通过 require 命令导入 SHA-1 digest 库，然后对目标字符串调用 Digest::SHA1.hexdigest()：

    >> store = header + content
    => "blob 16\u0000what is up, doc?"
    >> require 'digest/sha1'
    => true
    >> sha1 = Digest::SHA1.hexdigest(store)
    => "bd9dbf5aae1a3862dd1526723246b20206e5fc37"

Git 会通过 zlib 压缩这条新内容。在 Ruby 中可以借助 zlib 库做到这一点。 先导入相应的库，然后对目标内容调用 Zlib::Deflate.deflate()：

    >> require 'zlib'
    => true
    >> zlib_content = Zlib::Deflate.deflate(store)
    => "x\x9CK\xCA\xC9OR04c(\xCFH,Q\xC8,V(-\xD0QH\xC9O\xB6\a\x00_\x1C\a\x9D"

最后，需要将这条经由 zlib 压缩的内容写入磁盘上的某个对象。 要先确定待写入对象的路径（SHA-1 值的前两个字符作为子目录名称，后 38 个字符则作为子目录内文件的名称）。 如果该子目录不存在，可以通过 Ruby 中的 FileUtils.mkdir_p() 函数来创建它。 接着，通过 File.open() 打开这个文件。最后，对上一步中得到的文件句柄调用 write() 函数，以向目标文件写入之前那条 zlib 压缩过的内容：

    >> path = '.git/objects/' + sha1[0,2] + '/' + sha1[2,38]
    => ".git/objects/bd/9dbf5aae1a3862dd1526723246b20206e5fc37"
    >> require 'fileutils'
    => true
    >> FileUtils.mkdir_p(File.dirname(path))
    => ".git/objects/bd"
    >> File.open(path, 'w') { |f| f.write zlib_content }
    => 32

就是这样——你已创建了一个有效的 Git 数据对象。 所有的 Git 对象均以这种方式存储，区别仅在于类型标识——另两种对象类型的头部信息以字符串“commit”或“tree”开头，而不是“blob”。 另外，虽然数据对象的内容几乎可以是任何东西，但提交对象和树对象的内容却有各自固定的格式。

### **10.3 Git 内部原理 - Git 引用** ###

##### **Git 引用** #####

我们可以借助类似于 git log 1a410e 这样的命令来浏览完整的提交历史，但为了能遍历那段历史从而找到所有相关对象，你仍须记住 1a410e 是最后一个提交。 我们需要一个文件来保存 SHA-1 值，并给文件起一个简单的名字，然后用这个名字指针来替代原始的 SHA-1 值。

在 Git 里，这样的文件被称为“引用（references，或缩写为 refs）”；你可以在 .git/refs 目录下找到这类含有 SHA-1 值的文件。 在目前的项目中，这个目录没有包含任何文件，但它包含了一个简单的目录结构：
    
    $ find .git/refs
    .git/refs
    .git/refs/heads
    .git/refs/tags
    $ find .git/refs -type f

若要创建一个新引用来帮助记忆最新提交所在的位置，从技术上讲我们只需简单地做如下操作：

    $ echo "1a410efbd13591db07496601ebc7a059dd55cfe9" > .git/refs/heads/master

现在，你就可以在 Git 命令中使用这个刚创建的新引用来代替 SHA-1 值了：

    $ git log --pretty=oneline  master
    1a410efbd13591db07496601ebc7a059dd55cfe9 third commit
    cac0cab538b970a37ea1e769cbbde608743bc96d second commit
    fdf4fc3344e67ab068f836878b6c4951e3b15f3d first commit

我们不提倡直接编辑引用文件。 如果想更新某个引用，Git 提供了一个更加安全的命令 update-ref 来完成此事：

    $ git update-ref refs/heads/master 1a410efbd13591db07496601ebc7a059dd55cfe9

这基本就是 Git 分支的本质：一个指向某一系列提交之首的指针或引用。 若想在第二个提交上创建一个分支，可以这么做：

    $ git update-ref refs/heads/test cac0ca

这个分支将只包含从第二个提交开始往前追溯的记录：

    $ git log --pretty=oneline test
    cac0cab538b970a37ea1e769cbbde608743bc96d second commit
    fdf4fc3344e67ab068f836878b6c4951e3b15f3d first commit

至此，我们的 Git 数据库从概念上看起来像这样：

![](https://git-scm.com/book/en/v2/book/10-git-internals/images/data-model-4.png)

当运行类似于 git branch (branchname) 这样的命令时，Git 实际上会运行 update-ref 命令，取得当前所在分支最新提交对应的 SHA-1 值，并将其加入你想要创建的任何新引用中。

##### **HEAD 引用** #####

现在的问题是，当你执行 git branch (branchname) 时，Git 如何知道最新提交的 SHA-1 值呢？ 答案是 HEAD 文件。

HEAD 文件是一个符号引用（symbolic reference），指向目前所在的分支。 所谓符号引用，意味着它并不像普通引用那样包含一个 SHA-1 值——它是一个指向其他引用的指针。 如果查看 HEAD 文件的内容，一般而言我们看到的类似这样：

    $ cat .git/HEAD
    ref: refs/heads/master

如果执行 git checkout test，Git 会像这样更新 HEAD 文件：

    $ cat .git/HEAD
    ref: refs/heads/test

当我们执行 git commit 时，该命令会创建一个提交对象，并用 HEAD 文件中那个引用所指向的 SHA-1 值设置其父提交字段。

你也可以手动编辑该文件，然而同样存在一个更安全的命令来完成此事：symbolic-ref。 可以借助此命令来查看 HEAD 引用对应的值：

    $ git symbolic-ref HEAD
    refs/heads/master

同样可以设置 HEAD 引用的值：

    $ git symbolic-ref HEAD refs/heads/test
    $ cat .git/HEAD
    ref: refs/heads/test

不能把符号引用设置为一个不符合引用格式的值：

    $ git symbolic-ref HEAD test
    fatal: Refusing to point HEAD outside of refs/

##### **标签引用** #####

前文我们刚讨论过 Git 的三种主要对象类型，事实上还有第四种。 标签对象（tag object）非常类似于一个提交对象——它包含一个标签创建者信息、一个日期、一段注释信息，以及一个指针。 主要的区别在于，标签对象通常指向一个提交对象，而不是一个树对象。 它像是一个永不移动的分支引用——永远指向同一个提交对象，只不过给这个提交对象加上一个更友好的名字罢了。

正如 Git 基础 中所讨论的那样，存在两种类型的标签：附注标签和轻量标签。 可以像这样创建一个轻量标签：

    $ git update-ref refs/tags/v1.0 cac0cab538b970a37ea1e769cbbde608743bc96d

这就是轻量标签的全部内容——一个固定的引用。 然而，一个附注标签则更复杂一些。 若要创建一个附注标签，Git 会创建一个标签对象，并记录一个引用来指向该标签对象，而不是直接指向提交对象。 可以通过创建一个附注标签来验证这个过程（-a 选项指定了要创建的是一个附注标签）：

    $ git tag -a v1.1 1a410efbd13591db07496601ebc7a059dd55cfe9 -m 'test tag'

下面是上述过程所建标签对象的 SHA-1 值：

    $ cat .git/refs/tags/v1.1
    9585191f37f7b0fb9444f35a9bf50de191beadc2

现在对该 SHA-1 值运行 cat-file 命令：

    $ git cat-file -p 9585191f37f7b0fb9444f35a9bf50de191beadc2
    object 1a410efbd13591db07496601ebc7a059dd55cfe9
    type commit
    tag v1.1
    tagger Scott Chacon <schacon@gmail.com> Sat May 23 16:48:58 2009 -0700
    
    test tag

我们注意到，object 条目指向我们打了标签的那个提交对象的 SHA-1 值。 另外要注意的是，标签对象并非必须指向某个提交对象；你可以对任意类型的 Git 对象打标签。 例如，在 Git 源码中，项目维护者将他们的 GPG 公钥添加为一个数据对象，然后对这个对象打了一个标签。 可以克隆一个 Git 版本库，然后通过执行下面的命令来在这个版本库中查看上述公钥：

    $ git cat-file blob junio-gpg-pub

Linux 内核版本库同样有一个不指向提交对象的标签对象——首个被创建的标签对象所指向的是最初被引入版本库的那份内核源码所对应的树对象。

##### **远程引用** #####

我们将看到的第三种引用类型是远程引用（remote reference）。 如果你添加了一个远程版本库并对其执行过推送操作，Git 会记录下最近一次推送操作时每一个分支所对应的值，并保存在 refs/remotes 目录下。 例如，你可以添加一个叫做 origin 的远程版本库，然后把 master 分支推送上去：

    $ git remote add origin git@github.com:schacon/simplegit-progit.git
    $ git push origin master
    Counting objects: 11, done.
    Compressing objects: 100% (5/5), done.
    Writing objects: 100% (7/7), 716 bytes, done.
    Total 7 (delta 2), reused 4 (delta 1)
    To git@github.com:schacon/simplegit-progit.git
      a11bef0..ca82a6d  master -> master

此时，如果查看 refs/remotes/origin/master 文件，可以发现 origin 远程版本库的 master 分支所对应的 SHA-1 值，就是最近一次与服务器通信时本地 master 分支所对应的 SHA-1 值：

    $ cat .git/refs/remotes/origin/master
    ca82a6dff817ec66f44342007202690a93763949

远程引用和分支（位于 refs/heads 目录下的引用）之间最主要的区别在于，远程引用是只读的。 虽然可以 git checkout 到某个远程引用，但是 Git 并不会将 HEAD 引用指向该远程引用。因此，你永远不能通过 commit 命令来更新远程引用。 Git 将这些远程引用作为记录远程服务器上各分支最后已知位置状态的书签来管理。

