title: 'Golang --- 如何在exec连接ssh后, 还能继续操作'
author: _Tao
tags: []
categories:
  - Golang
date: 2021-07-24 19:57:00
---
### 问题
需要使用`cmd := exec.Command("ssh", "-t", "xxx@access.oa.xx.im")`命令, 连接后还要保持可操作, 但实际上, 连接完成后就退出了程序, 报错如下:
```text
0, Pseudo-terminal will not be allocated because stdin is not a terminal.
```
根据网上的一些搜索结果, 把`-t`改为`-tt`也还是无效.

### 答案
改成如下形式
```golang
binary, lookErr := exec.LookPath("ssh")
if lookErr != nil {
	panic(lookErr)
}
syscall.Exec(binary, []string{"ssh", "xxx@access.oa.xxx.im"}, os.Environ())
```

### 链接
[Execute ssh in golang](https://stackoverflow.com/questions/61293342/execute-ssh-in-golang)