# 安装配置&基础概念

```git
#查看git版本 确认是否安装完成
git --version

#配置用户信息
git config --global user.name "ccq"
git config --global user.email "chengchaoqun@hotmail.com"

#删除配置用户信息
git config --global --unset user.name
git config --global --unset user.email

#查看已有的配置信息
git config --list

#配置信息的级别
##当前操作系统
--system 
##当前用户
--global
##不说明就是 当前项目

```

## 区域

### 工作区

1. 沙箱环境, Git不会管理工作区

### 暂存区

```
#查看暂存区内容
git ls-files -s
```

1. 对于工作区所做的一系列操作, 都记录在暂存区
2. 提交到版本库才算一个版本
3. 工作区--->暂存区--->版本库

### 版本库



## 对象

### Git对象

```
#创建版本库
git init
```

1. key : value 组成的键值对 (key 是 value对应的hash)

2. 键值对在git内部是一个Blob类型

   1. 向数据库写入内容 并返回该内容的键值(hash)

      1. ```git
         echo "test content" | git hash-object -w --stdin
         ```

         

   2. -w 选项指示 hash-object 命令存储数据对象, 若不指定此选项, 则该命令仅返回对应的键值

   3. --stdin (standard input) 选项则指示该命令从标准输入读取内容; 若不指定此选项, 则须在命令尾部给出待存储文件的路径

   4. 存文件

      1. ```
         git hash-object -w 文件路径
         ```

   5. 返回对应文件的键值(hash)

      1. ```
         git hash-object 文件路径
         ```

3. 查看Git如何存储数据

   1. ```
      find ./.git/objects -type f
      ```

   2. 返回 

      1. ```
         ./.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
         ```

   3. 这就是开始时 Git  存储内容的方式 ： 一个文件对应一条内容 。 校验和的前两个字符用于命名子目录，余下的 38  个字符则用作文件名。

4. 根据键值拉取数据

   1. ```
      git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4	查看hash对应的内容
      git cat-file -t d670460b4b4aece5915caf5c68d12f560a9fe3e4	查看hash对应的类型
      ```

   2. -p 选项可指示该命令自动判断内容的类型，并为我们显示格式友好的内容

   3. -t 表示显示对象类型

   4. 返回 对应文件的内容

5.  对一个文件进行简单的版本控制

   1. 创建一个新文件并将其内容存入数据库

      1. ```
         echo "version1" > test.txt	生成test.txt文件并向其中写入version1
         git hash-object -w test.txt	将生成的文件添加到数据库生成一个git对象
         ```

      2. 返回该文件生成的git对象的键值(hash)即"version1" 内容对应的hash

   2. 向文件里写入新内容，并再次将其存入数据库

      1. ```
         echo "version2" > test.txt
         git hash-object -w test.txt
         ```

      2. 返回新的hash

   3. 查看数据库的内容

      1. ```
         #查看数据库所有对象
         find ,git/objects/ -type f
         #返回文件对应的内容
         git cat-file -p 文件对应的hash
         #返回文件对应的类型 (blob)
         git cat-file -t 文件对应的hash
         ```

      2. 利用 cat-file -t 命令，可以让 Git 告诉我们其内部存储的任何对象类型

      3. 返回 blob

6. 问题:

   1. 记住文件的每一个版本所对应的 hash 值并不现实
   2. 在 Git 中，文件名并没有被保存——我们仅保存了文件的内容
   3. 解决方案：树对象
   4. 当前的操作都是在对本地数据库进行操作 不涉及暂存区





### 树对象

1. 树对象（tree object），它能解决文件名保存的问题，也允许我们将多个文件组织到一起。Git 以一种类似于UNIX 文件系统的方式存储内容。所有内容均以树对象和数据对象(git  对象)的形式存储，其中树对象对应了 UNIX 中的目录项，数据对象(git  对象)则大致上对应文件内容。一个树对象包含了一条或多条记录（每条记录含有一个指向 git  对象或者子的 树对象的 SHA-1  指针 ， 以及相应的模式、类型、文件名信息 ）。一个树对象也可以包含另一个树对象。

