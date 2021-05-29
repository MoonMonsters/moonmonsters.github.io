title: Bug --- vue add axios报错
author: _Tao
tags:
  - vue
categories:
  - Bug
  - ''
date: 2021-05-02 20:08:00
---
### 报错
```text
🚀  Invoking generator for vue-cli-plugin-axios...
⠋  Running completion hooks...error: 'options' is defined but never used (no-unused-vars) at src/plugins/axios.js:42:32:
  40 | );
  41 |
> 42 | Plugin.install = function(Vue, options) {
     |                                ^
  43 |   Vue.axios = _axios;
  44 |   window.axios = _axios;
  45 |   Object.defineProperties(Vue.prototype, {
```

### 解决方案

原因:该项目安装了`eslint`规范，`ESLint` 是在 ECMAScript/JavaScript 代码中识别和报告模式匹配的工具，它的目标是保证代码的一致性和避免错误。但有时也会由于过于严谨,导致错误提醒
解决方案:
在package.json文件中添加如下代码
```json
"rules": {
	"generator-star-spacing": "off",
	"no-tabs":"off",
	"no-unused-vars":"off",
	"no-console":"off",
	"no-irregular-whitespace":"off",
	"no-debugger": "off"
}
```