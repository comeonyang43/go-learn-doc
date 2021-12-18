
## 1. 什么是Gvm

`Go` 语言版本管理器（`Gvm`）是管理 `Go` 语言环境的开源工具。`Gvm 「pkgsets」` 支持安装多个版本的 `Go` 并管理每个项目的模块。它最初由 `Josh Bussdieker` 开发，`Gvm` 与 `Ruby RVM` 类似，允许你为每个项目或一组项目创建一个开发环境，分离不同的 `Go` 版本和包依赖关系，来提供更大的灵活性，以防不同版本造成的问题。`Gvm` 主要有以下几个特性：

- 管理 `Go` 的多个版本，包括安装、卸载和指定使用 `Go` 的某个版本
- 查看官方所有可用的 `Go` 版本，同时可以查看本地已安装和默认使用的 `Go` 版本
- 管理多个 `GOPATH`，并可编辑 `Go` 的环境变量
- 可将当前目录关联到 `GOPATH`
- 可以查看 `GOROOT` 下的文件差异
- 支持 `Go` 版本切换

> [gvm Star6.8k 源码地址：https://github.com/moovweb/gvm](https://github.com/moovweb/gvm)

## 2. 安装Gvm

```bash
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
```


## 3. Gvm功能预览

```bash
➜  gvm
Usage: gvm [command]

Description:
  Gvm is the Go Version Manager

Commands:
  # 打印 Gvm 的版本
  version    - print the gvm version number 
  # 获取 Gvm 最新的代码
  get        - gets the latest code (for debugging) 
  # 使用某个go版本，临时生效，加上--default永久生效
  use        - select a go version to use (--default to set permanently) 
  diff       - view changes to Go root
  help       - display this usage text
  # 彻底删除gvm和安装的所有 go 版本和包
  implode    - completely remove gvm
  # 安装某个go的版本
  install    - install go versions
  # 卸载某个 go 版本
  uninstall  - uninstall go versions
  # 安装交叉编译器，gvm cross [os] [arch]，os = linux/darwin/windows，arch = amd64/386/arm
  cross      - install go cross compilers
  linkthis   - link this directory into GOPATH
  # 列出安装的 Go 版本
  list       - list installed go versions
  # 列出可用的 Go 版本，使用 --all，列出所有的 tags 
  listall    - list available versions
  # 管理 Go 版本别名
  alias      - manage go version aliases
   - # gvm alias list ，列出所有别名
   - # gvm alias create [alias name] [go version name]，创建别名
   - # gvm alias delete [alias name] ，删除别名
  # 管理 GOPATH 环境变量, 会在 ~/.gvm/environments 目录下创建相应的文件
  pkgset     - manage go packages sets
  # 编辑 pkgset 的环境变量
  pkgenv     - edit the environment for a package set
```



## 4. 使用Gvm

### 4.1 查看Go所有版本

```bash
➜  gvm listall
gvm gos (available)
   ...
   go1.16.12
   go1.17
   go1.17beta1
   go1.17rc1
   ...
   go1.17.5
   go1.18beta1
   release.r56
   ...
```
### 4.2 安装Go某版本

```bash
➜ gvm install go1.18beta1
Updating Go source...
Installing go1.18beta1...
 * Compiling...
go1.18beta1 successfully installed!
```

### 4.3 使用Go某版本

```bash
# 临时生效
➜ gvm use go1.18beta1
# 永久生效
➜ gvm use go1.18beta1 --default
```

### 4.4 查看当前使用版本

```bash
➜ gvm list
gvm gos (installed)
   go1.16
=> go1.17 # 代表当前正在使用的版本
   go1.18beta1
   system
```

## 5. 设置全局`go env`

### 5.1 修改`.bashrc`

```bash
#G OPATH
export GOPATH=/Users/xxx/xx
# GOBIN
export GOBIN=$GOPATH/bin
# GoEnv
export GOENV=$GOPATH/env
# 代理
export GOPROXY=https://goproxy.cn/,https://mirrors.aliyun.com/goproxy/,direct
# go module
export GO111MODULE=auto
```