2. 我们可以通过 update-index；write-tree；read-tree 等命令来构建树对像并塞入到暂存区。

   1. 利用  update-index  命令 为 test.txt 文件的首个版本——创建一个暂存区。并通过  write-tree  命令生成树对像

   2. ```
      git update-index --add --cacheinfo 10064
      560a3d89bf36ea10794402f6664740c284d4ae3b test.txt
      git write-tree	将暂存区内的内容生成一个tree对象 并添加到版本库作为一个版本
      ```

   3. update-index 代表为该文件的版本生成一个暂存区

   4. git write-tree 代表为暂存区内的内容生成一个树对象并添加到版本库作为一个版本

   5. --add 因为此前该文件并不在暂存区中 首次需要 --add

   6. --cacheinfo 因为将要添加的文件位于 Git  数据库中，而不是位于当前目录下 所有需要从版本库获取数据

   7. 文件模式为 

      1. 100644 ，表明这是一个普通文件
      2. 100755 ，表示一个可执行文件；
      3. 120000 ，表示一个符号链接

3. 新增 new.txt 将 new.txt 和 test.txt 文件的第二个个版本塞入暂存区。并通过  write- -e tree  命令生成树对像

   1. ```
      #新建文件
      echo "new v1" > new.txt
      
      #将test.txt第二个版本从版本库拿出来加入到暂存区覆盖第一个版本
      git update-index -cacheinfo 10064 32e2018080ca696bc9a781dcf13b3429d8bbb4a6 
      test.txt
      
      #直接将new.txt文件生成git对象然后放入到暂存区,将下面两步合成一步
      #1. git hash-object -w new.txt
      #2. git update-index --add --cacheinfo 10064
      eae614245cc5faa121ed130b4eba7f9afbcc7cd9 new.txt
      git update-index --add new.txt
      
      #为暂存区的内容生成tree对象作本一个版本添加到版本库
      git write-tree
      ```

4. 将第一个树对象加入第二个树对象，使其成为新的树对象

   1. ```
      git read-tree --prefix=bak 第一个树对象的hash
      git write-tree
      ```

   2. read-tree命令可以吧树对象读入暂存区

   3. 最后的树对象示意图

   4. ![image-20210515172704176](C:\Users\Chaoq\AppData\Roaming\Typora\typora-user-images\image-20210515172704176.png)

   5. 现在有三个树对象（执行了三次 write-tree），分别代表了我们想要跟踪
      的不同项目快照。然而问题依旧：若想重用这些快照，你必须记住所有三个
      SHA-1 哈希值。 并且，你也完全不知道是谁保存了这些快照，在什么时刻保
      存的，以及为什么保存这些快照。 而以上这些，正是提交对象（commit object）
      能为你保存的基本信息

暂存区的覆盖是根据文件名. 存在的文件名会被覆盖, 不存在的会被新增

Git对象代表文件的一次次版本

Tree对象代表项目的一次次版本

git write-tree 向版本库写入内容, 不会情况暂存区

在 Git 中每一个文件（数据）都对应一个 hash（类型 blob）每一个树对象都对应一个 hash（类型 tree）

我们可以认为树对象就是我们项目的快照



### 提交对象

1. 创建提交对象

   1. ```
      echo "first commit" | git commit-tree 树对象对应的hash
      ```

   2. 返回提交对象的hash

2. 查看提交对象

   1. git cat-file -p 提交对象的hash
   2. 返回
   3. ![image-20210515173333896](C:\Users\Chaoq\AppData\Roaming\Typora\typora-user-images\image-20210515173333896.png)

3. 提交对象的格式

   1. 提交对象的格式很简单：

   2. 它先指定一个顶层树对象，代表当前项目快照；然后是作者/提交者信息（依据你的  user.name 和  user.email 配置来设定，外加一个时间戳）；留空一行，最后是提交注释接着，我们将创建另两个提交对象，它们分别引用各自的上一个提交（作为其父提交对象）：

   3. ```
      echo "提交的注释" | git commit-tree 当前需要提交树对象的hash -p 前一个提交对象的hash
      ```

4. git commit- - tree  不但生成提交对象 而且会将对应的快照（树对象）提交到本地库中

