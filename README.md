# Vue-公共组件库
图片裁切
## 基于canvas实现图片裁切功能
在组件中引入
```
import Clipimage from './ClipImage.vue';
```
例：
```
<clipimage @saveImg="savebase" />
```
```
export default {
  components: {
    Clipimage,
  },
  methods: {
    savebase(e) {
      console.log(e);
    },
  },
};
```
### 配合 loadsh库 一起使用
```
npm i --save lodash
```
封装组件内已经引入loadsh，仅需安装loadsh即可。
loadsh->(throttle函数)用于mousemove函数的节流。

### 可传入的参数
CanW：canvas宽（默认420）   
CanH: canvas高（默认420）    
MarkW：遮罩层宽（默认220）   
MarkH：遮罩层高（默认220）     
@saveImg: 自定义事件 参数为截取图片base64码    
imgtype: 截取图片类型（默认png）     
isShow：是否开启预览的小画布（默认true）
```
<clipimage :CanW="600" :MarkW="300" :isShow="false" :imgtype="jpg" @saveImg="savebase" />
```
