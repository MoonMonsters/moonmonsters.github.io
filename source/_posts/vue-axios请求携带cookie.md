title: Vue -- axios请求携带cookie
author: _Tao
tags: []
categories:
  - Vue
date: 2021-05-29 14:08:00
---
### 修改vue.config.js文件
在该文件中, 添加`disableHostCheck: true`, 不需要host检查
```javascript
module.exports = {
  ...
  devServer: {
    port: port,
    open: true,
    overlay: {
      warnings: false,
      errors: true
    },
    proxy: 'http://localhost:5000',
    disableHostCheck: true
  }
}
```

### 封装axios时, 加上withCredentials
在封装axios时, 加上`withCredentials: true`<br/>
跨源请求不提供凭据(cookie、HTTP认证及客户端SSL证明等)。通过将withCredentials属性设置为true，可以指定某个请求应该发送凭据。<br/>
默认值为false。<br/>
true：在跨域请求时，会携带用户凭证<br/>
false：在跨域请求时，不会携带用户凭证；返回的 response 里也会忽略 cookie<br/>

### 后端返回的数据中, 修改请求头
```python
def _cors_response(response, status=200):
    resp = make_response(response, status)
    resp.headers['Access-Control-Allow-Credentials'] = 'true'
    resp.headers['Access-Control-Allow-Origin'] = 'http://key-ui.oa.zego.im:9528'

    return resp
```
需要注意的是:
1. 传入的是`true`字符串, 不是`True`布尔类型
2. origin不能是`*`, 必须是完整的域名
3. 可以修改hosts文件, 来将`127.0.0.1`指向测试的域名
4. 正式环境中, 这部分操作将由nginx完成, 不在代码中写死