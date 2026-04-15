## 关于首次执行 `set-url --add --push origin` 会替换默认推送地址的问题分析和解决方案

```bash
$ git remote add origin gitee1:jmsword/git_mul_remote_ssh.git

ysm@oyypc MINGW64 /e/IT_NOTE/git_mul_remote_ssh (main)
$ git remote -v
origin  gitee1:jmsword/git_mul_remote_ssh.git (fetch)
origin  gitee1:jmsword/git_mul_remote_ssh.git (push)

ysm@oyypc MINGW64 /e/IT_NOTE/git_mul_remote_ssh (main)
$ git remote set-url --add --push origin github3:jxsword/git_mul_remote_ssh.git

ysm@oyypc MINGW64 /e/IT_NOTE/git_mul_remote_ssh (main)
$ git remote -v
# 未显示 gitee1:jmsword/git_mul_remote_ssh.git (push)
origin  gitee1:jmsword/git_mul_remote_ssh.git (fetch)
origin  github3:jxsword/git_mul_remote_ssh.git (push)

# 查看 url 进一步验证，只输出新添加的
$ git remote get-url --all --push origin
github3:jxsword/git_mul_remote_ssh.git

# 查看 仓库 配置文件
$ cat .git/config
[core]
        repositoryformatversion = 0
        filemode = false
        bare = false
        logallrefupdates = true
        ignorecase = true
[alias]
        pushall = !git push origin && git push github
        pushallf = !git push -u origin main && git push -u github main
[remote "origin"]
        url = gitee1:jmsword/git_mul_remote_ssh.git
        fetch = +refs/heads/*:refs/remotes/origin/*
        pushurl = github3:jxsword/git_mul_remote_ssh.git
```

`git remote set-url --add --push origin github3:jxsword/git_mul_remote_ssh.git` 执行后 `origin` 默认的推送地址被替换为 `github3:jxsword/git_mul_remote_ssh.git`。导致只推送到 github 而未推送到 gitee。

但经过验证，首次执行后，后续再次执行添加新 url 时，不会替换默认的推送地址。而是额外添加新的推送地址。

为此，对于首次执行前的本地 git 仓库配置文件参数进行分析，如下：

```bash
ysm@oyypc MINGW64 /e/IT_NOTE/git_mul_remote_ssh (main)
# 删除所有 remote url
$ git remote | xargs -L1 git remote remove
$ git remote -v

$ git remote add origin gitee1:jmsword/git_mul_remote_ssh.git
$ git remote -v
origin  gitee1:jmsword/git_mul_remote_ssh.git (fetch)
origin  gitee1:jmsword/git_mul_remote_ssh.git (push)
$ git remote get-url --all --push origin
gitee1:jmsword/git_mul_remote_ssh.git
$ cat .git/config
[core]
        repositoryformatversion = 0
        filemode = false
        bare = false
        logallrefupdates = true
        ignorecase = true
[alias]
        pushall = !git push origin && git push github
        pushallf = !git push -u origin main && git push -u github main
[remote "origin"]
        url = gitee1:jmsword/git_mul_remote_ssh.git
        fetch = +refs/heads/*:refs/remotes/origin/*

ysm@oyypc MINGW64 /e/IT_NOTE/git_mul_remote_ssh (main)
$
```

通过上述输出可看到：

`git remote add origin gitee1:jmsword/git_mul_remote_ssh.git` 执行后并未添加 `pushurl` 配置参数，但是 `git remote get-url --all --push origin` 和 `git remote -v` 查询时均返回 `pushurl` 参数。

下面首次执行 `set-url --add --push` 时添加默认的推送地址，并添加第二个推送地址：

```bash
$ git remote set-url --add --push origin gitee1:jmsword/git_mul_remote_ssh.git

ysm@oyypc MINGW64 /e/IT_NOTE/git_mul_remote_ssh (main)
$ cat .git/config
[core]
        repositoryformatversion = 0
        filemode = false
        bare = false
        logallrefupdates = true
        ignorecase = true
[alias]
        pushall = !git push origin && git push github
        pushallf = !git push -u origin main && git push -u github main
[remote "origin"]
        url = gitee1:jmsword/git_mul_remote_ssh.git
        fetch = +refs/heads/*:refs/remotes/origin/*
        pushurl = gitee1:jmsword/git_mul_remote_ssh.git

ysm@oyypc MINGW64 /e/IT_NOTE/git_mul_remote_ssh (main)
$ git remote -v
origin  gitee1:jmsword/git_mul_remote_ssh.git (fetch)
origin  gitee1:jmsword/git_mul_remote_ssh.git (push)

ysm@oyypc MINGW64 /e/IT_NOTE/git_mul_remote_ssh (main)
$ git remote set-url --add --push origin github3:jxsword/git_mul_remote_ssh.git

ysm@oyypc MINGW64 /e/IT_NOTE/git_mul_remote_ssh (main)
$ git remote -v
origin  gitee1:jmsword/git_mul_remote_ssh.git (fetch)
origin  gitee1:jmsword/git_mul_remote_ssh.git (push)
origin  github3:jxsword/git_mul_remote_ssh.git (push)

ysm@oyypc MINGW64 /e/IT_NOTE/git_mul_remote_ssh (main)
$ cat .git/config
[core]
        repositoryformatversion = 0
        filemode = false
        bare = false
        logallrefupdates = true
        ignorecase = true
[alias]
        pushall = !git push origin && git push github
        pushallf = !git push -u origin main && git push -u github main
[remote "origin"]
        url = gitee1:jmsword/git_mul_remote_ssh.git
        fetch = +refs/heads/*:refs/remotes/origin/*
        pushurl = gitee1:jmsword/git_mul_remote_ssh.git
        pushurl = github3:jxsword/git_mul_remote_ssh.git

ysm@oyypc MINGW64 /e/IT_NOTE/git_mul_remote_ssh (main)
$
```

从上述执行可看到所有推送地址添加成功。

### 总结

#### 原因分析

从上面的实例可得到：

1. `git remote add origin <url>` 执行后 git 配置文件中并未添加 `pushurl` 参数，但`git remote` 相关命令执行查询时会返回 `pushurl` 参数，其原因是：当配置文件中无 `pushurl` 参数时，git 会使用 `url` 参数值作为 `pushurl` 参数值。
2. `git remote set-url --add --push origin <url2>` 首次执行时，若 git 配置文件中无 `pushurl` 参数，它执行后，git 配置文件中会添加 `pushurl` 参数，值为 `<url2>`。再次用 `git remote` 查询时，会返回 `pushurl` 参数的值，而非将 `url` 参数的值作为 `pushurl` 参数值返回。因此出现了替换默认推送地址的情况。
3. 后续执行`git remote set-url --add --push origin <urlk>`时，git 配置文件中会额外添加 `pushurl = <urlk>` ，增加新的推送地址。

#### 解决方案

鉴于以上情况，解决方案如下：

`git remote add origin <url>` 执行后，先使用 `git remote set-url --add --push origin <url>` 添加默认推送地址，再添加其他新的推送地址。
