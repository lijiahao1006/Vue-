# Vue-公共组件库
图片裁切
## 基于canvas实现图片裁切功能

### 配合 loadsh库 一起使用
```
npm i --save lodash
```
封装组件内已经引入loadsh，仅需安装loadsh即可。
loadsh->(throttle函数)用于mousemove函数的节流。

### 可传入的参数
CanW：画布宽
CanH：画布高
MarkW：遮罩层宽
MarkH：遮罩层高
isShow：是否开启预览画布（默认true）
@saveImg: 自定义事件 可获取截取图片的base64码 ；
例：
```
<clipimage :CanW="600" :CanH="700" @saveImg="savebase" />
```