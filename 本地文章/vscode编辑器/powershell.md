### 为vscode的powershell添加别名

执行 `echo $PROFILE` 命令，确定新建文件的名称和位置

![vscode-1](.\assets\vscode-1.png)

新建文件：C:\Users\41770\Documents\WindowsPowerShell

我这里的步骤（不一定和你一致）

- Users：用户
- Documents：文档
- 在 `Documents` 文件夹下新建 `WindowsPowerShell` 文件夹
-  继续新建 `Microsoft.PowerShell_profile.ps1` 文件

修改内容：

```shell
# 进入mini-vue
function cdvue {cd F:\github\mini-vue\}
# git使用代理提交
function git-p {
	git config --global http.proxy 127.0.0.1:1080
	git push
	git config --global --unset http.proxy
}
```

重启 vscode 即可
