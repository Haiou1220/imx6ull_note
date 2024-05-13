git push 的 -u 参数含义

Lakers2015

于 2020-12-21 12:09:09 发布

阅读量3w
 收藏 78

点赞数 42
分类专栏： Git 文章标签： git gitlab
版权

Git
专栏收录该内容
25 篇文章10 订阅
订阅专栏
问题场景：
一般将本地仓库推送到远程仓库的时候一般会使用 git push 命令。而作为新手，在网上看到一些教程有的会在 git push 的时候带上一个 -u 参数，而有的则没有。而推送的实际结果没有什么区别。就很好奇 -u 参数的作用到底是什么？

搜索了一番，综合了一些大家的说明和解析，总结记录一下。

参数解析：
首先对于 git push,有这样一段描述：

-u
–set-upstream

For every branch that is up to date or successfully pushed, add upstream (tracking) reference, used by argument-less git-pull(1) and other commands. For more information, see branch.<name>.merge in git-config(1).

在这个描述中，可以看到 -u 参数与下面这个变量相关

branch.<name>.merge

在git config中。对于这个变量又有描述如下：

Defines, together with branch.<name>.remote, the upstream branch for the given branch. It tells git fetch/git pull which branch to merge and can also affect git push (see push.default). When in branch <name>, it tells git fetch the default refspec to be marked for merging in FETCH_HEAD. The value is handled like the remote part of a refspec, and must match a ref which is fetched from the remote given by “branch.<name>.remote”. The merge information is used by git pull (which at first calls git fetch) to lookup the default branch for merging. Without this option, git pull defaults to merge the first refspec fetched. Specify multiple values to get an octopus merge. If you wish to setup git pull so that it merges into <name> from another branch in the local repository, you can point branch.<name>.merge to the desired branch, and use the special setting . (a period) for branch.<name>.remote.

主要就是说branch.<name>.merge与branch.<name>.remote一起定义给定分支的上游分支（upstream）。它告诉git fetch/git pull要合并哪个分支，还可以影响git push.

而upstream是指其他人将从中获取的主要存储库，例如您的GitHub存储库。-u选项自动为您设置上游，将您的仓库链接到一个中央仓库。这样，将来Git会“知道”您要推送到的位置以及您要从哪里提取的信息，因此您可以使用git pull或git push不使用参数。 当您git pull从分支进行操作而未指定源远程或分支时，git会查看 branch.<name>.merge 设置以了解从何处提取。而正是git push -u 命令为您要推送的分支设置此信息。

至此，简单来说，带上-u 参数其实就相当于记录了push到远端分支的默认值，这样当下次我们还想要继续push的这个远端分支的时候推送命令就可以简写成git push即可。

示例展示：
下面展示一个示例来说明这一点。

andy@AndyMacBookPro:/usr/local/github/andy/php-examples$ git pull
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
See git-pull(1) for details.

git pull <remote> <branch>

If you wish to set tracking information for this branch you can do so with:

git branch --set-upstream-to=origin/<branch> test

这个就是如果你之前未使用 -u 参数，后面省略了想要pull的分支参数而产生的结果。pull 因为没有track for the current branch. 所以他不知道你要从哪里pull，所以这也就是 -u 参数的意义，指定trach branch。

其实你可以在指定完-u之后，去.git/config看GIT配置文件，可以看到下面有了branch "test"的分支的记录：

[branch "master"] 
 	remote = origin 	
	merge = refs/heads/master 
[branch "test"] 	
	remote = origin 	
	merge = refs/heads/test
1
2
3
4
5
6
这样git才能知道当前test下的remote和merge的信息，如果你在git push的时候没有带入-u参数，那么config中就不会有branch "test"这一项。

 [branch "master"]
    remote = origin
    merge = refs/heads/master 
1
2
3
配置说明，这告诉Git 2件事：

当您在master分支上时，默认的遥控器是origin。
在git pullmaster分支上使用时（未指定任何远程和分支），请使用默认的remote（源）并合并来自remote master分支的更改。
配置修改
您可以手动去.git/config修改GIT配置文件内容，也可以使用命令行设置这些选项。

 $ git config branch.master.remote origin
 $ git config branch.master.merge refs/heads/master
1
2
如果使用命令进行配置，它将有一定的纠错能力。比如您键入了一个不存在的分支或者您没有执行git remote add 操作。在较新的git中，希望您使用 git branch --set-upstream-to=origin/master master

其实，执行添加了-u 参数的命令 git push -u origin master就相当于是执行了

git push origin master 和
git branch --set-upstream master origin/master。

所以，在进行推送代码到远端分支，且之后希望持续向该远程分支推送，则可以在推送命令中添加 -u 参数，简化之后的推送命令输入。

说明1：
Git can set a specific branch in the remote repository as the default “upstream” branch for that particular branch. For example, if you clone an existing repository, GIT by default associates your master branch with the branch in the origin repository from which you want to clone. This means that git can provide useful default values, such as only git pull is used when it is opened, and master does not have to specify the repository and branch from which to get and merge.

However, if you do not want to clone from an existing repository, but you want to create a new origin remote control, which represents the newly created GitHub library, you must manually notify git’s associated master and master about the new origin library. In - U and git push “as well as pulling” You only need to do it once to record the association in .git / config.

文章知识点与官方知识档案匹配，可进一步学习相关知识
————————————————

                            版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。
                        
原文链接：https://blog.csdn.net/Lakers2015/article/details/111318801