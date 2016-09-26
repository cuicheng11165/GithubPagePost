---
layout: post
title:  "Git Shell 基本命令(官网脱水版)"
date:   2016-02-03
author: 独上高楼
category: 建站
tags: [github]
---
## 用户信息
当安装完 Git 应该做的第一件事就是设置你的用户名称与邮件地址。 这样做很重要，因为每一个 Git 的提交都会使用这些信息，并且它会写入到你的每一次提交中，不可更改：  

$ **git config --global user.name "John Doe"**  
$ **git config --global user.email johndoe@example.com**

## 获取命令帮助

$ **git help verb**  
$ **git verb help**  
$ **man git-verb**  

## 检查配置信息
如果想要检查你的配置，可以使用 git config --list 命令来列出所有 Git 当时能找到的配置。

## 在现有目录中初始化仓库
如果你打算使用 Git 来对现有的项目进行管理，你只需要进入该项目目录(本地路径)并输入：    
$ **git init**

如果你是在一个已经存在文件的文件夹（而不是空文件夹）中初始化 Git 仓库来进行版本控制的话，你应该开始跟踪这些文件并提交。 你可通过 git add 命令来实现对指定文件的跟踪，然后执行 git commit 提交： 

$ **git add ***  
$ **git add LICENSE**  
$ **git commit -m 'initial project version'**

## 下载代码
如果你想下载托管在git上的代码，只需要知道代码托管路径，然后输入：  
$ **git clone https://github.com/libgit2/libgit2**  
如果你想重命名下载到本地的目录那么输入：  
$ **git clone https://github.com/libgit2/libgit2 foldername**  
这将执行与上一个命令相同的操作，不过在本地创建的仓库名字变为 foldername。

## 记录每次更新

### 检查当前文件状态
要查看哪些文件处于什么状态，可以用 git status 命令。 如果在克隆仓库后立即使用此命令，会看到类似这样的输出：  

$ **git status**    


现在，让我们在项目下创建一个新的 README 文件。 如果之前并不存在这个文件，使用 git status 命令，你将看到一个新的未跟踪文件：  

$ **echo 'Readme文件内容' > README**  
$ **git status**

使用这个命令你会看到哪些文件没有被版本库记录了。

### 跟踪新文件
把刚刚创建的README放入版本库中跟踪，只需要：
$ **git add README**  
$ **git status**

### 状态简览

$ **git status -s**  
　M README  
MM Rakefile  
A　 lib/git.rb  
M　 lib/simplegit.rb  
??　LICENSE.txt  
新添加的未跟踪文件前面有 ?? 标记，新添加到暂存区中的文件前面有 A 标记，修改过的文件前面有 M 标记。 你可能注意到了 M 有两个可以出现的位置，出现在右边的 M 表示该文件被修改了但是还没放入暂存区，出现在靠左边的 M 表示该文件被修改了并放入了暂存区。 例如，上面的状态报告显示： README 文件在工作区被修改了但是还没有将修改后的文件放入暂存区,lib/simplegit.rb 文件被修改了并将修改后的文件放入了暂存区。 而 Rakefile 在工作区被修改并提交到暂存区后又在工作区中被修改了，所以在暂存区和工作区都有该文件被修改了的记录。

### 忽略文件
一般我们总会有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。 通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。 在这种情况下，我们可以创建一个名为 .gitignore 的文件，列出要忽略的文件模式。 来看一个实际的例子：

$ **cat .gitignore**  
这个命令表示读取.gitignore文件下的内容，并输出到控制台上

### 查看已暂存和未暂存的修改
要查看尚未暂存的文件更新了哪些部分，不加参数直接输入 **git diff**
若要查看已暂存的将要添加到下次提交里的内容，可以用 **git diff --cached** 命令

### 提交更新
提交更新之前，git status看下所有修改是否都被记录了，然后  
$ **git commit**

可能会进入vim模式，这时候 ctrl+c 输入 :wq  保存文件并退出，也可以

$ **git commit -m "Story 182: Fix benchmarks for speed"**

### 跳过使用暂存区域
正常的提交流程是:  
* **git status** 查看状态  
* **git add .** 放入暂存区域  
* **git commit -m "提交记录"**  

来提交，如果你觉得这样太麻烦，可以-a来跳过暂存区域： 
$ **git commit -a -m 'added new benchmarks'**

### 移除文件
移除文件分两种，是否在当前目录保留这个文件，需要保留则：  
$ **git rm --cached filename**
不需要保留则：  
$ **git rm filename**

## 查看提交历史
$ **git log**  
即可。至于更高端的用法，查看[文档](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2)

## 撤销操作
类似于svn的revert操作：  
$ **git checkout -- filename.md**    
$ **git status**

## 远程仓库的使用
* 新代码publish到新的仓库
如果你想把你的没有被git托管的代码发布到github上，那么按照下面的操作：  
**echo "readme文件内容" >> README.md**  
**git init**  
**git add README.md**  
**git commit -m "first commit"**  
**git remote add origin https://github.com/cuicheng11165/1758.git**  
**git push -u origin master**  

* 现有仓库发布到远程仓库
如果git已经在本地托管了，那么只需要：  
**git remote add origin https://github.com/cuicheng11165/1758.git**  
**git push -u origin master**  

拉取远程的改动分为pull 和fetch，pull相当于fetch+merge  
* fetch的使用：  
**git fetch origin master**  
**git merge**  
* pull的使用:  
**git pull origin master**  

[官方文档](https://git-scm.com/book/zh/v2)
