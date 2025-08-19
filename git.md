# git config 
```dotnetcli
git config --list 
git config --global user.name
git config --global user.email

______________________________
git调试模式
 GIT_CURL_VERBOSE=1 GIT_TRACE=1 git clone https://github.com/xxx.git
problem :== Info: Couldn't find host github.com in the .netrc file; using defaults

vi ~/.gitconfig
[credential]
    helper = store

vi ~/.netrc
machine github.com
    login your-user-name
    password your-personal-access-token
登录GitHub：在浏览器中打开GitHub，然后登录您的账户。

创建Personal Access Token：

在页面右上角，点击您的头像，然后点击Settings（设置）。
在左侧边栏中，点击Developer settings（开发人员设置）。
在左侧边栏中，找到并单击Personal access tokens（个人访问令牌）。
单击Generate new token（生成新令牌）按钮。
为您的令牌指定一个描述性名称，以便于识别。
选择到期日期。您可以选择默认值或使用日历选择器设置令牌的过期日期。
选择您要授予此令牌的范围或权限。根据您的需求选择适当的选项，例如repo（存储库）
```
# git 创建切换分支
```dotnetcli
#创建新分支aaa
git branch aaa 
#切换到aaa分支
git checkout aaa 
#查看当前分支
git branch   
```

# git 列出所有作者

```dotnetcli
#git 列出所有作者并去重
git log --pretty=format:"%an" main | sort -u

#git 列出所有作者邮箱并去重#main是分支
git log --pretty=format:"%ae" main | sort -u

列出所有贡献者，并按照贡献数量排序
git shortlog -sn --all

查看所有作者
git log | grep Author: | sort | uniq
```
# Git中只克隆一个特定分支

```dotnetcli
git clone -b main --single-branch <repository URL>

#main 分支名
```

# 批量删除远程不活跃的分支

```dotnetcli
async function deleteStaleBranches(delay=500) {
    var stale_branches = document.getElementsByClassName('js-branch-delete-button');
    for (var i = 0; i < stale_branches.length; i++)
    {
        stale_branches.item(i).click();
        await new Promise(r => setTimeout(r, delay));
    }
}

(() => { deleteStaleBranches(500); })();
```
![delete branch](/img/delete_branch.png)

# git reset 
```dotnetcli
Soft: 回到选择的版本，但这个版本之后的所有提交(包括工作区未提交的改动)都会保存; 
Mixed: 退回到选择的版本，本地仓库也会变为这一版本的内容，但工作区不会变; 
Hard: 彻底回退到选择的版本，本地仓库也会变为这一版本的内容, 工作区所有改动都会丢失;
```
# 远程分支强制覆盖本地
```dotnetcli
git pull --force  <远程主机名> <远程分支名>:<本地分支名>
git pull --force origin master:master

方法二
$     git fetch --all
$ git reset --hard origin/main
$ git pull origin main

方法二做成别名
git config --global alias.tbmain '!git fetch --all && git reset --hard origin/main && git pull origin main'
git config --global alias.tbmaster '!git fetch --all && git reset --hard origin/master && git pull origin master'
```


# git撤销、还原、放弃本地文件修改
```dotnetcli
1. 未使用git add 缓存代码
git checkout -- filepathname 
放弃所有文件修改 git checkout 
git checkout .
此命令用来放弃掉所有还没有加入到缓存区（就是 git add 命令）的修改：内容修改与整个文件删除
不会删除掉刚新建的文件。因为刚新建的文件还没已有加入到 git 的管理系统中。所以对于git是未知的。自己手动删除就好了。


2. 已使用git add 缓存代码，未使用git commit
git reset HEAD
此命令用来清除 git 对于文件修改的缓存。相当于撤销 git add 命令所在的工作。
使用本命令后，本地的修改并不会消失，而是回到了第一步1. 未使用git add 缓存代码，继续使用git checkout -- filepathname，就可以放弃本地修改

3.已经用 git commit 提交了代码
git reset --hard HEAD^
回退到任意版本git reset --hard commitid，使用git log命令查看git提交历史和commitid
```

# 清除 git 所有历史提交记录方案
```dotnetcli
1.创建新分支
语法：git checkout --orphan <new_branch>
2.添加所有文件
git add .
3.commit代码
git commit -m "自定义提交说明"
4.删除原来的主分支(master)
git branch -D master
5.把当前分支重命名为master
git branch -m master
6.最后把代码推送到远程仓库
git push -f origin master
注意： 有些仓库有 master 分支保护，不允许强制 push，需要在远程仓库项目里暂时把项目保护关掉才能推送。
推送前 需要使用 git remote -v 查看关联的远程仓库的信息（主要是远程库的别名）。虽然远程库的别名默认是 origin ,但你可能设置过其他的别名（而非 origin）.
推送前，有的情况需要设置：git branch --set-upstream-to=origin/master master
7.从远程库拉取更新代码(测试)
git pull
8.确定清除历史记录的结果
git log --pretty=oneline
# 列出所有本地分支
git branch
# 列出所有远程分支
git branch -r
# 列出所有本地分支和远程分支
git branch -a
# 查看 tag 信息
# 查看本地标签
git tag
# 查看远程标签
git ls-remote --tags

9. 可登录远程仓库再次确认。
```

