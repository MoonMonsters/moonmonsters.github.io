title: '[转]Vue -- axios拦截器'
author: _Tao
tags: []
categories:
  - Vue
date: 2021-05-29 14:09:00
---
### 转载
[axios拦截器](https://hupeip.github.io/2018/10/08/axios%E6%8B%A6%E6%88%AA%E5%99%A8/)


### axios拦截器简单介绍

页面发送http请求，很多情况我们要对请求和其响应进行特定的处理；如果请求数非常多，单独对每一个请求进行处理会变得非常麻烦，程序的优雅性也会大打折扣。好在强大的axios为开发者提供了这样一个API：拦截器。拦截器分为 请求（request）拦截器和 响应（response）拦截器。

#### 请求拦截器
```html
axios.interceptors.request.use(function (config) {
    // 在发起请求请做一些业务处理
    return config;
  }, function (error) {
    // 对请求失败做处理
    return Promise.reject(error);
  });
```

#### 响应拦截器
```html
axios.interceptors.response.use(function (response) {
    // 对响应数据做处理
    return response;
  }, function (error) {
    // 对响应错误做处理
    return Promise.reject(error);
  });
```

### vue添加axios拦截器
#### 安装 axios
```
npm install axios –save-dev
```

#### 新建文件 axios.js
开始统一封装axios， 首先引入axios、qs依赖
```javascript
import axios from "axios";
import qs from "qs";
```
然后创建一个axios实例，这个process.env.BASE_URL在config/dev.evn.js、prod.evn.js里面进行配置：
```javascript
/****** 创建axios实例 ******/
const service = axios.create({
    baseURL: process.env.BASE_URL,  // api的base_url
    timeout: 5000  // 请求超时时间
});
```

#### 使用request拦截器对axios请求配置做统一处理
```javascript
service.interceptors.request.use(config => {    
    app.$vux.loading.show({        
        text: '数据加载中……'    
    });     
    config.method === 'post'        
        ? config.data = qs.stringify({...config.data})        
        : config.params = {...config.params};    
    config.headers['Content-Type'] = 'application/x-www-form-urlencoded';     
    return config;
    }, error => {  //请求错误处理   
        app.$vux.toast.show({        
            type: 'warn',        
            text: error   
        });    
        Promise.reject(error)
    }
);
```

#### 对response做统一处理
```javascript
service.interceptors.response.use(    
    response => {  //成功请求到数据        
        app.$vux.loading.hide();        
        //这里根据后端提供的数据进行对应的处理        
        if (response.data.result === 'TRUE') {            
            return response.data;        
        } else {            
            app.$vux.toast.show({  
                //常规错误处理                
                type: 'warn',                
                text: response.data.data.msg            
            });        
        }    
    },    
    error => {  //响应错误处理console.log('error');        
        console.log(error);        
        console.log(JSON.stringify(error));         
        let text = JSON.parse(JSON.stringify(error)).response.status === 404            
            ? '404'            
            : '网络异常，请重试';        
        app.$vux.toast.show({            
            type: 'warn',            
            text: text        
        });         
        return Promise.reject(error)   
    }
)
```

#### 将axios实例暴露出去
```javascript
export default service;
```
这样一个简单的拦截器就完成了

#### 在main.js中进行引用，并配置一个别名（$ajax）来进行调用
```javascript
import axios from 'axios'
import '../axios.js'    //axios.js的路径

Vue.prototype.$ajax = axios
```

#### 使用场景
#### 应用：一个简单的登录接口
```javascript
this.$ajax({
　　method: 'post',
　　url: '/login',
　　data: {
　　　　'userName': 'haha',
　　　　'password': '123456'
　　}
}).then(res => {
　　console.log(res)
})
```

#### 路由拦截
在定义路由的时候就需要多添加一个自定义字段requireAuth，用于判断该路由的访问是否需要登录。如果用户已经登录，则顺利进入路由，否则就进入登录页面。
```javascript
const routes = [
    {
        path: '/',
        name: '/',
        component: Index
    },
    {
        path: '/repository',
        name: 'repository',
        meta: {
            requireAuth: true,  // 添加该字段，表示进入这个路由是需要登录的
        },
        component: Repository
    },
    {
        path: '/login',
        name: 'login',
        component: Login
    }
];
```

定义完路由后，我们主要是利用vue-router提供的钩子函数beforeEach()对路由进行判断。
```javascript
router.beforeEach((to, from, next) => {
    if (to.meta.requireAuth) {  // 判断该路由是否需要登录权限
        if (token) {  // 判断当前的token是否存在
            next();
        }
        else {
            next({
                path: '/login',
                query: {redirect: to.fullPath}  // 将跳转的路由path作为参数，登录成功后跳转到该路由
            })
        }
    }
    else {
        next();
    }
})
```
to.meta中是我们自定义的数据，其中就包括我们刚刚定义的requireAuth字段
通过这个字段来判断该路由是否需要登录权限
需要的话，同时当前应用不存在token，则跳转到登录页面，进行登录。登录成功后跳转到目标路由。

这种方式只是简单的前端路由控制，并不能阻止用户访问，假设有一种情况：当前token失效了，但是token依然保存在本地。这时候你去访问需要登录权限的路由时，实际上应该让用户重新登录。这时候就需要结合 http 拦截器 + 后端接口返回的http 状态码来判断。

### 拦截器
要想统一处理所有http请求和响应，就得用上 axios 的拦截器。通过配置http response inteceptor，当后端接口返回401 Unauthorized（未授权），让用户重新登录。
```javascript
// http request 拦截器
axios.interceptors.request.use(
    config => {
        if (stoken) {  // 判断是否存在token，如果存在的话，则每个http header都加上token
            config.headers.Authorization = `token ${store.state.token}`;
        }
        return config;
    },
    err => {
        return Promise.reject(err);
    });

// http response 拦截器
axios.interceptors.response.use(
    response => {
        return response;
    },
    error => {
        if (error.response) {
            switch (error.response.status) {
                case 401:
                    // 返回 401 清除token信息并跳转到登录页面
                    
                    router.replace({
                        path: 'login',
                        query: {redirect: router.currentRoute.fullPath}
                    })
            }
        }
        return Promise.reject(error.response.data)   // 返回接口返回的错误信息
    });
```
