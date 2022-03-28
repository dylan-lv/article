# 将现有项目提交到空仓库

1、在本地项目目录下初始化 git 仓库

```bash
git init
```

2、将本地项目下工作区的所有文件添加到 git 版本库的暂存区中

```bash
git add .
```

3、将暂存区的文件进行提交到版本库

```bash
git commit -m "{描述}"
```

4、设置本地项目版本库的远程仓库地址

```bash
git remote add origin https://github.com/xxx/xxx.git
```

5、将本地版本推送到 git 上

```bash
git push origin master
```

























