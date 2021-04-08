title: golang - 解析yaml配置文件
author: _Tao
tags: []
categories: []
date: 2020-12-14 21:09:00
---
# golang - 解析yaml配置文件

### 安装第三方库

```go
go get -u "github.com/spf13/viper"
```

### 文件整体结构

```
.
├── configs
│   └── stand_alone.yaml
└── pkg
    └── settings
        ├── section.go
        └── setting.go
```

### yaml配置文件

路径: configs/stand_alone.yaml

详细格式可以参考:

[YAML 语言教程](http://www.ruanyifeng.com/blog/2016/07/yaml.html)

[YAML](https://en.wikipedia.org/wiki/YAML)

```yaml
Server:
  StorageRoot: "Storage/upload"
  ListenAddress: ":8888"
```

<!-- more -->

### 定义Setting结构体

路径: pkg/settings/setting.go

```go
package settings

import "github.com/spf13/viper"

type Setting struct {
	vp *viper.Viper
}

func NewSetting() (*Setting, error) {
	vp := viper.New()
	vp.SetConfigName("stand_alone.yaml")
	vp.AddConfigPath("stand_alone/configs")
	vp.SetConfigType("yaml")

	err := vp.ReadInConfig()
	if err != nil {
		return nil, err
	}

	return &Setting{vp}, nil
}

```

### 定义Section结构体

路径: pkg/settings/section.go

如果有多组配置, 那么按照格式, 创建多个struct变量

```go
package settings

type Server struct {
	StorageRoot   string
	ListenAddress string
}

func (s *Setting) ReadSection(sType string, section interface{}) error {
	err := s.vp.UnmarshalKey(sType, section)
	if err != nil {
		return err
	}

	return nil
}

```

### 解析

```go
package main

import (
	"Storage/stand_alone/pkg/settings"
	"fmt"
	"log"
)

func main() {
	setting, err := settings.NewSetting()
	if err != nil {
		log.Println("err: ", err)
		return
	}

	var server *settings.Server
	_ = setting.ReadSection("Server", &server)
	fmt.Println(server.StorageRoot, server.ListenAddress)
}

```