## Linux命令

```
#清楚屏幕
clear

#当前目录下所有的
./

#向控制台输出信息
echo "text"

#创建文件 并且写入信息
echo "text" > test1.txt

#查看当前目录下的子文件 $ 子目录
ll

#将对应目录下的子孙文件&子孙目录平铺在控制台
find 目录名

#将对应目录下的文件平铺在控制台
##f代表类型为file,文件不是文件夹
find 目录名 -type f

#删除文件
rm 文件名

#重命名
mv 源文件名 新文件名

#查看对应文件的内容
cat 文件名

#vim编辑器编辑文件
vim 文件名
## 按 i  进入插入模式, 即可以进行文件的编辑
## 按 ESC & 按 : 进行命令的指向
### q! 强制退出 不保存
### wq 保存退出
### set nu 设置行号
```



# 高层命令

## 基本的操作

```
#第一步初始化本地仓库
git init

#创建文件
echo "ccq v1" > ccq.txt

#添加到版本库
#1.将当前目录下所有改动的文件生成一个git对象添加到版本库
#2.将版本库里的git对象拿出来放到暂存区(仍然是blob类型)
#先后顺序是先到版本库变成git对象, 再从版本库移动到暂存区
#只有提交的时候才将暂存区里的所有git对象生成一个tree对象加入到版本库
#再将生成的tree对象拿出来做一些注释封装成一个commit对象添加到版本库
git add ./

#提交
#当前版本库有一个git对象, 暂存区有一个git对象, 还未生成树对象
#下面的操作包含了生成tree对象和commit对象
#一次提交至少涉及了1个git对象 1个tree对象 1个commit对象
git commit -m "提交注释,解释提交的内容"

git操作基本的流程
	创建工作目录 对工作目录进行修改
	git add ./
		git hash-object -w 文件名 (修改了多少个工作目录的文件, 此命令就要被执行多少次)
		git update-index --add --cacheinfo 10064 git对象的hash 文件名
	git commit -m "注释内容"
		git write-tree
		git commit-tree

```

## CRUD

1. CRUD

2. ```
   git init				初始化仓库
   git status				检查文件状态
   git diff				查看未暂存的修改
   git diff --staged		查看未提交的暂存
   git log --oneline		查看提交的历史记录
   git add ./ 				将修改添加到暂存区
   
   #将暂存区添加到版本库
   git commit				进入vim模式进行注释的书写, 通常用于提交需要写大段的情况
   git commit -m "注释"	   将暂存区提交到版本库
   git commit -a 			跳过暂存的过程(git add)直接将已经暂存的文件提交
   git commit -a -m		上面增加注释
   
   #删除
   git rm 文件名					删除指定文件,并且将修改提交到暂存区(等价于一下两个操作)
   	1. rm 文件名				删除工作区的文件
   	2. git add ./			  将修改提交到暂存区
   
   #重命名
   git mv 文件名from 文件名to		将工作目录中的文件进行重命名再将修改添加到暂存区
   	1. mv 文件名from 文件名to		重命名文件
   	2. git rm 文件名from		  删除初始文件并提交
   	3. git add 文件名to		  添加重命名后的文件
   ```

## 记录每次更新到仓库

1. 所有文件的状态有两种
   1. 已跟踪
      1. 已提交
      2. 已修改
      3. 已暂存
   2. 未跟踪



# Git分支

```
git branch					  					  显示分支列表
git branch 分支名									创建分支
git checkout 分支名								切换到指定分支
git branch -d 分支名								删除分支
git branch -D 分支名								强制删除未合并的分支
git branch -v 									  查看当前分支的最后一次提交
#在指定的提交对象创建分支
#切换到哪个新创建的分支以后就可以看到以前版本的信息
#使用完以后再切回当前的主分支, 删除新创建的分支即可
git branch 分支名 commitHash				新建一个分支并且使分支指向对应的提交对象		
git log --oneline --decorate --graph --all		  查看分支历史
```



## 创建分支

