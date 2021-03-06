<!-- toc -->

# 安装Go开发环境

## 工具链介绍

go 有两套编译工具链，分别是从 plant9 移植过来的 `gc` 和依赖 **gcc** 的 `gccgo` 。

gc 工具链对操作系统和 CPU 的支持情况如下：

| 操作系统                         | CPU类型         | 备注                                                                     |
| -------------------------------  | --------------- | ----------------------------------------------------------------------   |
| FreeBSD 8 or later               | amd64, 386, arm | Debian GNU/kFreeBSD not supported; FreeBSD/ARM needs FreeBSD 10 or later |
| Linux 2.6.23 or later with glibc | amd64, 386, arm | CentOS/RHEL 5.x not supported; no binary distribution for ARM yet        |
| Mac OS X 10.6 or later           | amd64, 386      | use the gcc** that comes with Xcode**                            |
| Windows XP or later              | amd64, 386      | use MinGW gcc. No need for cygwin or msys.                               |

对于其它操作系统或 CPU 类型，需要从源码编译 gc 工具链或使用 gccgo ：

+ 如果使用 cgo，则需要安装 gcc；
+ Xcode command tool 是 Xcode 的一部分，它包含 gcc 编译器, 可以从 Xcode 的 Componts->Downloads 对话框中下载该 tool；

## 安装工具链

官方为 gc 工具链提供了源码和二进制安装包，可以根据需要选择一种安装方式。

### 一、二进制安装

