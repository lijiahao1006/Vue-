<template>
  <div>
    <div class="wrap">
      <!-- 该盒子用于 画布 mark盒子 事件冒泡 -->
      <div
        @mousedown="downHan"
        @mouseup="upHan"
        @mousemove="moveHan" >
        <!-- 画布 -->
        <canvas
          :width="`${CanW}px`"
          :height="`${CanH}px`"
          ref="canvas"
        ></canvas>
        <!-- 遮罩层 -->
        <div
          class="mark"
          :style="{
            width:`${MarkW}px`,
            height:`${MarkH}px`,
            left:`${markLeft}px`,
            top:`${markTop}px`
          }"
          v-show="ISMARK"
        ></div>
        <!-- 预览截取后的画布 -->
        <canvas
          v-show="isShow"
          class="canvas2"
          width="220px"
          height="220px"
          ref="canvas2"
        ></canvas>
      </div>
      <!-- 按钮组 -->
      <div class="btnbox">
        <Button @click="fileHan">选择文件</Button>
        <input
          type="file"
          ref="file"
          hidden
          @change="changeIpt"
        >
        <Button @click="scaleHan(1)" >放大</Button>
        <Button @click="scaleHan(0)" >缩小</Button>
        <Button @click="saveHan">保存</Button>
      </div>
    </div>
  </div>
</template>

<script>
import _ from 'lodash';

export default {
  props: {
    // 画布 大小
    CanW: {
      type: Number,
      default: 420,
    },
    CanH: {
      type: Number,
      default: 420,
    },
    // mark 大小
    MarkH: {
      type: Number,
      default: 220,
    },
    MarkW: {
      type: Number,
      default: 220,
    },
    isShow: {
      type: Boolean,
      default: true,
    },
    imgtype: {
      type: String,
      default: 'png',
    },
  },
  data() {
    console.log(this);
    return {
      ISMARK: true,
      // 图片大小
      IMGW: 0,
      IMGH: 0,
      IMGL: 0,
      IMGR: 0,
      IMG: '',
      ctx: '',
      ctx2: '',
      startX: 0,
      startY: 0,
      flag: false,
      changeX: 0,
      changeY: 0,
    };
  },
  computed: {
    // 遮罩层位置
    markLeft() {
      return (this.CanW - this.MarkW) / 2;
    },
    markTop() {
      return (this.CanH - this.MarkH) / 2;
    },
  },
  methods: {
    // 代理input事件
    fileHan() {
      this.$refs.file.click();
    },
    // 转码 确定图片大小 定位图片
    changeIpt(e) {
      const file = e.target.files[0];
      // 判断用户是否真正上传图片
      if (file) {
        // 转码base 64
        const filereader = new FileReader();
        filereader.readAsDataURL(file);
        filereader.onload = (ee) => {
          this.IMG = new Image();
          this.IMG.src = ee.target.result;
          this.IMG.onload = () => {
            // 确定图片大小
            this.IMGW = this.IMG.width;
            this.IMGH = this.IMG.height;
            // 让图片宽或高 最长也不超不过 画布宽
            if (this.IMGH > this.IMGW) {
              const n = this.IMGH / this.CanH;
              this.IMGH = this.CanH;
              const w = this.IMGW;
              this.IMGW = w / n;
            } else {
              const x = this.IMGW / this.CanW;
              this.IMGW = this.CanW;
              const h = this.IMGH;
              this.IMGH = h / x;
            }
            // 图片在画布中 位置
            this.IMGL = (this.CanW - this.IMGW) / 2;
            this.IMGT = (this.CanH - this.IMGH) / 2;
            // 绘制图片
            this.drawImg();
          };
        };
      }
    },
    // 绘制画布
    drawImg() {
      this.ctx = this.$refs.canvas.getContext('2d');
      // 每次绘制之前清空画布
      this.ctx.clearRect(0, 0, this.CanW, this.CanH);
      this.ctx.drawImage(
        this.IMG, this.IMGL, this.IMGT, this.IMGW, this.IMGH,
      );
    },
    // 缩放按钮
    scaleHan(sign) {
      // 保证图片存在
      if (!this.IMG) return;
      // 计算图片宽高比
      const n = this.IMGW / this.IMGH;
      if (sign) {
        this.IMGW += 20;
        // 宽 除以 n = 高，等比变化
        this.IMGH = this.IMGW / n;
      } else {
        this.IMGW -= 20;
        this.IMGH = this.IMGW / n;
      }
      this.drawImg();
    },
    // 按下鼠标
    downHan(ev) {
      // 保证图片存在
      if (!this.IMG) return;
      this.flag = true;
      this.startX = ev.clientX;
      this.startY = ev.clientY;
    },
    // 抬起鼠标
    upHan() {
      this.flag = false;
    },
    // 移动鼠标(使用了loadsh中的函数节流)
    moveHan: _.throttle(function c(e) {
      if (this.flag) {
        // 鼠标移动的距离
        this.changeX = e.clientX - this.startX;
        this.changeY = e.clientY - this.startY;
        this.IMGL += this.changeX;
        this.IMGT += this.changeY;
        this.drawImg();
        // 重新赋值初始位置，进行下一次移动计算
        this.startX = e.clientX;
        this.startY = e.clientY;
      }
    }, 60),
    // 保存图片
    saveHan() {
      // 保证图片存在
      if (!this.IMG) return;
      // 获取mark阴影下的图片
      const imageData = this.ctx.getImageData(this.markLeft, this.markTop, this.MarkW, this.MarkH);
      this.ctx2 = this.$refs.canvas2.getContext('2d');
      // 把像素码放到画布中
      this.ctx2.putImageData(imageData, 0, 0, 0, 0, this.MarkW, this.MarkH);
      // 把画布变成一张图片
      this.$refs.canvas2.toDataURL('png');
      // base64 传递给父组件
      this.$emit('saveImg', this.$refs.canvas2.toDataURL(this.imgtype));
    },
  },
};
</script>

<style lang="scss" scoped>
.wrap{
  position: relative;
  canvas{
    position: relative;
    border: 1px solid black;
  }
  .mark{
    position: absolute;
    background-color: rgba(0, 0, 0, 0.199);
  }
  .canvas2{
    margin-left: 5px;
  }
  .btnbox{
    width: 500px;
  }
}
</style>