1. ```
   #创建新分支
   git branch 分支名
   
   #在指定的提交对象创建分支
   #切换到哪个新创建的分支以后就可以看到以前版本的信息
   #使用完以后再切回当前的主分支, 删除新创建的分支即可
   git branch 分支名 commitHash				新建一个分支并且使分支指向对应的提交对象
   
   git checkout -b 分支名	在当前对象新建一个分支并且切换到新建的分支
   	相当于两个操作
   	git branch 分支名
   	git checkout 分支名
   ```

   

## 切换分支

1. ```
   #切换到指定分支
   git checkout 切换到的分支名
   ```

2. 每次切换分支前当前分支一定是已提交的状态

3. 坑:

   1. 在切换分支时, 如果当前分支上有未暂存的修改(第一次) 或者 有未提交的暂存(第一次)
   2. 分支可以切换成功, 但是这种操作可能会污染其他分支

4. 切换分支会改变三个地方

   1. HEAD
   2. 暂存区
   3. 工作目录

5. 版本库只会无限变多

## 删除分支

```
删除当前所在的分支, 需要先切到其他分支再删除
git branch -d 分支名				删除已经合并过的分支
git branch -D 分支名				强制删除未合并的分支
```



## 配别名

```
#配置操作的别名
git config --global alias.别名 "需要代替的操作(这里开头不能加git,否则执行的时候等于git git ...)"
配置后: git 别名
配置的时候去掉本身操作的git
git config --global alias.lol "log --oneline --decorate --graph --all"

如果配置的操作只有一个单词可以不加双引号
```



## 合并分支

```
git checkout master	先切回主分支
git merge 分支名	再合并需要合并的分支
```



## 实际案例

1. 在自己的分支上进行工作,需要修改主分支上的一个bug
   1. 提交自己正在作业的分支上的代码
   2. 切回master分支, 
   3. 开一个分支解决bug, 提交
   4. 回到master分支, 合并已经解决bug的分支
   5. 切回自己的工作分支, 继续工作,
      1. 但是当前的工作分支是原本基于master创建的
      2. 修改完master分支的bug以后, master分支已经向前走了
      3. 当前分支落后于master且有可能合并的过程会产生冲突
   6. 合并当前分支到master分支
      1. 切回master分支
      2. 合并工作分支
      3. 产生冲突需要处理冲突
      4. 处理完 git add ./ 相当于处理完成冲突
      5. git commit 提交后分支就被自动合并了
      6. ![image-20210515204747528](C:\Users\Chaoq\AppData\Roaming\Typora\typora-user-images\image-20210515204747528.png)



## Git存储

```
git stash 					将未完成的修改保存到一个栈上
git stash list				查看栈上的内容

#apply命令不会将获得的内容从栈中删除
git stash apply				默认获得栈顶的内容
git stash apply stash@{2}	获得指定的存储的内容
git stash drop stash@{2}	删除指定的存储内容

git stash pop 				应用存储的内容, 然后立即从栈上扔掉它
	相当于先应用(apply) 再从栈内删除(drop)
```



# 后悔药

1. 工作区
   1. 如何撤回自己在工作目录中的修改
2. 暂存区
   1. 如何撤回自己的暂存区
3. 版本库
   1. 如何撤回自己的提交
      1. 重新给用户一次机会改注释,再去提交一次
      2. 没有真正的从版本库回退

```
# use "git restore <file>..." to discard changes in working directory
git restore filename				撤回自己在工作目录的修改

# use "git restore --staged <file>..." to unstage
git restore --staged filename		撤回自己的暂存区

#先将自己需要修改的内容添加到暂存区
#再提交, 而这次提交可以覆盖上一次注释的内容, 使日志更加美观
(git add ./)
git commit --amend	实际上是重新提交了一次修改,但是这次的注释内容可以覆盖上一次的注释内容
```

## 重置 reset

```
当前进行了三次提交, 工作区已经暂存区的内容如下
工作目录中file.txt文件			暂存区file.txt
file.txt v1					file.txt v1			
file.txt v2					file.txt v2
file.txt v3					file.txt v3	
```



### 三部曲

#### --soft