1. 从[官网](https://golang.org/dl/)下载二进制包如 `go1.6.linux-amd64.tar.gz`
1. 解压到 `/usr/local` 目录：

    ``` bash
    $ tar -C /usr/local -xzf go$VERSION.$GOOS-$GOARCH.tar.gz
    $
    ```

1. 将 `/usr/local/go/bin` 添加到 `PATH` 中：

    ``` bash
    $ export PATH=$PATH:/usr/local/go/bin
    $
    ```

go默认假设被安装到 `/usr/local/go`，如果被安装到其它位置则需要设置 `GOROOT` 环境变量，例如二进制包被解压到 `$HOME` 目录：

``` bash
$ export GOROOT=$HOME/go
$ export PATH=$PATH:$GOROOT/bin
$
```

### 二、从源码编译安装

#### 安装Go编译器二进制文件

1.4版本后的Go工具链是用go语言写的，如果要构建它，系统需要安装Go编译器：

+ 如果系统已经有 >= 1.4版本的go工具链，则将`GOROOT_BOOTSTRAP`变量设置为所在目录:

    ``` bash
    $ unset GOROOT GOPATH  # 如果系统已经有go工具链，需要清除`$GOPATH`和`$GOROOT`变量；
    $ export GOROOT_BOOTSTRAP=$HOME/local/go
    $
    ```

+ 否则，需要下载1.4版本的go工具链，该版本是C写的，只依赖gcc和glibc；可以下载二进制或编译并安装源码，然后将`GOROOT_BOOTSTRAP`变量设置为所在目录；

    ``` bash
    $ cd /tmp
    $ git clone git@github.com:golang/go.git
    $ cd go
    $ git checkout -b 1.4.3 go1.4.3
    $ cd src
    $ ./all.bash  # 编译go 1.4.3
    $ export GOROOT_BOOTSTRAP=/tmp/go/go  # GOROOT\_BOOTSTRAP缺省值为`$HOME/go1.4`，如果安装到其它位置，则需要重新定义；
    $
    ```

使用`GOROOT_BOOTSTRAP`变量指定Go工具链位置(如果位于$HOME/go1.4，则无需指定)后，可以执行源码中的`bootstrap.bash`脚本，生成支持`$GOOS`、`$GOARCH`指定的目标操作系统和架构的新工具链：

``` bash
$ GOOS=linux GOARCH=ppc64 ./bootstrap.bash
$
```

该命令生成的工具链位于`../../go-${GOOS}-${GOARCH}-bootstrap.`目录，可以设置为变量`GOROOT_BOOTSTRAP`的值，用于后续编译源码；

#### 编译最新的go源码

1. 设置git代理：

    ``` bash
    $ git config http.proxy http://user:passwd@host:port
    $ git config https.proxy https://user:passwd@host:port
    $
    ```

1. 设置go get代理：

    ``` bash
    $ export http_proxy=http://user:passwd@host:port
    $
    ```

1. 获取源代码：

    + 从[官网](https://golang.org/dl)下载

      ``` bash
      $ wget https://golang.org/dl/go$VERSION.src.tar.gz
      $ tar -xzvf go$VERSION.$OS-$ARCH.tar.gz
      $
      ```

    + 或者从git代码库clone：

      ``` bash
      $ git clone https://go.googlesource.com/go # 需翻墙
      $ git clone https://github.com/golang/go.git
      $ cd go
      $ git checkout go1.6 # 也可以切换到其它分支如master
      $
      ```

1. 编译源代码

    ``` bash
    $ pwd
    /tmp/
    $ cd go/src
    $ ./all.bash  #使用`GOROOT_BOOTSTRAP`指定的go 1.4版本以上的工具链来编译go源码
    ...
    ALL TESTS PASSED

    ---
    Installed Go for linux/amd64 in /tmp/go
    Installed commands in /tmp/go/bin
    *** You need to add /tmp/go/bin to your PATH.
    ```

    go 会将安装位置记录到二进制的`GOROOT`变量中，如果需要调整安装目录，可以设置`$GOROOT_FINAL=/path/to/goTree`, 这样编译完后会提示将`/tmp/go`移动到
    `/path/to/goTree`目录(这个参数只能在编译阶段有效，如果在编译后移动Go tree则参考步骤5)。

    ``` bash
    $ ls /tmp/go -F
    api/  AUTHORS  bin/  CONTRIBUTORS  doc/  favicon.ico  include/  lib/
    LICENSE  misc/  PATENTS  pkg/  README  robots.txt  src/  test/  VERSION
    $ ls /tmp/go/bin  # 源码包自带的二进制工具命令， 没有 godoc
    go  gofmt
    $ ls /tmp/go/pkg/tool/linux_amd64/
    addr2line  asm  cgo  compile  dist  doc  fix  link  nm  objdump  pack  pprof  tour yacc
    $ /tmp/go/bin/go env |grep -E 'GOROOT|GOTOOLDIR'
    GOROOT="/tmp/go"   # 可见Go tree被安装到预期位置
    GOTOOLDIR="/tmp/go/pkg/tool/linux_amd64"
    ```

    将`/tmp/go/bin`加入到PATH中，即可使用。

1. 移动 Go 源码目录

    可以将编译过的Go Tree移动到其它目录，然后修改`GOROOT`环境变量即可。

    ``` bash
    $ mkdir /tmp/xxx
    $ export GOROOT=/tmp/xxx
    $ mv * /tmp/xxx
    $ /tmp/xxx/bin/go env |grep -E 'GOROOT|GOTOOLDIR'
    GOROOT="/tmp/xxx" #Go tree和工具链自动调整
    GOTOOLDIR="/tmp/xxx/pkg/tool/linux_amd64"

    $ # 设置PATH和GOPATH
    $ export PATH=/tmp/xxx/bin:$PATH
    $ which go
    /tmp/xxx/bin/go
    $ go version
    go version go1.4 linux/amd64
    ```

1. 安装额外的工具如`godoc`, `vet`, `cover`(二进制发布版中包含这些工具，无需额外安装)：

    一些Go工具位于[go.tools](https://golang.org/x/tools)仓库中，需要额外安装。

    ``` bash
    $ #安装所有工具：
    $ go get golang.org/x/tools/cmd/... #...是通配符，参考： go help packages

    $ go get golang.org/x/tools/cmd/godoc #只安装godoc工具
    $ ls bin/ #多了godoc
    go  godoc  gofmt
    $ ls pkg/tool/linux_amd64/ #多了vet,cover
    addr2line  asm  cgo  compile  cover  dist  doc  fix  link  nm  objdump  pack  pprof  tour  trace  vet  yacc
    ```

    go 命令会将`godoc`安装到`$GOROOT/bin`或者`$GOBIN`，其它的`go tool`如`cover`、`vet`安装到`$GOROOT/pkg/tool/$GOOS_$GOARCH`。可以用`go tool cover`或`go tool vet`命令来调用后面目录中的程序。

## 测试工具链

1. 创建和设置`GOPATH`(非必须):

    ``` bash
    $ mkdir -p $HOME/go/{src,bin,pkg}
    $ export GOPATH=$HOME/go
    $ mkdir $HOME/go/src/demo
    $ cd !$
    $
    ```

1. 编写一个测试文件如 hello.go

    ``` bash
    package main

    import "fmt"

    func main() {
      fmt.Printf("hello, world\n")
    }
    ```

1. 编译并执行， `-x`选项可以打印出编译过程

    ``` bash
    $ go build -x demo.go
    WORK=/tmp/go-build333633893
    mkdir -p $WORK/command-line-arguments/_obj/
    mkdir -p $WORK/command-line-arguments/_obj/exe/
    cd /home/ksyun/golang/src
    /home/ksyun/local/go/pkg/tool/linux_amd64/compile -o $WORK/command-line-arguments.a -trimpath $WORK -p main -complete -buildid ed5feda32ea5b5ab51ac7fe9d1193005f6f99836 -D _/home/ksyun/golang/src -I $WORK -pack ./demo.go
    cd .
    /home/ksyun/local/go/pkg/tool/linux_amd64/link -o $WORK/command-line-arguments/_obj/exe/a.out -L $WORK -extld=gcc -buildmode=exe -buildid=ed5feda32ea5b5ab51ac7fe9d1193005f6f99836 $WORK/command-line-arguments.a
    mv $WORK/command-line-arguments/_obj/exe/a.out demo
    hello, world
    ```

## 环境变量(可选)

编译工具链可以使用以下环境变量进行配置；

`$GOROOT`
: 构建时，值为`all.bash`脚本所在目录的父目录，会被写入到生成的二进制中；如果后续移动了安装目录，则使用该变量指定新的Go Tree顶层目录；

`$GOROOT_FINAL`
: 一般和`$GOROOT`一致，定义在构建后的安装阶段安装到的位置；

`$GOOS`和`$GOARCH`
: 交叉编译时，分别定义目标操作系统和体系结构，默认和`$GOHOSTOS`和`$GOHOSTARCH`一致，各组合如下:

    ``` text
    $GOOS	$GOARCH
    android	arm
    darwin	386
    darwin	amd64
    darwin	arm
    darwin	arm64
    dragonfly	amd64
    freebsd	386
    freebsd	amd64
    freebsd	arm
    linux	386
    linux	amd64
    linux	arm
    linux	arm64
    linux	ppc64
    linux	ppc64le
    linux	mips64
    linux	mips64le
    netbsd	386
    netbsd	amd64
    netbsd	arm
    openbsd	386
    openbsd	amd64
    openbsd	arm
    plan9	386
    plan9	amd64
    solaris	amd64
    windows	386
    windows	amd64
    ```

    交叉编译：在 `linux amd64` 机器上生成 `windows amd64` 的二进制文件：

    ``` bash
    $ GOOS=windows GOARCH=amd64 go build
    $ file pssh.exe
    pssh.exe: PE32+ executable for MS Windows (console) Mono/.Net assembly
    ```

`$GOHOSTOS`和`$GOHOSTARCH`
: 编译工具链所在主机的操作系统和架构类型，必须与所在操作系统和CPU架构类型兼容；

`$GOBIN`
: 如果设置，则所有的Go二进制程序将安装到此目录，而不是默认的`$GOPATH/bin`

## 参考

1. [Getting Started](http://golang.org/doc/install)
1. [Installing Go from source](http://golang.org/doc/install/source)