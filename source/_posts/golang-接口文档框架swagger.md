title: Golang --- 接口文档框架swagger
author: _Tao
tags: []
categories:
  - Golang
date: 2020-11-15 14:20:00
---
### 安装
```shell
go get -u github.com/swaggo/swag/cmd/swag@v1.6.5
go get -u github.com/swaggo/gin-swagger@v1.2.0
go get -u github.com/swaggo/files
go get -u github.com/alecthomas/template
```

验证是否成功
```shell
❯ swag -v
swag version v1.6.5
```

### 写入注解
在安装完Swagger关联库后, 就需要在项目里的API接口编写注解,以便后续在生成时能够正确的运行.

#### 常用注解

|  注解   | 描述  |
| ---- | ---- |
| @Summary | 摘要 |
| @Produce | API可以长生的MIME类型的列表. 我们可以把MIME类型简单的理解为响应类型, 例如JSON, XML, HTML等. |
| @Param | 参数格式, 从左到右分别为: 参数名, 入参类型, 数据类型, 是否必填和注释 |
| @Success | 响应成功, 从左到右分别为: 状态码, 参数类型, 数据类型和注释 |
| @Failure | 响应失败, 从左到右分别为状态码, 参数类型, 数据类型和注释 |
| @Router | 路由, 从左到右分别为: 路由地址和HTTP方法  |


#### API函数代码示例
用的是Gin框架.

```golang
package v1

import (
	"github.com/gin-gonic/gin"
)

type Tag struct {
}

func NewTag() Tag {
	return Tag{}
}

func (t Tag) Get(c *gin.Context) {

}

// @Summary 获取多个标签
// @Produce json
// @Param name query string false "标签名称" maxlength(100)
// @Param state query int false "状态" Enums(0, 1) default(1)
// @Param page query int false "页码"
// @Param page_size query int false "每页数量"
// @Success 200 {object} model.Tag "成功"
// @Failure 400 {object} errcode.Error "请求错误"
// @Failure 500 {object} errcode.Error "内部错误"
// @Router /api/v1/tags [get]
func (t Tag) List(c *gin.Context) {

}

// @Summary 新增标签
// @Product json
// @Param name body string true "标签名称" minlength(3) maxlength(100)
// @Param state body int false "状态" Enums(0, 1) default(1)
// @Param created_by body string false "创建者" minlength(3) maxlength(100)
// @Success 200 {object} model.Tag "成功"
// @Failure 400 {object} errcode.Error "请求错误"
// @Failure 500 {object} errcode.Error "内部错误"
// @Router /api/v1/tags [post]
func (t Tag) Create(c *gin.Context) {

}

// @Summary 更新标签
// @Produce json
// @Param id path int true "标签ID"
// @Param name body string false "标签名称" minlength(3) maxlength(100)
// @Param state body int false "状态" Enums(0, 1) default(1)
// @Param modified_by body string true "修改者" minlength(3) maxlength(100)
// @Success 200 {array} model.Tag "成功"
// @Failure 400 {object} errcode.Error "请求错误"
// @Failure 500 {object} errcode.Error "内部错误"
// @Router /api/v1/tags/{id} [put]
func (t Tag) Update(c *gin.Context) {

}

// @Summary 删除标签
// @Produce json
// @Param id path int true "标签ID"
// @Success 200 {string} string "删除成功"
// @Failure 400 {object} errcode.Error "请求错误"
// @Failure 500 {object} errcode.Error "内部错误"
// @Router /api/v1/tags/{id} [delete]
func (t Tag) Delete(c *gin.Context) {

}

```

#### main方法
针对整个项目, 也能写入注解.

```golang
// @title 博客系统
// @version 1.0
// @description Go+Gin框架的博客项目
func main() {
	router := routers.NewRouter()

	s := &http.Server{
		Addr:           ":8080",
		Handler:        router,
		ReadTimeout:    10 * time.Second,
		WriteTimeout:   10 * time.Second,
		MaxHeaderBytes: 1 << 20,
	}
	s.ListenAndServe()
}

```


### 生成文档
在项目的主目录下, 使用命令
```shell
swag init
```
执行完后, 可以看到在docs文件夹中生成了docs.go, swagger.json和swagger.yarm三个文件.

### 路由
```golang
import (
  _ "GoProgrammingJourney/blog_service/docs"
  ginSwagger "github.com/swaggo/gin-swagger"
  "github.com/swaggo/gin-swagger/swaggerFiles"
)

r := gin.New()
r.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))

```

### 查看接口文档

访问网址可以看到生成后的项目文档了.<br/>
http://127.0.0.1:8080/swagger/index.html

![](https://qxinhai.oss-cn-shenzhen.aliyuncs.com/hexo/20201115143526.png)
