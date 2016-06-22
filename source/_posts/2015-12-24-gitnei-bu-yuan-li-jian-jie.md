---
layout: post
title: "Git内部原理简介"
date: 2015-12-24 00:49:55 +0800
comments: true
categories: Git
---


##创建仓库

	$ git init
	Initialized empty Git repository in /Users/zzmbp/demo/.git/
	
初始化一个git仓库后会生成一个.git目录，其中有如下的一些目录和文件：
	
	$ ll .git                                                              	total 24
	-rw-r--r--   1 zzmbp  staff    23B 12 10 21:56 HEAD
	drwxr-xr-x   2 zzmbp  staff    68B 12 10 21:56 branches
	-rw-r--r--   1 zzmbp  staff   137B 12 10 21:56 config
	-rw-r--r--   1 zzmbp  staff    73B 12 10 21:56 description
	drwxr-xr-x  11 zzmbp  staff   374B 12 10 21:56 hooks
	drwxr-xr-x   3 zzmbp  staff   102B 12 10 21:56 info
	drwxr-xr-x   4 zzmbp  staff   136B 12 10 21:56 objects
	drwxr-xr-x   4 zzmbp  staff   136B 12 10 21:56 refs

其中HEAD 文件、（尚待创建的）index 文件，和 objects 目录、refs 目录是Git的核心组成部分。objects 目录存储所有数据内容（blob、tree、commit）；refs 目录存储指向数分支最新的commit的指针；HEAD 文件指示目前所在的分支；index 文件保存暂存区信息。

<!--more-->

##blob
现在objects目录下只有两个空目录:

	$ ll .git/objects
	drwxr-xr-x  2 zzmbp  staff    68B 12 10 21:56 info
	drwxr-xr-x  2 zzmbp  staff    68B 12 10 21:56 pack
	
一个文件,内容为“Hello World！”,再将它放入暂存区

	$ vi new1.txt
	$ cat new1.txt                                                         
	Hello World!
	$ git add new1.txt
	
这时objects目录中多了一个以数字开头的目录“98”，“98”下有一个以38位字符命名的文件。
	
	$ ll .git/objects/98                                                   
	-r--r--r--  1 zzmbp  staff    29B 12 10 22:14 0a0d5f19a64b4b30a87d4206aade58726b60e3

980a0d5f19a64b4b30a87d4206aade58726b60e3其实是一个40位的SHA-1 哈希值，是由文件内容（这里就是new1.txt的内容）加上特定的头部信息一起做SHA-1 校验运算而得的校验和。校验和的前两个字符用于命名子目录，余下的 38 个字符则用作文件名。new1.txt的内容被压缩后存在了这个文件中。这是Git存储内容的方式。      

通过这个SHA-1 值可以通过`git cat-file`取回new1.txt的数据

	$ git cat-file -p 980a0d5f19a64b4b30a87d4206aade58726b60e3	Hello World!
这种类型的对象就叫做blob。一个文件被放入暂存去，Git就为它创建一个blob，存储其内容。`git cat-file`可以查看一个Git对象的类型

	$ git cat-file -t 980a0d5f19a64b4b30a87d4206aade58726b60e3
	blob

##Tree 和 Commit

把暂存区的文件放到仓库里

	$ git commit -m 'a	dd new1.txt'
	[master (root-commit) 21661c4] add new1.txt 	1 file changed, 1 insertion(+)
	create mode 100644 new	1.txt
 	
现在objects下又多了两个目录："21"和"cf"

	$ find .git/objects -type f
	.git/objects/21/661c40f062c04d4fe4922f297c1a49c2aada03
	.git/objects/98/0a0d5f19a64b4b30a87d4206aade58726b60e3
	.git/objects/cf/5fd42aadedd7f2b0fc9635bcd1dceaeaadff03
	
新增了两个对象tree和commit.
	
	$ git cat-file -t cf5fd42aadedd7f2b0fc9635bcd1dceaeaadff03 
	tree
	$ git cat-file -t 21661c40f062c04d4fe4922f297c1a49c2aada03
	commit
tree对象中存储了blob对象的信息，包括SHA-1值和文件名。tree对象中也可以存储别的tree对象。

	$ git cat-file -p cf5fd42aadedd7f2b0fc9635bcd1dceaeaadff03
	100644 blob 980a0d5f19a64b4b30a87d4206aade58726b60e3	new1.txt
commit对象存储了tree对象和上一个commit的信息，因为这个master分支上第一commit，所以没有上一个commit的信息。

	$ git cat-file -p 21661c40f062c04d4fe4922f297c1a49c2aada03
	tree cf5fd42aadedd7f2b0fc9635bcd1dceaeaadff03
	author zhouzhou02 <zhouzhou02@meituan.com> 1449759442 +0800
	committer zhouzhou02 <zhouzhou02@meituan.com> 1449759442 +0800

	add new1.txt
	
现在再添加一个文件new2.txt
	
	$ vi new2.txt
	$ cat new2.txt
	哈哈哈哈哈哈
	$ git add new2.txt
	$ git commit -m 'add new2.txt'
	[master df14c29] add new2.txt
	1 file changed, 1 insertion(+)
	create mode 100644 new2.txt

