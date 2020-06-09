# hello word

本节使用一个最简单的程序教会大家如何从零开始编写一个hello word。



## 环境搭建

本节不会教大家从源码开始安装，教大家如何在官方下载安装程序工具或者免安装压缩包进行安装和环境搭建。

[点击下载 https://golang.org/dl/](https://golang.org/dl/)，切记找对应的32或者64版本，下面例子是基于64位系统安装



### windown

Go项目为Windows用户提供了两个安装选项(除了从源代码安装之外):需要设置一些环境变量的zip归档文件和自动配置安装的MSI安装程序。

#### MSI安装

打开MSI文件，按照提示安装Go工具。默认情况下，安装程序将Go发行版放在c:\Go中。

安装程序应该将c:\Go\bin目录放在PATH环境变量中。

####  免安装包

[下载安装包](https://dl.google.com/go/go1.14.4.windows-amd64.zip) 解压，并将/bin目录增加到path环境变量。

### mac & Linux

[Download the archive](https://golang.org/dl/) and extract it into `/usr/local`, creating a Go tree in `/usr/local/go`. For example:

```
tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
```

添加/usr/local/go/bin到PATH环境变量。你可以将这行代码添加到你的/etc/profile(用于系统范围的安装)或$HOME/.profile中:

```
export PATH=$PATH:/usr/local/go/bin
```









```go
package main

import "fmt"

func main() {
	fmt.Println("hello word!")
}

输出： hello word!
```