1. 第一部: git reset --soft HEAD~  (--amend)

   1. 只动了HEAD

   2. ```
      git reset --soft HEAD~	撤销上一次的git commit操作
      ```

   3. ```
      回到前面一次提交之前以后
      git reset --soft 第三次提交对象的hash 可以回到第三次提交(撤销了第一次的reset操作)
      git reflog可以看到之前第三次提交的hash
      ```

   4. ```
      git reset --soft HEAD~ 
      HEAD->master 指向的是第二个提交对象
      操作后, 暂存区与工作目录的情况
      工作目录中file.txt文件			暂存区file.txt
      file.txt v1					file.txt v1			
      file.txt v2					file.txt v2
      file.txt v3					file.txt v3	
      没有未暂存的修改
      有未提交的暂存
      ```


#### --mixed

1. 第二部: git reset --mixed HEAD~

   1. 动HEAD(带着分支一起移动)

   2. 动暂存区

   3. ```
      回到了前面一次提交之前
      并且是将修改添加到暂存区之前
      此时的状态:工作区还有未暂存的修改
      git reset --mixed HEAD~
      ```

   4. ```
      git reset --mixed HEAD~ 
      HEAD->master 指向的是第二个提交对象
      操作后, 暂存区与工作目录的情况
      工作目录中file.txt文件			暂存区file.txt
      file.txt v1					file.txt v1			
      file.txt v2					file.txt v2
      file.txt v3		
      有未暂存的修改
      ```


#### --hard

1. 第三部: git reset --hard HEAD~

   1. 动HEAD

   2. 动暂存区

   3. 动工作目录

   4. 撤销了最后的一次提交(git commit), git add, 以及工作目录中所有的内容

   5. ```
      git reset --hard HEAD~ 
      HEAD->master 指向的是第二个提交对象
      操作后, 暂存区与工作目录的情况
      工作目录中file.txt文件			暂存区file.txt
      file.txt v1					file.txt v1			
      file.txt v2					file.txt v2
      没有未暂存的修改
      没有未提交的暂存
      ```

### checkout

```
git checkout commithash & git reset --hard commithash
只动HEAD					动HEAD并且带着分支一起走

checkout:
	如果当前目录下还有未被git跟踪的文件, checkout的时候回被带到切换的分支的工作目录
	所以checkout对工作目录是安全的
reset --hard:
	强制覆盖工作目录, 工作目录下的所有修改也会被删除, 如果未被提交则无法找回
```

![image-20210515231626273](C:\Users\Chaoq\AppData\Roaming\Typora\typora-user-images\image-20210515231626273.png)



```
git checkout commithash	切换分支
git checkout --filename 
	相当于 git reset --hard commithash --filename(但是这样写语法有错)
	不会动HEAD, 不会动暂存区(第一部,第二部都没做)
	只会动工作目录(撤销了工作目录的修改)
git checkout commithash <file>
	跳过第一部
	更新暂存区
	更新工作目录
```





### 路径reset

```
#HEAD 不动(跳过第一部), 但是回退了暂存区指定filename文件的状态变成未暂存
#只动了暂存区
git reset [--mixed] HEAD filename
```

1. HEAD相当于一个提交对象
   1. 一个提交对象对应一个树对象
   2. 一个树对象可能对应多个git对象
2. 不指定filename, 动了HEAD动了暂存区
3. 指定了filename, HEAD 不动(跳过第一部), 但是撤销了暂存区指定filename文件的状态变成未暂存,不会影响其他文件



## 数据恢复

1. 通过reflog找到需要恢复的分支的commithash

2. ```
   #在需要恢复的提交对象上创建一个名为recover-branch的分支指向这个提交对象
   git branch recover-branch commithash
   ```



## 打tag

### 列出标签

```
git tag	列出所有的标签
git tag -l 'v1.8.5*'	列出'v1.8.5'的所有标签	
```

### 打上标签

```
git tag tagname	给当前分支打上tagname的标签
git tag tagname commithash	给指定的提交对象打上标签
```

### 查看特定标签

```
#git show 可以显示任意类型的对象（git 对象 树对象 提交对象 tag 对象）
git show tagname
```



### 检出标签