从上面信息[master df14c29]可以看到新增一个commit，commit号前七位：df14c29，看看它的内容：

	$ git cat-file -p df14c29
	tree 257ce24780109049cb6f4a1bc6271f0b147d52bd
	parent 21661c40f062c04d4fe4922f297c1a49c2aada03
	author zhouzhou02 <zhouzhou02@meituan.com> 1449762137 +0800
	committer zhouzhou02 <zhouzhou02@meituan.com> 1449762137 +0800
	
	add new2.txt

这个commit指向了tree和上一个commit，现在的存储结构看起来是这样的：

	 /Users/zzmbp/Desktop/屏幕快照 2015-12-11 上午12.09.01.png

##Tag
tag对象（tag object）非常类似于一个commit对象——它包含一个标签创建者信息、一个日期、一段注释信息，以及一个指针。
	
	$ git tag -a v1.0 df14c29 -m 'first tag'
类似commit，创建一个tag对象后也会在objects下创建一个目录和文件，同时还在.git/refs/tags目录下创建一个tag的引用

	$ cat .git/refs/tags/v1.0
	40e912007f778c47b34984fb5e8ce6fc0aacd8f4
	$ git cat-file -t 40e912007f778c47b34984fb5e8ce6fc0aacd8f4
	tag
	$ git cat-file -p 40e912007f778c47b34984fb5e8ce6fc0aacd8f4
	object df14c299407af82fc3f4925a1e406e10ec4f740f
	type commit
	tag v1.0
	tagger zhouzhou02 <zhouzhou02@meituan.com> 1449767423 +0800
	
	first tag

tag对象中存储了它指向的对象的SHA-1值等信息。

##引用
一个文件来保存 SHA-1 值，并给文件起一个简单的名字，然后用这个名字指针来替代原始的 SHA-1 值m，可以在.git/refs 目录下找到这类含有 SHA-1 值的文件。现在只有一个master分支，refs/heads对应有一个名为master的文件

	$ ll .git/refs/heads
	-rw-r--r--  1 zzmbp  staff    41B 12 10 23:42 master
	
master文件内容就是刚才最后提交的那个commit的commit号
	
	$ cat .git/refs/heads/master
	df14c299407af82fc3f4925a1e406e10ec4f740f
	
再看.git/HEAD文件的内容
	
	$ cat .git/HEAD
	ref: refs/heads/master

HEAD文件存储一个路径，通过路径找到commit号。由此可见各种引用最终都指向SHA-1值，引用更容易记忆，操作方便。

##packfile
objects目录下现在有这些文件：

	$ find .git/objects -type f                                            
	.git/objects/21/661c40f062c04d4fe4922f297c1a49c2aada03
	.git/objects/25/7ce24780109049cb6f4a1bc6271f0b147d52bd
	.git/objects/40/e912007f778c47b34984fb5e8ce6fc0aacd8f4
	.git/objects/98/0a0d5f19a64b4b30a87d4206aade58726b60e3
	.git/objects/cf/5fd42aadedd7f2b0fc9635bcd1dceaeaadff03
	.git/objects/df/14c299407af82fc3f4925a1e406e10ec4f740f
	.git/objects/e8/a54ebd111c9d95a2c73738766b705dee751299

这些是刚才创建的那些blob、tree、commit、tag对象。Git 最初向磁盘中存储对象时所使用的格式被称为“松散（loose）”对象格式。 但是，Git 会时不时地将多个这些对象打包成一个称为“包文件（packfile）”的二进制文件，以节省空间和提高效率。 当版本库中有太多的松散对象，或者你手动执行 git gc 命令，或者你向远程服务器执行推送时，Git 都会这样做。 要看到打包过程，你可以手动执行 git gc 命令让 Git 对对象进行打包：

	$ git gc                                                               
	Counting objects: 7, done.
	Delta compression using up to 8 threads.
	Compressing objects: 100% (4/4), done.
	Writing objects: 100% (7/7), done.
	Total 7 (delta 0), reused 0 (delta 0)

刚才objects目录下那些文件消失了，多了两个新文件

	$ find .git/objects -type f
	.git/objects/info/packs
	.git/objects/pack/pack-4788682aff34a626ef71c24ed0ead3f47906e840.idx
	.git/objects/pack/pack-4788682aff34a626ef71c24ed0ead3f47906e840.pack
这两个文件是新创建的包文件和一个索引。包文件包含了刚才从文件系统中移除的所有对象的内容。 索引文件包含了包文件的偏移信息，我们通过索引文件就可以快速定位任意一个指定对象。使用 `git verify-pack`查看已打包的内容：
	
	$ git verify-pack -v .git/objects/pack/pack-4788682aff34a626ef71c24ed0ead3f47906e840.idx
	df14c299407af82fc3f4925a1e406e10ec4f740f commit 231 149 12
	21661c40f062c04d4fe4922f297c1a49c2aada03 commit 183 121 161
	40e912007f778c47b34984fb5e8ce6fc0aacd8f4 tag    140 124 282
	257ce24780109049cb6f4a1bc6271f0b147d52bd tree   72 74 406
	cf5fd42aadedd7f2b0fc9635bcd1dceaeaadff03 tree   36 46 480
	980a0d5f19a64b4b30a87d4206aade58726b60e3 blob   13 22 526
	e8a54ebd111c9d95a2c73738766b705dee751299 blob   19 17 548
	non delta: 7 objects
	.git/objects/pack/pack-4788682aff34a626ef71c24ed0ead3f47906e840.pack: ok

 
  
