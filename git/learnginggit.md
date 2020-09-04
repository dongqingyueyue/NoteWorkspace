#git实践小结
## 情景一
 在本地的工作目录中,如果已经commit了,但是发现commit中有些不妥的地方,或者有两个文件不想提交怎么办?
方案一:如果是commit中的注释写错了,想重新提交的话,只需要:   
<center><font color = red>git commit --amend</center> </font>  
此时会进入默认的vim编辑器,修改注释完毕后保存就好了.  
方案二:git reset 命令和参数  
git reset --soft HEAD^ &nbsp; 不删除工作区间改动的代码,撤销commit操作,但不撤销git add操作  
git reset --mixed HEAD^ &nbsp; 不删除工作区间改动的代码,撤销commit操作,同时也撤销git add的操作  
git reset --hard HEAD^ &nbsp; 删除工作区间改动的代码,撤销commit操作和git add的操作  

##情景二
commit push 代码已经更新到远程仓库,想取消修改

对于已经把代码push到线上仓库,你回退本地代码其实也想同时回退线上代码,回滚到某个指定的版本,线上,线下代码保持一致.你要用到下面的命令

git revert commitid

revert 之后你的本地代码会回滚到指定的历史版本,这时你再 git push 既可以把线上的代码更新。

 

注意：git revert是用一次新的commit来回滚之前的commit，git reset是直接删除指定的commit，看似达到的效果是一样的,其实完全不同。

第一:上面我们说的如果你已经push到线上代码库, reset 删除指定commit以后,你git push可能导致一大堆冲突.但是revert 并不会.
第二:如果在日后现有分支和历史分支需要合并的时候,reset 恢复部分的代码依然会出现在历史分支里.但是revert 方向提交的commit 并不会出现在历史分支里.
第三:reset 是在正常的commit历史中,删除了指定的commit,这时 HEAD 是向后移动了,而 revert 是在正常的commit历史中再commit一次,只不过是反向提交,他的 HEAD 是一直向前的.  



##情景三
  如果文件不小心git add将文件提交到暂存区域,想取消这个操作   
使用git status给出的建议:  
git reset HEAD files...取消暂存  
如果想将i修改的文件保持和版本中一模一样的版本,可以使用 git checkout file进行操作  

##情景四
git diff last current进行对比,位置不能对调,如果对调的话执行命令后,显示不同地方+和-号会有改变;  

比较本地当前分支与远程分支的不同:git diff master origin/master  

##配置SSH-key并将其加入github遇到的一个问题
本地已经有两个ssh-key公钥和私钥,用于配置其他服务器上的ssh链接服务.当自己申请了一个github账号,新建了一个仓库.为了避免繁琐额输入账号和密码的过程.经过一系列尝试,将公钥加入github之后,使用ssh -T git@github.com,返回的结果是:Hi guy! You've successfully authenticated, but GitHub does not provide shell access.


解决方法:git remote set-url origin git@github.com:lut/EvolutionApp.git   
[参考](https://stackoverflow.com/questions/26953071/github-authentication-failed-github-does-not-provide-shell-access)
## 情景五 
学会使用.gitignore文件来过滤不需要提交和上传的文件  
大量与项目无关的文件全推到远程仓库上，同步的时候会非常慢，且跟编辑器相关的一些配置推上去之后，别人更新也会受其影响。所以，我们使用该文件，对不必要的文件进行忽略，使其不被git追踪。

 一把情况下，.gitignore文件，在项目一开始创建的时候就创建，并推送到远程服务器上。这样大家初次同步项目的时候，就是用到该文件，避免以后，团队成员把与项目无关的文件，传到远程服务器上。
```
*.log           #表示忽略项目中所有以.log结尾的文件
123?.log        #表示忽略项目中所有以123加任意字符的文件
/error.log      #表示忽略项目中根目录中的error.log 这个文件
src/main/test/* #表示忽略/src/main/test/目录下的所有文件
**/java/        #匹配所有java目录下的所有文件
!/error.log     #表示在之前的匹配规则下，被命中的文件，可以使用!对前面的规则进行否定
```

对于已经提交到远程或本地仓库的文件，.gitignore配置之后不会生效。我们必须先删除本地暂存区里的文件，之后在加上.gitignore 文件，最后再把变更提交到远程仓库上。
```
git rm --cached 文件名     #从暂存区删除某个文件
git rm -rf --cached 文件夹 #表示递归删除暂存区该文件夹的所有东西
```

[add a blog link](https://www.cnblogs.com/qdhxhz/p/9763546.html)  

## 情景六
git 中几个重要的概念
HEAD---当前commit的引用,指的是当前工作目录所对应的commit
事实上,当使用checkout reset等指令手动指定改变当前commit的时候,HEAD也会一起跟过去.

branch--

当有人使用 git clone 时，除了从远程仓库把 .git 这个仓库目录下载到工作目录中，还会 checkout （签出） master（checkout 的意思就是把某个 commit 作为当前 commit，把 HEAD 移动过去，并把工作目录的文件内容替换成这个 commit 所对应的内容
