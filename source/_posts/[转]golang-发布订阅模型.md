title: '[转]golang - 发布订阅模型'
author: _Tao
tags: []
categories:
  - golang
date: 2020-06-04 19:56:00
---
### 发布订阅者
```golang
package pubsub

import (
	"sync"
	"time"
)

type (
	// 订阅者是一个管道
	subscriber chan interface{}
	// 主题是一个过滤器
	topicFunc func(v interface{}) bool
)

// 发布者对象
type Publisher struct {
	// 读写锁
	m sync.RWMutex
	// 订阅队列的缓存大小
	buffer int
	// 发布的过期时间
	timeout time.Duration
	// 订阅者信息
	subscribers map[subscriber]topicFunc
}

// 构建一个发布者, 设置超时时间和缓存队列长度
func NewPublisher(publishTimeout time.Duration, buffer int) *Publisher {
	return &Publisher{
		buffer:      buffer,
		timeout:     publishTimeout,
		subscribers: make(map[subscriber]topicFunc),
	}
}

// 添加一个订阅者, 订阅过滤器筛选后的主题
// 添加订阅主题参数, 返回订阅者
func (p *Publisher) SubscribeTopic(topic topicFunc) chan interface{} {
	ch := make(chan interface{}, p.buffer)
	p.m.Lock()
	p.subscribers[ch] = topic
	p.m.Unlock()
	return ch
}

// 订阅所有的主题
func (p *Publisher) Subscribe() chan interface{} {
	return p.SubscribeTopic(nil)
}

//退订
func (p *Publisher) Evict(sub chan interface{}) {
	p.m.Lock()
	defer p.m.Unlock()

	delete(p.subscribers, sub)
	close(sub)
}

// 发送一个主题
func (p *Publisher) Publish(v interface{}) {
	p.m.RLock()
	defer p.m.RUnlock()

	var wg sync.WaitGroup
	// 遍历所有的订阅者, 一个个的发布
	for sub, topic := range p.subscribers {
		wg.Add(1)
		go p.sendTopic(sub, topic, v, &wg)
	}

	wg.Wait()
}

// 实际上的发送函数
/**
sub: 订阅者, chan
topic: 订阅主题, 即传进来的函数
v: 函数参数
 */
func (p *Publisher) sendTopic(sub subscriber, topic topicFunc, v interface{}, wg *sync.WaitGroup) {
	defer wg.Done()
	// 该代码中, topic会返回true或者false, 如果返回false就表示没有订阅
	// 如果topic是nil, 就表示全订阅
	if topic != nil && !topic(v) {
		return
	}

	select {
	case sub <- v:	// 将参数传递给了订阅者, 订阅者能打印, 则表示收到发布的内容
	case <-time.After(p.timeout):	// 超时时间
	}
}

// 关闭发布者对象, 同时关闭所有的订阅者管道
func (p *Publisher) Close() {
	p.m.Lock()
	defer p.m.Unlock()

	for sub := range p.subscribers {
		delete(p.subscribers, sub)
		close(sub)
	}
}

```
<!-- more -->

### 测试代码

```golang
package main

import (
	"GoStudy/chai2010.gitbooks.io/p1/pubsub"
	"fmt"
	"strings"
	"time"
)

func main() {
	p := pubsub.NewPublisher(100*time.Microsecond, 10)
	defer p.Close()

	// 订阅所有主题
	all := p.Subscribe()
	// 订阅包含了golang字符串的主题
	// 返回了一个chan
	golang := p.SubscribeTopic(func(v interface{}) bool {
		if s, ok := v.(string); ok {
			return strings.Contains(s, "golang")
		}

		return false
	})

	p.Publish("hello World!")
	p.Publish("hello golang!")

	go func() {
		for msg := range all {
			fmt.Println("all: ", msg)
		}
	}()

	go func() {
		for msg := range golang {
			fmt.Println("golang: ", msg)
		}
	}()

	time.Sleep(3 * time.Second)

}

```


### 参考
[常见的并发模式](https://chai2010.gitbooks.io/advanced-go-programming-book/content/ch1-basic/ch1-06-goroutine.html)