```
#查看某个标签所指向的文件版本
#但是这个tagname没有对应一个分支, HEAD移动到tagname对应的提交对象的时候, 就没有指向分支, 
#处于一个头部分离的状态
git checkout tagname
#需要在当前标签对应的提交对象上创建分支,让HEAD指向这个分支
git checkout -b 分支名
```



### 删除标签

```
#删除指定的 tagname标签
git tag -d tagname
```





# Git远程仓库

## 推送本地仓库到远程仓库

```
#创建本地仓库
git init
#添加一个需要推送到的远程 Git 仓库，同时指定一个你可以轻松引用的别名
git remote add <shortname> <url(HTTPS | SSH)>
#显示远程仓库使用的 Git 别名与其对应的 URL
git remote -v
#配置用户信息
git config user.name "用户名"
git config user.email "邮箱"
#推送本地仓库到远程仓库
git push 远程仓库别名 分支名

#本地创建的仓库推送到远程仓库会有三个分支
#1.本地创建的master分支
#2.本地分支推送到远程仓库时生成的(被推送分支的)远程跟踪分支
#3.远程仓库的远程分支
```



## clone远程仓库到本地

```
新建空文件夹, 不需要init 
git clone <远程仓库HTTPS | SSH 地址>
	完整的把远程库下载到本地
	创建origin远程地址别名
	初始化本地库

打开克隆下来的仓库文件夹 git bash here

自动已经创建默认的远程仓库的别名 
git remote -v 显示origin

配置自己的用户信息
git config user.name ""
git config user.email ""

克隆下来的项目默认为远程仓库的的master分支创建远程跟踪分支(origin/master)

```



## fetch 在同步到远程仓库的本地仓库获取更新

```
#fetch 将远程仓库更新到远程跟踪分支
git fetch <远程仓库配置的别名> 更新了远程跟踪分支

#fetch 以后仍然在自己的本地分支,查看更新需要切换到远程跟踪分支
git checkout 远程跟踪分支
#切换后处于头部分离的状态 因为HEAD指向了远程跟踪分支没有指向本地分支

#将远程跟踪分支的更新合并到本地master分支
#1.切回到本地master分支
git checkout master
#2.合并远程跟踪分支
git merge 远程跟踪分支
```

## Pull

```
git pull = fetch + merge
git pull [远程仓库别名] [远程跟踪分支名]
git fetch [远程仓库别名] [远程跟踪分支名]
git merge [远程仓库别名/远程跟踪分支名]

想要获取远程仓库对应分支的更新, 需要先将当前的 本地分支 跟踪对应的 远程跟踪分支
(当前在的分支就是需要跟踪远程分支的那个分支)git branch -u 远程跟踪分支
跟踪以后 git pull  git push 就会直接推送和拉去对应分支的信息

```



## 解决冲突

1. 如果不是基于GitHub远程库的最新版所做的修改, 不能推送, 必须先拉
2. 拉去下来如果进入冲突状态, 则按照"分支冲突解决" 操作解决即可





## 一次团队协作

1. 项目经理初始化远程仓库

   1. 一定要初始化空的仓库, 在github上操作 (不要git init)

2. 项目经理创建本地仓库

   1. ```
      git init
      ```

   2. ```
      #添加一个需要推送到的远程 Git 仓库，同时指定一个你可以轻松引用的别名
      git remote add <shortname> <url(HTTPS | SSH)>
      ```

   3. 将源码复制进来

   4. ```
      #修改用户名和邮箱
      git config user.name "用户名"
      git config user.email "邮箱"
      ```

   5. ```
      git add
      git commit
      ```

3. 项目经理推送本地仓库到远程仓库

   1. ```
      git push 别名 分支名 (推完后会附带生成远程跟踪分支)
      ```

4. 项目经理邀请成员 & 成员接受邀请

   1. github上操作

5. 成员克隆远程仓库

   1. ```
      git clone 仓库地址(HTTPS | SSH)
      #在本地生成.git 文件 默认为远程仓库起了别名 origin 默认主分支有对应的远程跟踪分支
      #只有在克隆的时候 本地分支master 和 远程跟踪分支/master 有同步关系的
      
      git push & git fetch 是远程跟踪分支 和 远程分支之间数据的同步
      只有在同步的时候, 本地分支才和远程跟踪分支有同步关系 
      即 push fetch 操作 远程跟踪分支/master 的更新会同步到 本地分支master
      ```

