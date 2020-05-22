常用命令：

1. 下载：
```bash
git clone URL;
```

切换远程分支：
```bash
git checkout -t origin/dev
```
2. 打补丁包
如果没有新加的文件，直接用
```bash
git diff > ~/1.patch
```
如果有新增文件，用git add添加后，git diff的参数加上--cached，然后与HEAD版本比较差异：
```bash
git diff --cached HEAD > ~/1.patch
```
应用补丁包：
```![](assets/git-53cea446.reg)bash
git apply ~/1.patch
```
basskdfj
