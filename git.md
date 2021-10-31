### git学习笔记

git是一种分布式版本控制系统。一般来讲如果一份代码有多个版本，就只能用重命名的方法管理，文件一多就相当不方便。git可以记录每个版本的文件，并且把不同分支的文件像一棵树一样组织起来，大大方便了管理。除此以外还可以由于其分布式的特点，可以在本地建立仓库，不需要每次都连接上远程仓库。

##### git分区结构

![img](https://upload-images.jianshu.io/upload_images/17945306-8c2faeae66c9769f?imageMogr2/auto-orient/strip|imageView2/2/w/1080/format/webp)

本地的git分为工作区(workspace)、暂存区(index)、本地仓库(repository)三个部分，除此之外还有一个远程仓库(remote)。其中，工作区就是当前的目录，如果一个文件从工作区被移走了，那么你在目录里面也看不到它了。暂存区是用来存放更新的区，只是把文件提交到暂存区而不是本地仓库的话是不算一次更改的。本地仓库才是存放版本的库，把文件交到这里才算一次真正的提交。远程仓库诸如github之类的网站或者是公司的服务器，顾名思义就是在本机之外存放代码的地方。

##### git本地命令

###### git init

建立一个本地仓库。执行这条指令之后当前目录就会出现.git文件夹，这样git就可以对这个文件夹进行版本控制，此时这个文件夹就变成了之前说的“工作区(workspace)”。

###### git status

获取当前文件的状态。所谓的“当前文件”其实说的是当前分支的文件，主要会呈现“未跟踪的文件(Untracked files)”、“工作区与暂存区不一致的文件(Changes not staged for commit)”、“暂存区与本地仓库不一致的文件(Changes to be committed)”。第一种一般就是新建了一个文件，还没有让git对其进行任何操作；第二种就是以前已经commit过了，但是后来又进行了修改，而且暂时还没有add；第三种就是已经add了，但是还没有commit的文件。

###### git add

把文件的修改从工作区提交到暂存区。这个感觉是用的最多的命令了，比如说我们新建了一个文件写了点代码，完事之后就先git add一下把它交到暂存区；后来又把这代码改改改，再git add，就可以把这次修改再存到暂存区。删除文件也算一次修改，所以如果把文件删了，也可以一个git add把“删除文件”这个更改提交上去。

###### git commit & git log

git commit 把暂存区的文件全部提交到本地仓库，git log 查看所有提交的记录。一般是`git commit -m '<message>'`，就可以提交文件同时把此次提交命名为`'<message>'`。提交完成之后如果用`git log`就可以看到此次提交的信息，包含作者、时间、版本号等信息。因为普通的git log输出的信息太多眼花缭乱，我们很多时候会用`git log --pretty=oneline`来让输出更直观。

###### git branch & git checkout

git branch 创建新分支，git checkout 切换分支。一般默认的会有一个master分支，然后如果`git branch <name>`就可以创建一个名字叫做`<name>`的分支。注意这个分支刚刚创建的时候，里面存的信息，和你创建它之前所在的分支，所存的信息，是完全一样的。创建完成之后可以`git checkout '<name>'`切换到名字为`<name>`的分支，此时你的工作区里面的文件往往会发生更改。

###### git rm & git restore

git rm 用来删除文件，git restore用来恢复文件。如果是要删除工作区的某个文件file，一般来讲就是先手动删除`rm -f file`，然后`git rm file`，最后`git add file`或者commit一下就行。当然这个删除不是那种开弓没有回头箭的永久删除，只要你还记得文件名，就可以用`git restore file`把误删除的文件恢复回来。

###### git reset

回退版本的关键指令。虽然上文说git restore可以恢复文件，但是很多时候我们是要彻彻底底地“回到过去”，把之前的修改都撤销了的那种。首先我们要用`git log --pretty=oneline`得到之前每一次commit的信息，一般输出出来是像这样的：

```git
7c063e35b8d0decab79f0ffcd78c841b002b3adc (HEAD -> master) no rename
5512fc6e9c4cb9bab56d5623da7053a5e29d09f1 10.22 first commit
b89921c04d08a5ea8af199778c3ac20154941ff4 (branch2) Merge branch 'branch1'
1309495fcdfe944473733d5758ebd3eb506576ea fourth commit
4893a1bfeae8340d17d401cf22b9c12424e022f1 third commit
1c7f6824c71a842dcda3b41f940293c10dcd862b second commit
deec19c83a18a38bb9277570fa8196dea6a131aa first commit
```

前面一大堆奇怪的乱码就是所谓的“版本号”。要回退到版本号为`<version>`，就使用指令`git reset -hard <version>`，比如说我们要回退到`third commit`的版本，就是`git reset -hard 4893a1bfeae8340d17d401cf22b9c12424e022f1`。但是这个版本号太长太长了很麻烦，所以一般来讲我们只要敲前几位就好，比如`git reset -hard 4893a1`这样。

不过很多时候我们只是想回到上一个版本而已，每次要查一堆乱七八糟的版本号还是很恼火。有个简便方式`git reset -hard HEAD^`，其中这个`HEAD^`就表示上一个版本的版本号。除此之外还可以套娃式地用\^符号，比如`HEAD^^`就表示上两个版本这样。

回退了版本如果发现不对，又想回来怎么办呢？如果此时使用`git log`查看，只能看到“比当前版本还老”的版本号。比如说如果是从final版本回到first版本，一查git log啥也没有，这就傻眼了。这个时候可以用`git reflog`查看之前输入的所有指令的记录，大概就是长这样

```
...(省略)
4893a1b HEAD@{10}: commit: final commit
1309495 HEAD@{11}: commit: fourth commit
...(省略)
```

找到final版本的commit指令的记录，前面就有这个版本的版本号的缩写，`git reset -hard 4893a1b`完事。

##### git 远程命令

###### git clone

这个命令用来拉取一个远程仓库到本地，格式为git clone (项目的url)。比如说你要拉取github上一个叫karshilov的人的jz项目，就可以直接`git clone https://github.com/karshilov/jz.git`，下载一段时间之后这个项目就到你的本地了。默认情况下克隆下来的仓库会被取名origin，如果想要自定义一个名字，就再在后面加一个自定义的名称，比如`git clone https://xxx.xxx.xx/xx.git test`就在克隆的同时把这个仓库命名为test。注意这个命令会把远程仓库里的所有分支全部拉取过来，每个文件的历史版本都会被拷贝过来，如果哪一天服务器挂了，你甚至可以通过客户端把所有信息全部恢复过来。

###### git remote

当我们从远程仓库与本地同步后，那么本地即与远程仓库进行了关联操作。这个命令就可以查看有哪些关联的远程仓库。使用参数-v可以看到更详细的信息，比如这样的：

```
origin  https://github.com/cheer-fun/pixivic-pc.git (fetch)
origin  https://github.com/cheer-fun/pixivic-pc.git (push)
```

git remote show还可以看得更详细，包含分支、url等等信息；除此以外，还可以使用`git remote add <shortname> <url>`来手动拉取一个远程仓库，效果和`git clone <url>  <shortname>`差不多，也可以由自己来自定义一个别名方便管理。

重命名还可以用`git remote rename <oldname> <newname>`，这样就不用每次只能在下载的时候命名了。

###### git fetch

如果有这种情况，就是你之前git clone把仓库拉取过来了，但是后来远程仓库里面有更新，使得你本地的版本已经不是最新的。这个时候要把这个更新拉过来，就要用`git fetch <remote>`。这个命令会访问远程仓库，从中拉取所有你还没有的数据。比如说我要更新本地的仓库，名字叫origin，就是`git fetch origin`；运行完这个指令，理论上讲远程仓库和本地仓库就同步了。不过实际上这个命令并不会帮你合并分支，它只是帮你下载，分支管理需要手动进行。

###### git push

将分支推送到远程服务器。具体来讲是`git push <remote> <branch>`，比如拉取了一个仓库命名为origin，现在要把master分支推送到远程仓库，就是`git push origin master`。推送的分支是已经存在本地代码仓库里的代码，也就是说如果你想要把手头（工作区）的代码推送到远程仓库，需要先git add再git commit，之后才能用git push。除此以外，如果你当前的本地版本不是最新的，git push也不能生效，需要你先git fetch把代码同步到本地才行。

###### git pull

和git fetch一样也是把代码同步到本地的操作。但是不一样的是git pull会直接帮你合并好分支。可以说git pull就等于git fetch + git merge。但是这两个的实现原理是有区别的，有文献说建议使用git fetch + git merge，少用git pull，原因我暂且蒙古。

