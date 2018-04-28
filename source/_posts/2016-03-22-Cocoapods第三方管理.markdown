---
title:  "Cocoapods第三方管理"
date:   2016-03-22 16:20:00 +8:00
categories: 
- mac
tags: 
- 工具
---
## 安装Cocoapods

打开终端, 在终端输入指令

系统 `10.11` 以下

```
$ sudo gem install Cocoapods
```

系统 `10.11` 以上

```
$ sudo gem install Cocoapods -n /usr/local/bin
```

> 在系统 `10.11` 以上的 `mac` 版本中, 苹果使用了 `rootless` 保护机制, 禁止了用户对 `/usr/bin` `/usr/sbin` `/sbin` 的所有文件的修改权限. 所以无法向以上三个目录中安装 `Cocoapods`. 当然, 关闭 `rootless` 是一种办法, 我这里使用的是安装在 `/usr/local/bin` 下.

## 下载specs

这里我提供两种方法, 第一种是直接使用官方的镜像, 第二种是使用国内的镜像, 使用官方的镜像下载速度慢但胜在方便, 使用国内镜像不必多说, 网速有多快, 下载速度有多快.

1. ```
$ pod setup
```

2. ```
$ pod repo add master https://gitcafe.com/akuandev/Specs.git
```

如果使用了第二种方法, 每次在编辑 `Podfile` 都需要在第一行加入 `source "https://gitcafe.com/akuandev/Specs.git"`, 不然会自动从官方源下载第三方类库, 而一旦找不到官方源镜像...就会重新下载一遍官方镜像.

## 找到项目目录并且生成 Podfile 文件

找到项目根目录

```
$ vim Podfile
```

在其中输入

```
source "https://gitcafe.com/akuandev/Specs.git"
platform: ios, '8.0'

target '项目名' do
    pod '第三方类库名', `~> 版本`
    ...
end
```

`:wq` 保存并且退出

## 集成第三方类库

```
$ pod install
```

如果不想在 下载集成第三方类库的时候更新 `spec`, 可以使用以下命令

```
$ pod install --verbose --no-repo-update
```

在更新第三方类库时同理

```
$ pod update --verbose --no-repo-update 
```

> 集成完成后, 切记使用后缀为 `xcworkspace` 打开项目.
