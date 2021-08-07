title: 网络 --- vegeta压测工具使用
author: _Tao
tags: []
categories:
  - 网络
date: 2021-06-20 15:05:00
---
### 介绍
Vegeta 是一个用 Go 语言编写的多功能的 HTTP 负载测试工具，它提供了命令行工具和一个开发库。

官方地址：https://github.com/tsenart/vegeta

### 安装
mac版本:`brew install vegeta`
其他版本没试过, 去上面的github看

### 参数介绍
```
-cpus int:    使用 CPU 的数量 (默认为 4 个)
-profile string:    指定在执行期间启用哪个分析器，支持 cpu 和 heap。
-version:    打印版本并退出。attack command:
-body string:    指定请求主体文件里的内容。
-cert string:    指定用于 HTTPS 请求的 PEM 格式的客户端证书文件。如果 -key 未指定，它会被设置为这个标志的值。
-connections int:    指定每个目标主机打开的空闲连接的最大数目，默认值为 10000。
-duration duration:    指定发送请求到目标主机的时长，用 0 表示永久。
-header value:    指定目标的请求头，可以重复指定多个请求头。
-http2:    指定是否向支持的服务器发送 HTTP/2 请求，默认为：true。
-insecure:    指定是否忽略无效的服务器 TLS 证书。
-keepalive:    指定是否使用持久链接，默认值为：true。
-key string:    指定 HTTPS 请求中使用的 PEM 编码的 SSL 客户端证书私钥文件。
-laddr value:    指定要使用的本地 I P地址，默认值为：0.0.0.0。
-lazy:    指定是否使用延迟模式读取目标。
-output string:    指定输出文件的位置，默认为标准输出。
-rate uint:    指定每秒钟对目标发送的请求数，默认值为：50。
-redirects int:    指定每个请求的重定向的最大次数，默认为 10 次。当值为 -1, 不会遵循重定向但响应标记为成功。
-root-certs value:    指定可信的 TLS 根证书文件，多个的情况下使用逗号分隔。如果未指定，使用系统默认的 CA 证书。
-targets string:    指定目标文件，默认为标准输入。
-timeout duration:    指定每个请求的超时时间，默认值为 30s。
-workers uint:    指定初始化进程数量，默认值为 10。
-inputs string:    指定报告输入文件，默认为标准输入。
-output string:    指定报告输出文件，默认为标准输出。
-reporter string:    指定要生成的报告的格式，支持 text，json, plot, hist[buckets]。默认为文本。dump command:
-dumper string:    指定转存文件，支持 json, csv 格式。默认为 json 格式。
-inputs string:    指定要转存的输入文件，默认为标准输入，指定多个用逗号分隔。
-output string:    指定要转存的输出文件，默认为标准输出。
```

### 输出介绍
```
#使用标准输入进行压测并生成报告
[root@localhost1]echo "GET http://10.0.0.141"| vegeta attack -rate=500 -connections=1 -duration=1s | tee results.bin | vegeta report
Requests（请求）      [total（请求总数）, rate（请求速度）]            500, 501.00
Duration（攻击）      [total（总共攻击与等待的时间）, attack（攻击的时间）, wait（等待时间）]    998.571503ms, 997.999647ms, 571.856µs
Latencies（执行时间）     [mean（单个请求的平均值）, 50（50%请求达到的时间）, 95, 99, max（单个最大请求时间）]  1.088556ms, 561.997µs, 2.414125ms, 12.116341ms, 22.107566ms
Bytes In（请求的大小（字节））      [total（请求总大小）, mean（请求平均大小）]            306000, 612.00
Bytes Out（字节输出）     [total（总输出）, mean（平均输出）]            0, 0.00
Success（请求成功率）       [ratio（请求成功率）]                  100.00%
Status Codes  [code（状态码）:count（请求次数）]             200:500 
Error Set:（错误集）
```

举例:
```
❯ echo "GET http://127.0.0.1:8888/2" | vegeta attack -duration=20s  -rate=0 -max-workers=200 -workers=200 | tee results.bin | vegeta report
Requests      [total, rate, throughput]         376, 18.74, 6.51
Duration      [total, attack, wait]             32.885s, 20.064s, 12.821s
Latencies     [min, mean, 50, 90, 95, 99, max]  533.233µs, 13.995s, 16.971s, 24.209s, 30s, 30.003s, 30.003s
Bytes In      [total, mean]                     3638, 9.68
Bytes Out     [total, mean]                     0, 0.00
Success       [ratio]                           56.91%
Status Codes  [code:count]                      0:162  200:214
Error Set:
Get "http://127.0.0.1:8888/2": read tcp 127.0.0.1:50213->127.0.0.1:8888: read: connection reset by peer
Get "http://127.0.0.1:8888/2": read tcp 127.0.0.1:50217->127.0.0.1:8888: read: connection reset by peer
Get "http://127.0.0.1:8888/2": read tcp 127.0.0.1:50218->127.0.0.1:8888: read: connection reset by peer
```