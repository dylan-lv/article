gti打tag

通常在发布版本的时候给 `git` 打一个 `tag`，`tag`可以记录版本的 `commit`，方便后面回溯

### 列出已有 `tag`

```bash
git tag
```

加上 `-l` 命令可以过滤

```bash
git tag -l "v0.1.*"
```



### 新建tag

使用`git tag`命令跟上tag名字，直接创建一个tag。

```bash
git tag v1.0.0
```

还可以加上`-a`参数来创建一个带备注的tag，备注信息由`-m`指定。如果你未传入`-m`则创建过程系统会自动为你打开编辑器让你填写备注信息。

```bash
git tag -a tagName -m "my tag"
```



### 查看tag详细信息

`git show`命令可以查看tag的详细信息，包括commit号等。

```bash
git show tagName
```

tag最重要的是有git commit号，后期我们可以根据这个commit号来回溯代码。



### 给指定的某个commit号加tag

打tag不必要在head之上，也可在之前的版本上打，这需要你知道某个提交对象的校验和（通过git log获取，取校验和的前几位数字即可）。

```bash
git tag -a v1.2 9fceb02 -m "my tag"
```



### 将tag同步到远程服务器

同提交代码后，使用`git push`来推送到远程服务器一样，`tag`也需要进行推送才能到远端服务器。
使用`git push origin [tagName]`推送单个分支。

```bash
git push origin v1.0
```

推送本地所有tag，使用`git push origin --tags`。



### 切换到某个tag

跟分支一样，可以直接切换到某个tag去。这个时候不位于任何分支，处于游离状态，可以考虑基于这个tag创建一个分支。

```bash
git checkout v1.1.0
```



### 删除某个 tag

- 本地删除

```bash
git tag -d v0.1.2 
```

- 远程删除

`git push origin :refs/tags/<tagName>`

```bash
git push origin :refs/tags/v0.1.2
```