# 删除全部历史记录
```dotnetcli
把旧项目提交到git上，但是会有一些历史记录，这些历史记录中可能会有项目密码等敏感信息。
如何删除这些历史记录，形成一个全新的仓库，并且保持代码不变呢？

1.Checkout
   git checkout --orphan latest_branch
2. Add all the files
   git add -A
3. Commit the changes
   git commit -am "commit message"
4. Delete the branch
   git branch -D master
5.Rename the current branch to master
   git branch -m master
6.Finally, force update your repository
   git push -f origin master
```

# git 切换远程分支
```dotnetcli
$ git remote set-url origin https://github.com/digger-yu/trex-core.git
$ git remote set-url origin https://github.com/cisco-system-traffic-generator/trex-core.git

digger@Digger-Desktop MINGW64 /d/git/trex-core (master)
$ git remote -v

```

# gitignore不生效
```dotnetcli
gitignore中已经标明忽略的文件目录下的文件，git push的时候还会出现在push的目录中，或者用git status查看状态，想要忽略的文件还是显示被追踪状态。
原因是因为在git忽略目录中，新建的文件在git中会有缓存，如果某些文件已经被纳入了版本管理中，就算是在.gitignore中已经声明了忽略路径也是不起作用的，
这时候我们就应该先把本地缓存删除，然后再进行git的提交，这样就不会出现忽略的文件了。
解决方法: git清除本地缓存（改变成未track状态），然后再提交:
git rm -r --cached .
git add .
git commit -m 'update .gitignore'
git push -u origin master
```

# git rebase
```dotnetcli
git log --oneline --decorate
git rebase -i 1654b73
git rebase -i ec34056
git rebase -i cc13e0d
git rebase -i 5549f79
git push origin 5549f79:main --force  从本地分支强制推送到远程分支
git push origin main:main --force  从本地分支强制推送到远程分支
git push REMOTE-NAME LOCAL-BRANCH-NAME:REMOTE-BRANCH-NAME
```

# 添加/移除一个远程库
```dotnetcli
##添加一个远程库
git remote add origin https://github.com/digger-yu/dpef/dperf.git

$ git remote -v  查看远程库的详细信息
origin  https://github.com/digger-yu/dperf.git (fetch)
origin  https://github.com/digger-yu/dperf.git (push)

#移除远程库名字
git remove rm xxx  

#本地分支推送到远程版本库 不跟踪远程分支变化
git push origin 本地库 

#将本地分支推送到远程版本库  跟踪远程分支变化 
git push -u origin master  

## 从本地master分支推送到远程的origin分支
git push -u origin master 
```

# win10上git比较慢
```dotnetcli
git config --global core.preloadindex true
git config --global core.fscache true
git config --global gc.auto 256
```
# git 添加证书信任
```dotnetcli
git config user.signingKey 749C0243
git config commit.gpgSign true
``````
# git diff

```dotnetcli
git diff --color-words

grep -rn "enable" ./* --colour=auto
git commit -S -am "fix typo"
git push origin main:patch1

#查看某次提交修改的所有文件
git show --raw commit_id
```
# personal 
```dotnetcli
git config --list
git config --global user.name "digger yu"
git config --global user.email digger-yu@outlook.com
git config --global user.signingkey 93F04D48749C0243
git config --global alias.tbmain '!git fetch --all && git reset --hard origin/main && git pull origin main'
git config --global alias.tbmaster '!git fetch --all && git reset --hard origin/master && git pull origin master'

gpg --list-keys
gpg --import public-file.key
gpg --import private-file.key
导出
#gpg -a -o public.key --export 93F04D48749C0243
#gpg -a -o private.key --export-secret-keys 93F04D48749C0243

gpg --edit-key 93F04D48749C0243
trust
5
yes
curl https://github.com/web-flow.gpg | gpg --import
删除
gpg --delete-keys 4AEE18F83AFDEB23
gpg --sign-key B5690EEEBB952194

pip升级及更改源
python -m pip install --upgrade pip
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

如果用ssh的方式git clone出现需要输入密码
Enter passphrase for key '/c/Users/digger/.ssh/id_rsa':
可以将密码设置为空,以源密码为123456举例,之后就不需要输入密码了
$ ssh-keygen -p -P 123456 -N '' -f id_rsa
ssh-keygen -p [-P old_passphrase][-N new_passphrase] [-f keyfile]


```
# git 设置代理

```
设置全局代理
git config --global http.proxy socks5://127.0.0.1:1080   
git config --global https.proxy socks5://127.0.0.1:7897
取消全局代理
git config --global --unset http.proxy   
git config --global --unset https.proxy

设置对当前项目生效的代理
git config https.proxy https://proxy_host:proxy_port
取消项目代理
git config --unset https.proxy

只对github进行代理
git config --global http.https://github.com.proxy socks5://127.0.0.1:10808
git config --global https.https://github.com.proxy socks5://127.0.0.1:10808
```
# git 加速
```
 git config --global --list
#增加
 git config --global url."https://gitclone.com/".insteadOf https://
#删除
git config --global --unset url."https://gitclone.com/".insteadOf
```








