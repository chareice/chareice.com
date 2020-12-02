---
title: "在Apple M1上运行golang native"
date: 2020-12-01T08:09:48+08:00
draft: true
---

Golang对Apple silicon的支持要等到21年2月份，但是现在，可以通过源码编译的方式自行编译Golang来实现对Apple silicon的原生运行。

编译除了需要Apple M1的机器之外，还需要一台已经安装了Go的机器。整体步骤如下：
	1. 在已经安装好Go的机器上编译Apple M1的Bootstrap
	2. 将编译好的Bootstrap拷贝到Apple M1机器上
	3. 在Apple M1机器上使用Bootstrap编译Go

特别提示：由于目前的Go在Apple m1上存在codesigning的问题，有未合并的提交解决了此问题，所以编译需要切换到此提交：[https://go-review.googlesource.com/c/go/+/272258/](https://go-review.googlesource.com/c/go/+/272258/)

### 编译Bootstrap

```bash
git clone https://go.googlesource.com/go
git fetch https://go.googlesource.com/go refs/changes/58/272258/2 && git checkout -b change-272258 FETCH_HEAD
cd go/src
GOOS=darwin GOARCH=arm64 ./bootstrap.bash
```

编译完成后将打包好的go-darwin-arm64-bootstrap.tbz复制到Apple m1机器上。

### 编译Go
在Apple M1上，解压go-darwin-arm64-bootstrap.tbz，同样clone go的源代码。

```bash
git clone https://go.googlesource.com/go
git fetch https://go.googlesource.com/go refs/changes/58/272258/2 && git checkout -b change-272258 FETCH_HEAD


➜  src git:(change-272258) cd src
# 这里将GOROOT_BOOTSTRAP替换成你机器上的位置
➜  src git:(change-272258) GOROOT_BOOTSTRAP=/Users/chareice/Downloads/go-darwin-arm64-bootstrap ./make.bash
go-darwin-arm64-bootstrap ./make.bash
Building Go cmd/dist using /Users/chareice/Downloads/go-darwin-arm64-bootstrap. (devel +5d1a63e769 Mon Nov 30 14:24:04 2020 -0500 darwin/arm64)
Building Go toolchain1 using /Users/chareice/Downloads/go-darwin-arm64-bootstrap.
Building Go bootstrap cmd/go (go_bootstrap) using Go toolchain1.
Building Go toolchain2 using go_bootstrap and Go toolchain1.
Building Go toolchain3 using go_bootstrap and Go toolchain2.
Building packages and commands for darwin/arm64.
---
Installed Go for darwin/arm64 in /Users/chareice/projects/goroot
Installed commands in /Users/chareice/projects/goroot/bin
```

编译完成之后，将goroot/bin所在位置添加到PATH中，就可以使用Go了。

```bash
➜  ~ go version
go version devel +5d1a63e769 Mon Nov 30 14:24:04 2020 -0500 darwin/arm64
```
