
<template>
  <div class="mdui-bottom-nav-fixed mdui-theme-primary-indigo mdui-theme-accent-pink mdui-loaded">
    <div class="eraser hidden"></div>
    <canvas class="canvas" id="thecanvas" width="400" height="250"></canvas>

    <div class="mdui-bottom-nav" style="overflow: inherit;">
      <a href="javascript:;" @click="setAttr()" class="mdui-ripple">
        <label>清空画布</label>
      </a>
      <span>候选字：{{itemstring}}</span>
    </div>

    <div id="beforeWrapper" class="mdui-dialog-content" style="background-color: rgb(251, 251, 251); min-height: 200px;">
      <ul>
        <li v-for="(item, index) in imageList" :key="index">
          <img @click="goback(item)" mdui-dialog-close class='mdui-shadow-7' :src='item'>
        </li>
      </ul>
    </div>
    <button class="mdui-btn mdui-ripple" @click="beforeListClear()" mdui-dialog-close="">清空历史</button>
    <button class="mdui-btn mdui-ripple" mdui-dialog-close="">取消</button>
  </div>
</template>
<script type="text/javascript">
import axios from 'axios'
import md5 from './utils/md5.min'
export default {
  data() {
    return {
      startX: '',
      startY: '',
      drawing: true, 
      endX: '',
      endY: true,
      width: 5,
      paint: "",
      color: 'black',
      canvas: '',
      isDown: false,
      imageList: [],
      itemstring: ''
    }
  },
  mounted() {
    //画笔
    var self = this
    self.canvas = document.getElementsByTagName("canvas")[0]
    self.canvas.addEventListener('touchstart', this.touchstart, false); 
    self.canvas.addEventListener('touchmove', this.touchmove, false); 
    self.canvas.addEventListener('touchend', this.touchend, false); 
    self.canvas.addEventListener('mousemove', this.onMouseMove, false); 
    self.canvas.addEventListener('mousedown', this.onMouseDown, false); 
    self.canvas.addEventListener('mouseup', this.onMouseUp, false); 
    $(function () {
      self.setAttr()
    })
    //检测是否是移动端
    // function goPAGE() {
    //     if (!(navigator.userAgent.match(/(phone|pad|pod|iPhone|iPod|ios|iPad|Android|Mobile|BlackBerry|IEMobile|MQQBrowser|JUC|Fennec|wOSBrowser|BrowserNG|WebOS|Symbian|Windows Phone)/i))) {
    //         alert("请在移动端查看（或者开发者模式）");
    //     }
    // }
  },
  methods:{
    //基本设置
    setAttr() {
      // this.canvas.width = document.documentElement.clientWidth;
      // this.canvas.height = document.documentElement.clientHeight;
      this.paint = this.canvas.getContext("2d");
      //背景色
      this.paint.fillStyle = "#fff";
      this.paint.fillRect(0,0, this.canvas.width, this.canvas.height);
    },
    handwriting (img) {
      // debugger  腾讯
      let self = this
      var crypto = require('crypto')
      /* eslint-disable */
      var secretId  = 'AKIDMZYJqAk6j7IFySLma9TZR4UgeN0XcZ3P',
          secretKey = 'HMCnDNIyPs9Bzc1DAtPgiU4asvjJekbR',
          appid     = '1257270260',
          pexpired  = 86400,
          userid   = 0;

      var now = parseInt(Date.now() / 1000),
          rdm = parseInt(Math.random() * Math.pow(2, 32)),
          plainText = 'a=' + appid + '&k=' + secretId + '&e=' + ( now + pexpired) + '&t=' + now + '&r=' + rdm + userid + '&f=',
          data = new Buffer(plainText,'utf8'),
          res = crypto.createHmac('sha1', secretKey).update(data).digest(),
          bin = Buffer.concat([res,data])

      var sign = bin.toString('base64')
      // debugger
      axios({
        method: 'post',
        // baseURL: 'https://recognition.image.myqcloud.com',
        url: 'https://recognition.image.myqcloud.com/ocr/handwriting',
        // url: 'https://aip.baidubce.com/rest/2.0/ocr/v1/handwriting?access_token=24.a70880b79f72a0a031ef6cd7bded1336.2592000.1542357753.282335-14462587',  // 百度api，客户端可使用
        headers: {
          Authorization: sign,
          // 'Content-type': 'multipart/form-data' // 百度请求头
        },
        data: {
          url: self.$config.filePath + img,
          // image: img // 百度 图片传参 （）
          appid: "1257270260",
          bucket: "test",
        }
      }).then(data => {
        // debugger
        if (data.message === 'OK') {
          self.itemstring = data.data.items[0].itemstring
        } else {
          self.itemstring = '无法识别'
        }
      })
    },
    onMouseMove(event) {
      var self = this
      event.preventDefault();
      self.endX = event.offsetX;
      self.endY = event.offsetY;
      if (self.drawing&&self.isDown) { //绘图
        //画下线段
        self.paint.beginPath();
        self.paint.moveTo(self.startX, self.startY);
        self.paint.lineTo(self.endX, self.endY);
        self.paint.closePath();
        //动态的设置颜色
        self.paint.strokeStyle = self.color;
        self.paint.lineWidth = self.width;
        self.paint.stroke();
      } else if (self.isDown) {
        //橡皮擦
        $(".eraser").css({
          "top": self.endY - $(".eraser").height() * 0.5 + "px",
          "left": self.endX - $(".eraser").width() * 0.5 + "px"
        });
        self.paint.clearRect($(".eraser").offset().left, $(".eraser").offset().top, $(".eraser").width(), $(
          ".eraser").height());
      }

      self.startX = self.endX;
      self.startY = self.endY;
    },
    onMouseDown(event) {
      var self = this
      self.isDown = true
      self.startX = event.offsetX;
      self.startY = event.offsetY;
      //如果处于檫的状态
      if (!self.drawing) {
        //显示橡皮檫那div
        $(".eraser").show();
        $(".eraser").css({
          "top": self.startY - $(".eraser").height() * 0.5 + "px",
          "left": self.startX - $(".eraser").width() * 0.5 + "px"
        });
        //从哪个点开始清理的宽度，高度
        self.paint.clearRect($(".eraser").offset().left, $(".eraser").offset().top, $(".eraser").width(), $(
          ".eraser").height());
      }
    },
    onMouseUp(event) {
      this.isDown = false
      this.stackImgs();
      $(".eraser").hide();
    },
    touchstart (event) {
      var self = this
      self.startX = event.targetTouches[0].clientX - $('#thecanvas').offset().left;
      self.startY = event.targetTouches[0].clientY - $('#thecanvas').offset().top;
      //如果处于檫的状态
      if (!self.drawing) {
        //显示橡皮檫那div
        $(".eraser").show();
        $(".eraser").css({
          "top": self.startY - $(".eraser").height() * 0.5 + "px",
          "left": self.startX - $(".eraser").width() * 0.5 + "px"
        });
        //从哪个点开始清理的宽度，高度
        self.paint.clearRect($(".eraser").offset().left, $(".eraser").offset().top, $(".eraser").width(), $(
          ".eraser").height());
      }
    },
    touchmove (event) {
      var self = this
      event.preventDefault();
      self.endX = event.targetTouches[0].clientX - $('#thecanvas').offset().left;
      self.endY = event.targetTouches[0].clientY - $('#thecanvas').offset().top;
      if (self.drawing) { //绘图
        //画下线段
        self.paint.beginPath();
        self.paint.moveTo(self.startX, self.startY);
        self.paint.lineTo(self.endX, self.endY);
        self.paint.closePath();
        //动态的设置颜色
        self.paint.strokeStyle = self.color;
        self.paint.lineWidth = self.width;
        self.paint.stroke();
      } else {
        //橡皮擦
        $(".eraser").css({
          "top": self.endY - $(".eraser").height() * 0.5 + "px",
          "left": self.endX - $(".eraser").width() * 0.5 + "px"
        });
        self.paint.clearRect($(".eraser").offset().left, $(".eraser").offset().top, $(".eraser").width(), $(
          ".eraser").height());
      }

      self.startX = self.endX;
      self.startY = self.endY;
    },
    touchend (event) {
      this.stackImgs();
      $(".eraser").hide();
    },
    //点击历史操作中的图片
    goback(image) {
      let self = this
      self.setAttr();
      let img = new Image();
      img.src = image;
      img.onload = function () {
        self.paint.drawImage(img, 0, 0);
      }
    },
    //每次手离开 向历史操作里推送本次绘图
    stackImgs() {
      var mycanvas = document.getElementById("thecanvas");
      var image = mycanvas.toDataURL("image/png");
      var imageu8 = this.dataURLtoBlob(image)
      // var image1 = image.replace('data:image/png;base64,','')
      this.uploadFile (imageu8)
      this.imageList.push(image)
      // $("#beforeList").append("<li><img mdui-dialog-close class='mdui-shadow-7' @click=\"goback('" + image +
      //   "')\" src='" + image + "'></li>");
    },
    toHexString (data) {
      const array = new Uint8Array(data)
      let hex = ''
      for (let i = 0; i < array.length; i++) {
        const c = array[i].toString('16')
        hex += c.length === 1 ? `0${c}` : c
      }
      return hex
    },
    uploadFile (file) {
      let self = this
      axios.post(self.$config.fileServer + '/api/upload/' + self.toHexString(md5.arrayBuffer(file)) + '/png', file, {
        headers: {
          Authorization: self.$config.fileServerAuthor
        }
      }).then ( data => {
        self.handwriting(data)
      })
    },
    dataURLtoBlob(dataurl) {
      var arr = dataurl.split(','),
      mime = arr[0].match(/:(.*?);/)[1],
      bstr = atob(arr[1]),
      n = bstr.length,
      u8arr = new Uint8Array(n);
      while (n--) {
          u8arr[n] = bstr.charCodeAt(n);
      }
      return u8arr
    },
    //清空历史记录
    beforeListClear() {
      $("#beforeList").empty();
    }
  }
}
</script>