6. 成员工作

   1. 修改

   2. ```
      git add
      git commit -m ""
      ```

   3. ```
      git push 别名 分支 (push会附带生成被push到远程仓库的分支的远程跟踪分支)
      #例: git push origin(远程仓库别名) dev(推送的分支名) 
      #推送完以后会生成 origin/dev远程跟踪分支
      ```

7. 项目经理更新修改

   1. ```
      git fetch 别名(将修改同步到远程跟踪分支上)
      git merge 远程跟踪分支
      ```



## 三个分支

### 远程分支

#### 删除远程分支

```
删除远程分支
git push origin --delete 远程分支

列出仍在远程跟踪但是远程已被删除的无用分支
git remote prune origin --dry-run

清除上面命令列出来的远程跟踪
git remote prune origin
```



### 远程跟踪分支

### 本地分支

1. 正常的数据推送 和 拉取步骤
   1. 确保本地分支已经跟踪了远程跟踪分支
   2. 拉取数据: git pull
   3. 上传数据: git push

#### 本地分支跟踪远程跟踪分支

1. 当克隆的时候, 会自动生成一个master本地分支 ,(已经跟踪了对应的 远程跟踪分支 )

2. 在新建其他分支时, 可以指定想要 跟踪的 远程跟踪分支

   1. ```
      git checkout -b 本地分支名 远程跟踪分支名
      git checkout --track 远程跟踪分支名
      ```

   2. 将一个已经存在的本地分支 改成 一个跟踪分支

      1. ```
         git branch -u 远程跟踪分支名
         ```

      2. ```
         git branch -vv 查看当前跟踪分支的情况
         ```

      3. 



## 冲突

1. git本地操作会不会有冲突?
   1. 典型合并的时候
2. git远程协作的时候, 会不会有冲突
   1. push
   2. pull





## pull request 流程





# 总结

## 底层命令

### Git对象

```
git hash-object -w fileUrl 生成一个key(hash值):val(压缩后的文件内容)键值对存到.git/objects
```

### tree对象

```
#往暂存区添加一条记录(让git对象对应上文件名) 存到.git/index
git update-index --add --cacheinfo 10064 对象的hash 对应文件名 
git write-tree 生成树对象
```

### commit对象

```
echo "注释信息" | git commit-tree tree对象hash  生成一个提交对象存到.git/objects
```

### 对以上对象的查询

```
git cat-file -p hash		查看对象的内容
git cat-file -t hash		查看对象的类型 (git, tree, commit)
```

### 查看 暂存区

```
git ls-files -s
```



## 高层命令

### 初始化配置

```
git config --global user.name "用户名"
git config --global user.email "邮箱"
git config --list	查看所有配置信息
```

### 初始化 仓库

```
git init
```

### C(新增)

```
在工作目录中新增文件
git status
git add ./
git commit -m "注释信息"
```

### U(修改)

```
在工作目录中修改文件
git status
git add ./
git commit -m "注释信息"
```

### D(删除 & 重命名)

```
git mv 老文件 新文件 (重命名)
git rm 要删除的文件名
git status
git commit -m "注释信息"
```

### R(查询)

```
git status	  查看工作目录文件的状态(已跟踪(已提交,已暂存,已修改), 未跟踪)
git diff			查看未暂存的修改
git diff --staged	查看未提交的修改
git log --oneline	查看提交记录

```



## 分支

### Git分支的本质

1. 分支的本质就是一个提交对象,
   1. 是一个没有后缀名的文件
   2. 里面的内容是提交对象所对应的hash
   3.  分支都会被HEAD所引用(HEAD一个时刻只会指向一个分支)
   4. 当我们有新的提交的时候, HEAD会携带当前持有的分支往前移动
2. HEAD
   1. 是一个指针 它默认指向master分支, 切换分支其实就是让HEAD指向不同的分支
   2. 每次有新的提交, HEAD都会带着当前指向的分支一起向前移动

