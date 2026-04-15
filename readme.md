将本地仓库推送到多个远程仓库配置的 demo 仓库。

使用 ssh 协议。

## 方法1：设置一个别名推送所有仓库

### 添加远程仓库：

```Bash
git remote add origin gitee1:jmsword/git_mul_remote_ssh.git
git remote add github github3:jxsword/git_mul_remote_ssh.git
```

查看配置结果：

```bash
$ git remote -v
github  github3:jxsword/git_mul_remote_ssh.git (fetch)
github  github3:jxsword/git_mul_remote_ssh.git (push)
origin  gitee1:jmsword/git_mul_remote_ssh.git (fetch)
origin  gitee1:jmsword/git_mul_remote_ssh.git (push)
```

### 创建推送所有仓库的别名

```Bash
# 首次推送
git config alias.pushallf '!git push -u origin main && git push -u github main'
# 后续推送
git config alias.pushall '!git push origin && git push github'
```

查看别名：

```bash
$ git config --list | grep alias
alias.aliases=config --get-regexp ^alias\.
alias.pushall=!git push origin && git push github
alias.pushallf=!git push -u origin main && git push -u github main
```

### 使用别名推送

```bash
# 首次推送
git pushallf
# 后续推送
git pushall
```

执行示例：

```bash
ysm@oyypc MINGW64 /e/IT_NOTE/git_mul_remote_ssh (main)
$ git pushallf
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 211 bytes | 211.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Powered by GITEE.COM [1.1.23]
remote: Set trace flag 684fc2ab
To gitee1:jmsword/git_mul_remote_ssh.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 211 bytes | 211.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To github3:jxsword/git_mul_remote_ssh.git
 * [new branch]      main -> main
branch 'main' set up to track 'github/main'.
```

## 方法2：使用 `git remote set-url --add` 为同一远程别名添加多个推送地址

使用 `git remote set-url --add` 为同一远程别名添加多个推送地址，然后使用 `git push --all origin` 推送到所有地址。

### 1. 使用 `git remote set-url --add` 为同一远程别名添加多个推送地址

删除方法1操作时添加的 remote 仓库。

```bash
# 删除所有: 包含 origin 和 github
git remote | xargs -L1 git remote remove

# 只删除 github (实际上只删除 github 即可)
git remote remove github
```

```Bash
# 上述步骤，若删除了所有的 remote 仓库，需要重新添加 origin
git remote add origin gitee1:jmsword/git_mul_remote_ssh.git

# 为 origin 添加第一个推送地址（默认的推送地址，和 remote add origin <url> 中的地址相同）
# 这一步是必需的，否则首次执行 set-url --add --push origin <url2> 时，会替换默认的推送地址，
#   导致推送 url 列表中将不包含默认的推送。
git remote set-url --add --push origin gitee1:jmsword/git_mul_remote_ssh.git

# 为 origin 添加第二个推送地址
git remote set-url --add --push origin github3:jxsword/git_mul_remote_ssh.git

# 如果需要，可以继续添加更多推送地址
git remote set-url --add --push origin <url>
```

### 2. 查看配置结果

```Bash
# 查看远程仓库详细信息
$ git remote -v
origin  gitee1:jmsword/git_mul_remote_ssh.git (fetch)
origin  gitee1:jmsword/git_mul_remote_ssh.git (push)
origin  github3:jxsword/git_mul_remote_ssh.git (push)
```

### 3. 使用 `git push --all origin` 推送到所有地址

```Bash
# 一次性推送到所有配置的推送地址
git push --all origin

# 也可以推送到特定分支
git push origin main

# 推送所有分支和标签
git push --all origin
git push --tags origin
```

## ssh 客户端配置

`~/.ssh/config`

```
Host gitee1
    HostName gitee.com
    User git
    IdentityFile ~/.ssh/id_ed25519_gitee
    IdentitiesOnly yes

Host github3
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github3
    IdentitiesOnly yes
    # Port 2222

Host txserver
    HostName 101.43.17.214
    User sy
    Port 2967
    IdentityFile ~/.ssh/id_rsa_tencent_cloud
```
