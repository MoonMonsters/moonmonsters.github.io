title: '[转]Flask --- Flask + Vue 生成漂亮的词云'
author: _Tao
tags:
  - flask
  - ''
  - vue
  - ''
categories:
  - Python
date: 2021-07-24 20:00:00
---
### 转载
[Python的Web框架Flask + Vue 生成漂亮的词云](https://cloud.tencent.com/developer/article/1592758)


### 讲解
- 编写完前端代码后, 打包生成dist文件
- flask后端, 返回dist文件中的首页
- 完成了


### 前端代码
#### 插件
```html
npm i element-ui -S
npm install --save axios
```

#### App.vue
```html
<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>


```

#### pages/WordCloud.vue
```html
<template>
    <div>
        <h2>小词云</h2>
        <div id="word-text-area">
            <el-input type="textarea" :rows="10" placeholder="请输入内容" v-model="textarea">
            </el-input>
            <div id="word-img">
                <el-image :src="'data:image/png;base64,'+pic" :fit="fit">
                    <div slot="error" class="image-slot">
                        <i class="el-icon-picture-outline"></i>
                    </div>
                </el-image>
            </div>
            <div id="word-operation">
                <el-row>
                    <el-button type="primary" @click="onSubmit" round>生成词云</el-button>
                    <el-button type="success" @click="onDownload" round>下载图片</el-button>
                </el-row>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: 'wordcloud',
        data() {
            return {
                textarea: '',
                pic: "",
                pageTitle: 'Flask Vue Word Cloud',
            }
        },
        methods: {
            onSubmit() {
                var param = {
                    "word": this.textarea
                }
                this.axios.post("/word/cloud/generate", param).then(
                    res => {
                        this.pic = res.data
                        console.log(res.data)
                    }
                ).catch(res => {
                    console.log(res.data.res)
                })
            },
            onDownload() {
                const imgUrl = 'data:image/png;base64,' + this.pic
                const a = document.createElement('a')
                a.href = imgUrl
                a.setAttribute('download', 'word-cloud')
                a.click()
            }
        }
    }
</script>

<style scoped>

</style>

```


#### main.js
```javascript
import Vue from 'vue'
import App from './App'
import router from './router'
import ElementUI from 'element-ui'
import axios from 'axios'
import 'element-ui/lib/theme-chalk/index.css'

Vue.config.productionTip = false
Vue.use(ElementUI)
Vue.prototype.axios = axios

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  components: { App },
  template: '<App/>'
})

```

#### router/index.js
```javascript
import Vue from 'vue'
import Router from 'vue-router'
import WordCloud from '@/pages/WordCloud'

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'index',
      component: WordCloud
    }
  ]
})

```

### 后端代码
#### routers.py
```python
from flask import render_template
from flask import request
from wordcloud import WordCloud
import io
import base64

from flask import Flask

app = Flask(__name__, template_folder="../frontend/dist", static_folder="../frontend/dist/static")


# 真正调用词云库生成图片
def get_word_cloud(text):
    font = "backend/SimHei.ttf"
    # pil_img = WordCloud(width=500, height=500, font_path=font).generate(text=text).to_image()

    pil_img = WordCloud(width=800, height=300, background_color="white", font_path=font).generate(text=text).to_image()
    img = io.BytesIO()
    pil_img.save(img, "PNG")
    img.seek(0)
    img_base64 = base64.b64encode(img.getvalue()).decode()
    return img_base64


# 主页面
@app.route('/', endpoint="index")
@app.route('/index', endpoint="index")
def index():
    return render_template('index.html')


# 生成词云图片接口，以base64格式返回
@app.route('/word/cloud/generate', methods=["POST"], endpoint="cloud")
def cloud():
    text = request.json.get("word")
    res = get_word_cloud(text)
    return res


if __name__ == "__main__":
    app.run(port=8080)

```
