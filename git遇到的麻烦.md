### 问题1

问题描述：git登录的是其他账号，怎么切换成另一个账号（除了SSH，另一种git会弹出github登录框的登陆方式）



删除windows凭据，在git push的时候git会让你重新登录github授权。

（控制面板->用户账户->凭据管理器）

### 问题2

在upstream main branch上fork后开了 feat-xxx branch ，然后在 feat-xxx 修改了 a.md 文件，然后在upstream main branch要merge feat-xxx branch时，upstream main branch中已经merge了很多commit（不包含对a.md的变动），而期间feat-xxx又没有保持sync fork，这样合并会冲突吗？



在这种前提下，此时的upstream main branch的base已经和feat-xxx的base不一样了，那么实际上merge的时候git是怎么解决这个情况的？还是说只考虑是否加入commit的部分。



解：

冲突的本质是同时修改同一个文件的同个位置，在git中是看base上的commit是否冲突，不会发生冲突，因为upstream main branch merge的commit不包含对a.md的变动，在a.md的维度上它们的base是一样的，merge branch的时候只对比两个branch的base与base之后的commit，考虑是否加入commit的部分。

#### 猜想

结论：我觉得git应该是会同时考虑commit的内容和base，来判断它们是否属于同时修改的冲突。

推导：

联想到一种情况：为了将pr的多个commit整理为1个，在使用rebase整理commit后push也造成冲突，因为rebase后相当于base变成相对早期的commit（这里称为A），然后在原先的分支上的A之后的commit的内容 与 整理后的commit内容是一样的，所以会冲突。

上面的冲突本质上等同于两个人同时修改一个文件：

m1与m2修改了同个文件，而m2的commit先被并入到主分支，落后的m1在并入主分支时就发生了冲突。我觉得此时git是根据它们的base与commit内容来判断先后的，在这种情况下就意味着m1与m2同时修改了同个文件。

### 问题3

本地代码是databend上游代码，此时我想切换到自己fork的远程repo的patch-5 branch，敲一些代码，然后push到远程repo的patch-5分支，以此来更新我pr中的commit，一种最粗暴的方法是：拉取一份新的远程repo的patch-5 branch分支。可是这其中的代码大多都跟本地代码重复，感觉冗余浪费空间且浪费时间，有没有一种优雅的办法将本地代码同步为自己fork的远程repo的patch-5 branch代码？



1、添加上游remote的fetch源。

2、方式一：git pull 上游源对应的分支。方式二：git fetch上游源对应的分支后，git merge该分支。

### 问题4

git设置代理时，为什么只设置 `git config --global https.proxy http://127.0.0.1:7890` 没有走代理，而只设置 ` git config --global http.proxy http://127.0.0.1:7890` 就走了代理？

好像只设置 `git config --global https.proxy https://127.0.0.1:7890` 是有走代理的



### 问题5

怎么在服务器上的Git使用本地生成的GPG key呢？

解：

1. 找到本地 ~/.gnupg 文件夹，将其发送到服务器的 ~目录中，方式一：使用xftp，方式二：使用wsl发送，命令：`scp -r ~/.gnupg <user>@<ip>:~` （若没有-r会报错）

2. 检查是否迁移成功 `gpg --list-keys`

3. git配置gpg：`git config --global user.signingkey '<gpg key id>'` (与github上配置的keyid一致即可，或者直接复制本地的配置)

4. 若配置后commit出现下面的错误：

   ```bash
   git commit -s -S -m 'docs: add databend info'
   error: gpg failed to sign the data
   fatal: failed to write commit object
   ```

   根据这个方法配置后即可：[解决GPG签名失败的问题 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/97984430)

   ```bash
   vim ~/.zshrc
   #在最后面加入
   export GPG_TTY=$(tty)
   #重新读取
   source ~/.zshrc
   ```

   ### 问题6
   
   git push的时候报错
   
   ```
   Missing or invalid credentials.
   Error: connect ECONNREFUSED /run/user/1007/vscode-git-ab4b33d46a.sock
       at PipeConnectWrap.afterConnect [as oncomplete] (node:net:1157:16) {
     errno: -111,
     code: 'ECONNREFUSED',
     syscall: 'connect',
     address: '/run/user/1007/vsThere isn’t anything to compare.
   sqlancer:master and hanyisong:update-ci are identical.code-git-ab4b33d46a.sock'
   }
   Missing or invalid credentials.
   Error: connect ECONNREFUSED /run/user/1007/vscode-git-ab4b33d46a.sock
       at PipeConnectWrap.afterConnect [as oncomplete] (node:net:1157:16) {
     errno: -111,
     code: 'ECONNREFUSED',
     syscall: 'connect',
     address: '/run/user/1007/vscode-git-ab4b33d46a.sock'
   }
   remote: No anonymous write access.
   fatal: Authentication failed for 'https://github.com/hanyisong/databend.git/'
   ```
   
   理论上可以解决（未实践）：[Git 推送：缺少凭据或凭据无效。致命：“https://github.com/username/repo.git”的身份验证失败 - 堆栈溢出 (stackoverflow.com)](https://stackoverflow.com/questions/62860280/git-push-missing-or-invalid-credentials-fatal-authentication-failed-for-http)

本质上是vscode远程开发时的影响，重新开一个远程窗口就行了，或者按照上面那个方法试试。

