[TOC]

# Go环境变量

为方便开发，在开发环境往往需要设置三个环境变量：

1. `$GOROOT`：go的安装目录，配置后不会再更改；
1. `$GOPATH`：go项目在本地的开发环境的的项目根路径(以便项目编译，`go build`, `go install`)，不同的项目在编译的时候该环境变量可以不同；
1. `$PATH`（重要）：需要将go的bin目录添加到系统`$PATH`中以便方便使用go的相关命令，配置后也不会再更改；

Go的环境变量在官方文档中也有详情的说明，请参考链接：[https://golang.google.cn/doc/install/source](https://golang.google.cn/doc/install/source)

> 环境变量中的`$GOOS`和`$GOARCH`是比较实用的两个变量，可以用在不同平台的交叉编译中，只需要在`go build`之前设置这两个变量即可，这也是go语言的优势之一：可以编译生成跨平台运行的可执行文件。感觉比QT更高效更轻量级，虽然生成的可执行文件是大了一点，不过也在可接受的范围之内。
例如，在`Linux amd64`架构下编译`Windows x86`的可执行文件，可以使用如下命令：
```
CGO_ENABLED=0 GOOS=windows GOARCH=386 go build hello.go
```
遗憾的是交叉编译暂不支持`cgo`方式，因此需要将环境变量`$CGO_ENABLED`设置为0，这样执行之后会在当前目录生成一个`hello.exe`的`windows x86`架构的可执行文件。

## 环境变量设置

除了`$PATH`环境外，其他环境变量都是可选的。

为什么说这个步骤可选呢？因为未来的Go版本慢慢开始移除对`$GOPATH`/`$GOROOT`的支持。此外，在Goland这个IDE中集成有`Terminal`功能，直接使用这个功能中已经设置好了环境变量。

![](/images/goland7.png)


## `*nix`下设置环境变量
在`*nix`系统下(`Linux/Unix/MacOS/*BSD`等等)，需要在`/etc/profile`中增加以下环境变量设置后，执行命令`#source /etc/profile`重新加载profile配置文件（或重新登录），将以下变量添加到用户的环境变量中:
```shell
export GOROOT=/usr/local/go
export GOPATH=/Users/john/Workspace/Go/GOPATH
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```

## `Windows`下设置环境变量
Windows如何修改系统环境变量，以及修改环境变量`PATH`，请参考网上教程([百度](https://www.baidu.com/s?wd=Windows%20%E4%BF%AE%E6%94%B9%E7%B3%BB%E7%BB%9F%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%20PATH) 或 [Google](https://www.google.com/search?q=Windows+修改系统环境变量+PATH))。



# IDE工具配置

本文以`Goland`开发工具为基础，介绍在该IDE下的常用工具配置。

常用的工具包括：

1. `go fmt` : 统一的代码格式化工具（必须）。
1. `golangci-lint` : 静态代码质量检测工具，用于包的质量分析（推荐）。
1. `goimports` : 自动`import`依赖包工具（可选）。
1. `golint` : 代码规范检测，并且也检测单文件的代码质量，比较出名的Go质量评估站点[Go Report](https://goreportcard.com)在使用（可选）。

## `go fmt`, `goimports`, `golangci-lint`的配置

由于这三个工具是`Goland`自带的，因此配置比较简单，参考以下图文操作示例：

1. 在`Goland`的设置中，选择`Tools` - `File Watchers`，随后选择添加

    ![](/images/WX20190619-092825@2x.jpg)

1. 依次点击添加这3个工具，使用默认的配置即可

    ![](/images/WX20190619-093508@2x.jpg)

1. 随后在撸代码的过程中保存代码文件时将会自动触发这3个工具的自动检测。

## `golint`工具的安装及配置


### `golint`的安装
由于`Goland`没有自带`golint`工具，因此首先要自己去下载安装该工具。

使用以下命令安装：
```html
mkdir -p $GOPATH/src/golang.org/x/
cd $GOPATH/src/golang.org/x/
git clone https://github.com/golang/lint.git
git clone https://github.com/golang/tools.git
cd $GOPATH/src/golang.org/x/lint/golint
go install
```
安装成功之后将会在`$GOPATH/bin`目录下看到自动生成了`golint`二进制工具文件。

### `golint`的配置

1. 随后在`Goland`的`Tools` - `File Watchers`配置下，通过复制`go fmt`的配置

    ![](/images/WX20190619-201818@2x.jpg)

1. 修改`Name`, `Program`, `Arguments`三项配置，其中`Arguments`需要加上`-set_exit_status`参数，如图所示：

    ![](/images/WX20190619-201304@2x.jpg)

1. 保存即可，随后在代码编写中执行保存操作时将会自动触发`golint`工具检测。


## 配置备份导入

也可以通过保存以下`XML`配置文件内容，使用`import`导入功能即可完成配置（`golint`还是得自己安装）。

![](/images/WX20190619-202618@2x.jpg)

文件内容：
```xml
<TaskOptions>
  <TaskOptions>
    <option name="arguments" value="fmt $FilePath$" />
    <option name="checkSyntaxErrors" value="true" />
    <option name="description" />
    <option name="exitCodeBehavior" value="ERROR" />
    <option name="fileExtension" value="go" />
    <option name="immediateSync" value="false" />
    <option name="name" value="go fmt" />
    <option name="output" value="$FilePath$" />
    <option name="outputFilters">
      <array />
    </option>
    <option name="outputFromStdout" value="false" />
    <option name="program" value="$GoExecPath$" />
    <option name="runOnExternalChanges" value="false" />
    <option name="scopeName" value="Project Files" />
    <option name="trackOnlyRoot" value="true" />
    <option name="workingDir" value="$ProjectFileDir$" />
    <envs>
      <env name="GOROOT" value="$GOROOT$" />
      <env name="GOPATH" value="$GOPATH$" />
      <env name="PATH" value="$GoBinDirs$" />
    </envs>
  </TaskOptions>
  <TaskOptions>
    <option name="arguments" value="-w $FilePath$" />
    <option name="checkSyntaxErrors" value="true" />
    <option name="description" />
    <option name="exitCodeBehavior" value="ERROR" />
    <option name="fileExtension" value="go" />
    <option name="immediateSync" value="false" />
    <option name="name" value="goimports" />
    <option name="output" value="$FilePath$" />
    <option name="outputFilters">
      <array />
    </option>
    <option name="outputFromStdout" value="false" />
    <option name="program" value="goimports" />
    <option name="runOnExternalChanges" value="false" />
    <option name="scopeName" value="Project Files" />
    <option name="trackOnlyRoot" value="true" />
    <option name="workingDir" value="$ProjectFileDir$" />
    <envs>
      <env name="GOROOT" value="$GOROOT$" />
      <env name="GOPATH" value="$GOPATH$" />
      <env name="PATH" value="$GoBinDirs$" />
    </envs>
  </TaskOptions>
  <TaskOptions>
    <option name="arguments" value="run --disable=typecheck $FileDir$" />
    <option name="checkSyntaxErrors" value="true" />
    <option name="description" />
    <option name="exitCodeBehavior" value="ERROR" />
    <option name="fileExtension" value="go" />
    <option name="immediateSync" value="false" />
    <option name="name" value="golangci-lint" />
    <option name="output" value="" />
    <option name="outputFilters">
      <array />
    </option>
    <option name="outputFromStdout" value="false" />
    <option name="program" value="golangci-lint" />
    <option name="runOnExternalChanges" value="false" />
    <option name="scopeName" value="Project Files" />
    <option name="trackOnlyRoot" value="true" />
    <option name="workingDir" value="$ProjectFileDir$" />
    <envs>
      <env name="GOROOT" value="$GOROOT$" />
      <env name="GOPATH" value="$GOPATH$" />
      <env name="PATH" value="$GoBinDirs$" />
    </envs>
  </TaskOptions>
  <TaskOptions>
    <option name="arguments" value="-set_exit_status $FilePath$" />
    <option name="checkSyntaxErrors" value="true" />
    <option name="description" />
    <option name="exitCodeBehavior" value="ERROR" />
    <option name="fileExtension" value="go" />
    <option name="immediateSync" value="false" />
    <option name="name" value="golint" />
    <option name="output" value="$FilePath$" />
    <option name="outputFilters">
      <array />
    </option>
    <option name="outputFromStdout" value="false" />
    <option name="program" value="golint" />
    <option name="runOnExternalChanges" value="false" />
    <option name="scopeName" value="Project Files" />
    <option name="trackOnlyRoot" value="true" />
    <option name="workingDir" value="$ProjectFileDir$" />
    <envs>
      <env name="GOROOT" value="$GOROOT$" />
      <env name="GOPATH" value="$GOPATH$" />
      <env name="PATH" value="$GoBinDirs$" />
    </envs>
  </TaskOptions>
</TaskOptions>
```
