title: Golang --- 使用令牌桶限制接口访问频率
author: _Tao
tags:
  - gin
categories:
  - Golang
  - ''
date: 2020-12-12 19:33:00
---
### 三方库

```go
go get -u github.com/juju/ratelimit@v1.0.1
```



### 令牌桶结构体

```go
package limiter

import (
	"github.com/gin-gonic/gin"
	"github.com/juju/ratelimit"
	"time"
)

// 令牌桶信息
type Limiter struct {
	limiterBuckets map[string]*ratelimit.Bucket
}

// 令牌Bucket
type LimiterBucketRule struct {
	// 自定义键值对名称
	Key string
	// 间隔多久放N个令牌
	FillInterval time.Duration
	// 令牌桶的容量
	Capacity int64
	// 每次到达间隔时间后所放的具体令牌数量
	Quantum int64
}

// 接口
type LimiterIface interface {
	// 获取对应的限流器的键值对名称
	Key(c *gin.Context) string
	// 获取令牌桶
	GetBucket(key string) (*ratelimit.Bucket, bool)
	// 新增令牌桶
	AddBuckets(rules ...LimiterBucketRule) LimiterIface
}

```

<!-- more -->


### 实现令牌桶的接口

```go
package limiter

import (
	"github.com/gin-gonic/gin"
	"github.com/juju/ratelimit"
	"strings"
)

type MethodLimiter struct {
	*Limiter
}

// 创建一个令牌桶
func NewMethodLimiter() LimiterIface {
	return MethodLimiter{
		Limiter: &Limiter{
			limiterBuckets: make(map[string]*ratelimit.Bucket),
		},
	}
}

// 获取键值对, 使用核心路由做key值
func (l MethodLimiter) Key(c *gin.Context) string {
	uri := c.Request.RequestURI
	index := strings.Index(uri, "?")
	if index == -1 {
		return uri
	}

	return uri[:index]
}

// 获取Bucket
func (l MethodLimiter) GetBucket(key string) (*ratelimit.Bucket, bool) {
	bucket, ok := l.limiterBuckets[key]
	return bucket, ok
}

// 添加bucket
func (l MethodLimiter) AddBuckets(rules ...LimiterBucketRule) LimiterIface {
	for _, rule := range rules {
		if _, ok := l.limiterBuckets[rule.Key]; !ok {
			l.limiterBuckets[rule.Key] = ratelimit.NewBucketWithQuantum(rule.FillInterval, rule.Capacity, rule.Quantum)
		}
	}

	return l
}

```



### 中间件

```go
package middleware

import (
	"GoProgrammingJourney/blog_service/pkg/app"
	"GoProgrammingJourney/blog_service/pkg/errcode"
	"GoProgrammingJourney/blog_service/pkg/limiter"
	"github.com/gin-gonic/gin"
)

func RateLimiter(l limiter.LimiterIface) gin.HandlerFunc {
	return func(c *gin.Context) {
		// 获取令牌的key值
		key := l.Key(c)
		if bucket, ok := l.GetBucket(key); ok {
			// 传入1, 表示已使用一个令牌
			count := bucket.TakeAvailable(1)
			// 如果剩余可用令牌数为0, 则抛出异常, 禁止访问
			if count == 0 {
				response := app.NewResponse(c)
				response.ToErrorResponse(errcode.TooManyRequests)
				c.Abort()
				return
			}
		}

		c.Next()
	}
}

```



### 加入gin的中间件

```go
// 限制了/auth的访问频率
// 限制时间间隔, 1秒钟
// 一秒钟内, 最多被访问10次
// 当一秒后, 重新放入10个令牌到令牌桶内, 也就是下一秒可再次被访问10次
var methodLimiters = limiter.NewMethodLimiter().AddBuckets(limiter.LimiterBucketRule{
	// 令牌桶限制的url
	Key: "/auth",
	// 时间间隔
	FillInterval: time.Second * 1,
	// 令牌总容量
	Capacity: 10,
	// 重新放入令牌桶数量
	Quantum: 10,
})
```

```go
r := gin.New()
r.Use(middleware.RateLimiter(methodLimiters))
```

### 从配置文件中读取方式

#### 配置yaml

```yaml
Limiter:

  Limits:
    - Key: "/auth"
      FillInterval: 1
      Capacity: 10
      Quantum: 10

    - Key: "/api/v1/tags"
      FillInterval: 1
      Capacity: 10
      Quantum: 10
```



#### 配置Setting

```go
type LimiterSetting struct {
	Limits []struct {
		Key          string
		FillInterval time.Duration
		Capacity     int64
		Quantum      int64
	}
}
```



### 修改中间件参数传入方式

```go
func newLimiter() limiter.LimiterIface {

	var rules []limiter.LimiterBucketRule
	for _, limit := range global.LimiterSetting.Limits {
		rules = append(rules, limiter.LimiterBucketRule{
			// 令牌桶限制的url
			Key: limit.Key,
			// 时间间隔
			FillInterval: limit.FillInterval * time.Second,
			// 令牌总容量
			Capacity: limit.Capacity,
			// 重新放入令牌桶数量
			Quantum: limit.Quantum,
		})
	}
	var methodLimiters = limiter.NewMethodLimiter().AddBuckets(rules...)

	return methodLimiters
}
```

```go
r.Use(middleware.RateLimiter(newLimiter()))
```