```
git log --oneline --decorate --graph --all	查看整个项目的分支历史
git reflog	查看完整的log 
git branch	查看分支列表
git branch -v	查看分支最新的提交
git branch 分支名	在当前提交对象上创建新的分支
git branch 分支名 commitHash	在指定的提交对象上创建新的分支
git checkout 分支名	切换分支
git branch -d 分支名	删除空的分支,已经被合并的分支
git branch -D 分支名	强制删除分支

```



### 分支相关操作

```
创建分支: 		git branch branchname
切换分支: 		git checkout branchname
创建&切换分支: 	git checkout -b branchname
版本穿梭(时光机): git branch branchname commithash
普通删除分支: 	git branch -d branchname
强制删除分支: 	git branch -D branchname
合并分支: 		git merge branchname
	快进合并 --> 不会产生冲突
	典型合并 --> 有机会产生冲突
	解决冲突 --> 打开冲突的文件 进行修改 add commit

查看分支列表:		git branch 
查看合并到当前分支的分支列表:	git branch --merged
	一旦出现在这个列表中就应该删除
查看没有合并到当前分支的分支列表: git branch --no-merged
	一旦出现在这个列表中就应该观察一下是否需要合并
```



### Git分支的注意点

1. 在切换的时候, 一定要保证当前分支是干净的
   1. 允许切换分支:
      1. 分支上所有的内容处于	已提交状态
      2. 避免
         1. 分支上的内容是初始化创建  处于未跟踪状态(未 add commit)
         2. 分支上的内容是初始化创建 第一次 处于已暂存状态(add 未 commit)
   2. 不允许切分支
      1. 分支上所有的内容处于 已修改的状态 或 第二次以后的已暂存状态





### Git 存储

1. 在分支上的工作做到一半时 如果有切换分支的需求, 我们应该将现有的工作存储起来

   1. ```
      git stash: 会将当前分支上的工作推到一个栈中
      ```

   2. 分支切换 进行其他工作 完成其他工作后 切回原分支

   3. ```
      git stash apply : 将栈顶的工作内容还原, 不让任何内容出栈
      git stash drop : 取出栈顶的工作内容后 就应该将其删除(出栈)
      git stash pop : git stash apply + git stash drop
      git stash list : 查看存储
      ```



### 后悔药

```
撤销工作目录的修改:	git restore filename
撤销暂存区的修改:	 git restore --staged filename
撤销提交:			git commit --amend
```

### reset三部曲

```
HEAD~
当前HAED前一个提交对象
```

```
git reset --soft commithash(HEAD~) --> 用commithash的内容重置HEAD内容
git reset --mixed commithash(HEAD~) --> 用commithash的内容重置HEAD内容 重置暂存区
git reset --hard commithash(HEAD~) --> 用commithash的内容重置HEAD内容 重置暂存区 重置工作目录

```

### 路径reset

```
所有的路径reset都要省略第一步
	第一步是重置HEAD内容 我们知道HEAD本质指向一个分支 分支的本质是一个提交对象
	提交对象 指向一个树对象 树对象又很有可能指向多个git对象 一个git对象代表一个文件
	HEAD可以代表一系列文件的状态
	
git reset [--mixed] commithash filename
	用commithash中filename的内容重置暂存区
```

### checkout深入理解

```
git checkout branchname	根 git reset --hard commithash 特别想
	共同点
		都需要重置HEAD 暂存区  工作目录
	不同点
		checkout对工作目录是安全的  reset --hard是强制覆盖
		checkout动HEAD时不会带着分支走而是切换分支
		reset --hard是带着分支走
checkout + 路径
	git checkout commithash filename
		重置暂存区
		重置工作目录
	git checkout -- filename
		重置工作目录
```



## 远程协作

```
第一步: 项目经理创建一个空的远程仓库
第二步: 项目经理创建一个待推送的本地仓库
第三步: 为远程仓库配别名  配完用户名 邮箱
第四步: 在本地仓库中初始化代码 提交代码
第五步: 推送
第六步: 邀请成员
第七步: 成员克隆远程仓库
第八步: 成员做出修改
第九步: 成员推送自己的修改
第十步: 项目经理拉取成员的修改
```





