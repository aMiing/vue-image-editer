<template>
  <div id="dialog">
    <div class="toolBar">
      <fieldset style="float: left">
        <legend>工具</legend>
        <div class="toolsOptions">
          <span v-for="item in toolBar.tool" :key="item.name">
            <input
              type="radio"
              name="toolsOption"
              :id="'tools_' + item.name"
              @change="changeTool(item.name)"
              :checked="item.name == currentTool"
            />
            <label
              :for="'tools_' + item.name"
              :title="item.display"
              :class="item.name"
            ></label>
          </span>

          <span v-for="item in toolBar.shade" :key="item.name">
            <input
              type="radio"
              name="toolsOption"
              :id="'tools_' + item.name"
              @change="changeTool(item.name)"
              :checked="item.name == currentTool"
            />
            <label
              :for="'tools_' + item.name"
              :title="item.display"
              :class="item.name"
            ></label>
          </span>
        </div>
      </fieldset>
      <fieldset style="float: left; margin-left: 20px">
        <legend>操作</legend>
        <div class="toolsOptions">
          <label
            v-for="item in toolBar.opration"
            :key="item.name"
            :id="'tools_' + item.name"
            :class="item.name"
            @click="oprationClickHandle(item.name)"
          ></label>
          <span
            >边框色:
            <el-color-picker
              v-model="borderColor"
              size="mini"
              @change="borderColorChange"
            ></el-color-picker>
          </span>
          <span
            >文字颜色:
            <el-color-picker
              v-model="textColor"
              size="mini"
              @change="textColorChange"
            ></el-color-picker>
          </span>
        </div>
      </fieldset>

      <fieldset style="float: left; margin-left: 20px">
        <legend>文件操作</legend>
        <div class="toolsOptions">
          <label
            v-for="item in toolBar.fileOpration"
            :key="item.name"
            :id="'tools_' + item.name"
            @click="handleFileOpration(item.name)"
            :class="item.name"
          ></label>
        </div>
      </fieldset>

      <div style="clear: both"></div>
    </div>

    <div
      id="container"
      :style="
        'width:' + (canvasSize.w + 6) + 'px;height:' + (canvasSize.h + 6) + 'px'
      "
      @mousedown="mouseDownEventHandler"
      @mousemove="mouseMoveEventHandler"
      @mouseleave="mouseleave"
      @mouseup="mouseupEventHandler"
    >
      <canvas
        ref="myCanvas"
        id="myCanvas"
        :width="canvasSize.w"
        :height="canvasSize.h"
        :class="containerClass"
        style="position: absolute; top: 1px; left: 1px; background: #fff"
      >
      </canvas>
      <canvas
        ref="myCanvas1"
        id="myCanvas1"
        :width="canvasSize.w"
        :height="canvasSize.h"
        class="container_pencil"
        style="position: absolute; top: 1px; left: 1px"
        @mousemove="canvas1Mousemove"
        @mouseup="canvas1Mouseup"
      >
      </canvas>
      <div
        class="anyLine"
        v-show="anyLineShow"
        :style="
          'border-top: 1px solid ' +
          borderColor +
          ';width: ' +
          lineObj.length +
          'px;transform: rotate(' +
          lineObj.rotate +
          ');left: ' +
          lineObj.left +
          'px;top: ' +
          lineObj.top +
          'px'
        "
      ></div>
      <div
        class="circleDom"
        ref="circleDom"
        v-show="rectTipShow"
        :style="
          'border: 1px solid ' +
          borderColor +
          ';width: 1px;height: 1px;position: absolute;'
        "
      ></div>
      <div
        class="fontTip"
        v-show="fontTipObj.show"
        :style="
          'top:' +
          fontTipObj.top +
          'px;left:' +
          fontTipObj.left +
          'px;width:' +
          fontTipObj.width +
          'px;height:' +
          fontTipObj.height +
          'px;color: ' +
          fontTipObj.textColor
        "
      >
        <el-input
          type="textarea"
          ref="fontTip"
          autosize
          autofocus
          @blur="drawWords"
          v-model="textValue"
        ></el-input>
      </div>
      <img
        v-show="arrowImgShow"
        :src="arrowImg"
        class="arrowImg"
        height="38"
        :width="lineObj.length"
        :style="
          'top:' +
          lineObj.top +
          'px;left:' +
          lineObj.left +
          'px;transform: rotate(' +
          lineObj.rotate +
          ')'
        "
      />
    </div>
  </div>
</template>

<script>
const toolbar = {
  tool: [
    {
      name: "pencil",
      display: "画笔",
      icon: "images/pencil.svg",
    },
    {
      name: "eraser",
      display: "橡皮擦",
      icon: "images/eraser.svg",
    },
    {
      name: "clear",
      display: "清空",
      icon: "images/clear.svg",
    },
  ],
  opration: [
    {
      name: "undo",
      display: "undo",
      icon: "images/undo.svg",
    },
    {
      name: "redo",
      display: "redo",
      icon: "images/redo.svg",
    },
  ],
  shade: [
    {
      name: "line",
      display: "直线",
      icon: "images/line.svg",
    },
    {
      name: "circle",
      display: "圆形",
      icon: "images/circle.svg",
    },
    {
      name: "rect",
      display: "矩形",
      icon: "images/rect.svg",
    },
    {
      name: "arrow",
      display: "箭头",
      icon: "images/arrow.svg",
    },
    {
      name: "text",
      display: "文字",
      icon: "images/text.svg",
    },
  ],
  fileOpration: [
    {
      name: "uploadImg",
      display: "uploadImg",
      icon: "images/upload.svg",
    },
    {
      name: "save",
      display: "save",
      icon: "images/save.svg",
    },
  ],
};
let width = window.innerWidth - 20;
let height = window.innerHeight - 100;
export default {
  name: "app",
  props: {
    width: {
      type: Number,
      default: 0,
    },
    height: {
      type: Number,
      default: 0,
    },
  },
  data() {
    return {
      flag: false,
      rectTipShow: false,
      textValue: "",
      fontTipObj: {
        show: false,
        top: "0px",
        left: "0px",
        width: "0px",
        height: "0px",
      },
      canvasSize: {
        w: this.width || width,
        h: this.height || height,
      },
      containerClass: "container_pencil",
      toolBar: toolbar,
      currentTool: "pencil",
      cStep: -1, //记录步伐，用于回退前进
      history: [],
      borderColor: "#333",
      textColor: "#000",
      lineObj: {
        length: 1,
        left: 0,
        top: 0,
        rotate: "0deg",
      },
      anyLineShow: false,
      arrowImgShow: false,
      arrowImg: require("./assets/arrow.png"),
    };
  },
  mounted() {
    this.$nextTick(() => {
      this.canvas = this.$refs.myCanvas;
      this.canvas1 = this.$refs.myCanvas1;
      this.ctx = this.canvas.getContext("2d");
      this.ctx1 = this.canvas1.getContext("2d");

      this.canvasLeft = this.canvas.offsetParent.offsetLeft;
      this.canvasTop = this.canvas.offsetParent.offsetTop;

      this.rectTip = this.$refs.circleDom;
      this.fontTip = this.$refs.fontTip;
    });

    window.onresize = () => {
      this.canvasSize = {
        w: this.width || window.innerWidth - 20,
        h: this.height || window.innerHeight - 100
      }
      this.timer && clearTimeout(this.timer)
      this.timer = setTimeout(() => {
        this.cStep++
        this.undo()
      }, 0)
    };
  },
  methods: {
    //   sava\undo\redo
    oprationClickHandle(name) {
      if (name == "redo") {
        this.redo();
      } else if (name == "undo") {
        this.undo();
      }
    },
    //   绘制完成清空canvas1
    canvas1Mouseup() {
      this.clearCanvas(this.ctx1);
    },
    //   绘制圆的过程在canvas1上
    canvas1Mousemove() {
      if (this.currentTool === "circle" && this.flag) {
        this.rectTipShow = false;
        this.clearCanvas(this.ctx1);
        this.drawCircle(this.ctx1);
      }
    },
    textColorChange(color) {
      this.textColor = color;
    },
    borderColorChange(color) {
      this.borderColor = color;
    },
    changeTool(name) {
      console.log(name);
      this.currentTool = name;
      if (name === "clear") {
        this.clearCanvas(this.ctx);
        this.currentTool = "pencil";
      }
    },
    mouseDownEventHandler(e) {
      this.flag = true;
      this.beginX = e.pageX - this.canvas.offsetParent.offsetLeft;
      this.beginY = e.pageY - this.canvas.offsetParent.offsetTop;
      this.rectTip.style.border = this.borderColor + " " + "1px solid";
    },
    mouseupEventHandler(e) {
      this.flag = false;
      this.endX = e.pageX - this.canvasLeft;
      this.endY = e.pageY - this.canvasTop;

      // 用于撤销或重做的记录操作历史
      this.historyPush();

      switch (this.currentTool) {
        case "line": {
          this.drawline();
          break;
        }
        case "arrow": {
          this.drawArrow();
          break;
        }
        case "rect": {
          this.drawRectangle();
          break;
        }
        case "text": {
          this.fontTip.focus();
          break;
        }
        case "circle": {
          this.drawCircle(this.ctx);
          break;
        }
      }
    },
    mouseMoveEventHandler(e) {
      this.ctx.lineWidth = 1;
      switch (this.currentTool) {
        case "pencil": {
          this.drawPencil(e);
          break;
        }
        case "eraser": {
          this.ctx.lineWidth = 15;
          this.drawEraser(e);

          break;
        }
        case "line": {
          this.fakeLineInput(e);
          break;
        }
        case "arrow": {
          this.drawArrow(e);
          break;
        }
        case "rect": {
          this.fakeRectangleInput(e);
          break;
        }
        case "text": {
          this.fakeWordsInput(e);
          break;
        }
        case "circle": {
          this.fakeRectangleInput(e);
          break;
        }
      }
    },
    mouseleave(e) {
      console.log("mouseleave", e);
    },
    //在画布上画不规则的线
    drawRandomLine(e) {
      //按下鼠标时
      var x = e.pageX - this.canvasLeft;
      var y = e.pageY - this.canvasTop;
      if (this.flag) {
        this.ctx.lineTo(x, y);
        this.ctx.stroke();
      } else {
        this.ctx.beginPath();
        this.ctx.moveTo(x, y);
      }
    },
    drawPencil(e) {
      (this.containerClass = "container_pencil"),
        (this.ctx.strokeStyle = this.borderColor);
      this.drawRandomLine(e);
    },
    drawEraser(e) {
      (this.containerClass = "container_eraser"), (this.ctx.lineCap = "round");
      this.ctx.fillStyle = this.borderColor;
      this.ctx.strokeStyle = this.borderColor;
      this.drawRandomLine(e);
    },
    /**
     * 使用文字工具时，
     * 当鼠标被按下时，可以在画布上拖动一行
     */
    fakeWordsInput(e) {
      const top = e.pageY - this.canvasTop;
      const left = e.pageX - this.canvasLeft;
      if (this.flag) {
        this.fontTipObj = {
          show: true,
          top: this.beginY,
          left: this.beginX,
          width: left - this.beginX,
          height: top - this.beginY,
        };
      }
    },
    fakeRectangleInput(e) {
      const left = this.canvasLeft;
      const top = this.canvasTop;
      this.endX = e.pageX - left;
      this.endY = e.pageY - top;

      var borderWidth = 1;
      if (this.flag) {
        this.currentTool === "rect" && (this.rectTipShow = true);
        const x = this.beginX;
        const y = this.beginY;
        let left, top;
        if (this.endX - x < 0 && this.endY - y < 0) {
          left = x - Math.abs(this.endX - x);
          top = y - Math.abs(this.endY - y);
        } else if (this.endX - x < 0) {
          left = x - Math.abs(this.endX - x);
          top = y;
        } else if (this.endY - y < 0) {
          left = x;
          top = y - Math.abs(this.endY - y);
        } else {
          left = x;
          top = y;
        }
        this.rectTip.style.left = left + "px";
        this.rectTip.style.top = top + "px";
        this.rectTip.style.width = Math.abs(this.endX - x) - borderWidth + "px";
        this.rectTip.style.height =
          Math.abs(this.endY - y) - borderWidth + "px";
      }
    },

    /**
     * 画线
     */
    fakeLineInput(e) {
      if (this.flag) {
        this.endX = e.pageX - this.canvasLeft;
        this.endY = e.pageY - this.canvasTop;
        const { degree, x, y, z } = this.getDegreeOfTowPoints(
          { x: this.beginX, y: this.beginY },
          { x: this.endX, y: this.endY }
        );
        this.degree = degree; //两个点的角度
        this.lineObj = {
          length: z,
          left: this.beginX + x,
          top: this.beginY + y,
          rotate: degree,
        };
        if (this.currentTool == "arrow") {
          this.arrowImgShow = true;
        } else if (this.currentTool == "line") {
          this.anyLineShow = true;
        }
      }
    },

    /**
     * 清空画布上的内容
     */
    clearCanvas(ctx) {
      ctx.clearRect(0, 0, this.canvasSize.w, this.canvasSize.h);
    },
    /**
     * 画线的方法
     */
    drawline() {
      this.ctx.beginPath();
      this.ctx.moveTo(this.beginX, this.beginY);
      this.ctx.lineTo(this.endX, this.endY);
      this.ctx.stroke();
      this.anyLineShow = false;
    },
    // 画箭头
    drawArrow(e) {
      const ctx = e ? this.ctx1 : this.ctx;
      if (e) {
        if (!this.flag) return false;
        const left = this.canvasLeft;
        const top = this.canvasTop;
        this.endX = e.pageX - left;
        this.endY = e.pageY - top;
        this.clearCanvas(ctx);
      }
      let lineWidth = 10,
        arrowLength = 30;
      this.drawArrowFun(
        ctx,
        this.beginX,
        this.beginY,
        this.endX,
        this.endY,
        30,
        arrowLength,
        lineWidth,
        "#f00"
      );
    },

    drawArrowFun(ctx, fromX, fromY, toX, toY, theta, headlen, width, color) {
      theta = typeof theta != "undefined" ? theta : 30;
      headlen = typeof theta != "undefined" ? headlen : 10;
      width = typeof width != "undefined" ? width : 1;
      color = typeof color == "string" ? color : "#000";
      if (fromX == toX && fromY == toY) {
        return false;
      }

      const length = Math.sqrt(
        Math.pow(fromY - toY, 2) + Math.pow(fromX - toX, 2)
      );
      headlen = Math.min(headlen, length / 2);
      width = Math.min(width, length / 4);
      // 计算各角度和对应的P2,P3坐标
      var angle = (Math.atan2(fromY - toY, fromX - toX) * 180) / Math.PI,
        angle1 = ((angle + theta) * Math.PI) / 180,
        angle2 = ((angle - theta) * Math.PI) / 180,
        topX = headlen * Math.cos(angle1),
        topY = headlen * Math.sin(angle1),
        botX = headlen * Math.cos(angle2),
        botY = headlen * Math.sin(angle2);
      // 找两个拐点
      var tempX1 =
        ((Math.cos(((angle + theta / 2) * Math.PI) / 180) * 3) / 4) * headlen +
        toX;
      var tempX2 =
        ((Math.cos(((angle - theta / 2) * Math.PI) / 180) * 3) / 4) * headlen +
        toX;
      var tempY1 =
        ((Math.sin(((angle + theta / 2) * Math.PI) / 180) * 3) / 4) * headlen +
        toY;
      var tempY2 =
        ((Math.sin(((angle - theta / 2) * Math.PI) / 180) * 3) / 4) * headlen +
        toY;

      ctx.save();
      ctx.beginPath();

      let arrowX = toX + topX;
      let arrowY = toY + topY;
      // ctx.arc(fromX, fromY, Math.abs(Math.floor(topY/24)), 0 , 2*Math.PI);
      ctx.moveTo(fromX + topX / 15, fromY + topY / 15); //point1
      ctx.lineTo(tempX1, tempY1); //point2
      ctx.lineTo(arrowX, arrowY); //point3

      ctx.lineTo(toX, toY); //point4

      const arrowX2 = toX + botX;
      const arrowY2 = toY + botY;
      ctx.lineTo(arrowX2, arrowY2); //point5

      ctx.lineTo(tempX2, tempY2); //point6

      ctx.lineTo(fromX + botX / 15, fromY + botY / 15); //point8

      ctx.lineTo(fromX + topX / 15, fromY + topY / 15); //point9

      ctx.fillStyle = color;
      ctx.fill();
      ctx.restore();
    },

    /**
     * 画矩形的方法e
     */
    drawRectangle() {
      // var borderWidth = 1;
      this.ctx.strokeStyle = this.borderColor;
      this.ctx.beginPath();
      this.ctx.strokeRect(
        this.beginX - 1,
        this.beginY - 1,
        this.endX - this.beginX,
        this.endY - this.beginY
      );
      this.rectTip.style.display = "none";

      this.rectTipShow = false;

      this.rectTip.style.width = "0px";
      this.rectTip.style.Height = "0px";
      this.rectTip.style.top = "0px";
      this.rectTip.style.left = "0px";
    },

    /**
     * 画圆形的方法（贝塞尔曲线）
     */

    drawCircle(context) {
      var k = (this.endX - this.beginX) / 0.75 / 2,
        // w = (this.endX - this.beginX) / 2,
        h = (this.endY - this.beginY) / 2,
        x = (this.endX + this.beginX) / 2,
        y = (this.endY + this.beginY) / 2;
      if (context) {
        context.strokeStyle = this.borderColor;
        context.beginPath();
        context.moveTo(x, y - h);
        context.bezierCurveTo(x + k, y - h, x + k, y + h, x, y + h);
        context.bezierCurveTo(x - k, y + h, x - k, y - h, x, y - h);
        context.closePath();
        context.stroke();
      }
    },
    /**
     * 输入文字的方法
     */
    drawWords() {
      var words = this.textValue.split(/\s/);
      console.log(words);
      if (words.length && words[0]) {
        this.ctx.strokeStyle = this.textColor;
        this.ctx.fillStyle = this.textColor;
        // var fontSize = 14;
        this.ctx.font = "14px 宋体";
        words.forEach((item, index) => {
          this.ctx.fillText(
            item,
            this.fontTipObj.left,
            this.fontTipObj.top + 14 + 1 + index * 19
          );
        });

        this.textValue = "";
      }
      this.fontTipObj.show = false;
    },
    /**
     * 记录当前画布（相当于缓存）
     */
    historyPush() {
      this.cStep++;
      if (this.cStep < this.history.length) {
        this.history.length = this.cStep;
      }

      this.history.push(this.canvas.toDataURL());
      console.log("history", this.history.length);
      console.log("cstep", this.cStep);
    },
    /**
     * 撤销
     */
    undo() {
      if (this.cStep >= 0) {
        this.clearCanvas(this.ctx);
        this.cStep--;
        var tempImage = new Image();
        tempImage.src = this.history[this.cStep];
        tempImage.onload = () => {
          this.ctx.drawImage(tempImage, 0, 0);
        };
      }
    },

    /**
     * 重做
     */
    redo() {
      if (this.cStep < this.history.length - 1) {
        this.clearCanvas(this.ctx);
        this.cStep++;
        var tempImage = new Image();
        tempImage.src = this.history[this.cStep];
        tempImage.onload = () => {
          this.ctx.drawImage(tempImage, 0, 0);
        };
      }
    },

    // 计算两点的旋转角度
    getDegreeOfTowPoints(p1, p2) {
      var data = {};
      var width = p2.x - p1.x;
      var height = p2.y - p1.y;
      var arctan = Math.abs(height / width);
      var arc = Math.atan(arctan);
      let x = 0,
        y = 0;

      var angle = (arc / Math.PI) * 180;

      data.z = Math.sqrt(Math.pow(height, 2) + Math.pow(width, 2));

      if (width > 0 && height > 0) {
        y = (data.z * Math.sin(arc)) / 2;
        x = -(data.z - data.z * Math.cos(arc)) / 2;
        data.degree = angle;
      } else if (width < 0 && height > 0) {
        x = -(data.z + data.z * Math.cos(arc)) / 2;
        y = (data.z * Math.sin(arc)) / 2;
        data.degree = 180 - angle;
      } else if (width < 0 && height < 0) {
        x = -(data.z + data.z * Math.cos(arc)) / 2;
        y = -(data.z * Math.sin(arc)) / 2;
        data.degree = 180 + angle;
      } else if (width > 0 && height < 0) {
        x = -(data.z - data.z * Math.cos(arc)) / 2;
        y = -(data.z * Math.sin(arc)) / 2;
        data.degree = -angle;
      }

      return { degree: (data.degree || 0) + "deg", x, y, z: data.z };
    },
    // 文件操作
    handleFileOpration(name) {
      if (name == "uploadImg") {
        this.uploadImg();
      } else if (name == "save") {
        this.saveItAsImage();
      }
    },

    saveItAsImage() {
      var image = this.canvas.toDataURL();
      //保存到本地
      const oa = document.createElement("a");
      oa.href = image;
      oa.download = "图片下载";
      oa.click();
    },
    /**
     *
     * 本地上传图片
     */
    uploadImg() {
      var inputObj = document.createElement("input");
      inputObj.type = "file";
      inputObj.accept = "image/*";
      const _this = this;
      inputObj.addEventListener(
        "change",
        function () {
          var file = this.files[0]; //获取input输入的图片
          if (!/image\/\w+/.test(file.type)) {
            alert("请确保文件为图像类型");
            return false;
          } //判断是否图片，在移动端由于浏览器对调用file类型处理不同，虽然加了accept = 'image/*'，但是还要再次判断
          var reader = new FileReader();
          reader.readAsDataURL(file); //转化成base64数据类型
          reader.onload = function () {
            _this.checkImage(
              _this.canvasSize.w - 20,
              _this.canvasSize.h - 20,
              this.result,
              _this
            );
          };
        },
        false
      );
      // inputObj.id = id;
      inputObj.click();
    },
    /*修改图片尺寸*/
    checkImage(maxWidth, maxHeight, imgData, _this) {
      var img = new Image();
      img.src = imgData;
      img.onload = function () {
        var hRatio;
        var wRatio;
        var Ratio = 1;
        var w = img.width;
        var h = img.height;
        wRatio = maxWidth / w; /*画布除原  >1 不缩放*/
        hRatio = maxHeight / h; /*小于1 缩放*/
        if (wRatio > 1 && hRatio > 1) {
          Ratio = 1;
        } else if (wRatio > 1 && hRatio < 1) {
          Ratio = hRatio;
        } else if (wRatio < 1 && hRatio > 1) {
          Ratio = wRatio;
        } else if (wRatio < 1 && hRatio < 1) {
          if (wRatio > hRatio) {
            Ratio = hRatio;
          } else Ratio = wRatio;
        }
        w = w * Ratio;
        h = h * Ratio;
        img.height = h;
        img.width = w;
        _this.ctx.drawImage(img, 10, 10, w, h);
        _this.historyPush();
      };
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
#container {
  position: relative;
  overflow: hidden;
}
.toolBar {
  display: flex;
  justify-content: center;
  vertical-align: middle;
}
input[type="radio"] {
  width: 0px;
  height: 0px;
  margin: 0;
}
input:checked {
  ~ label {
    background-color: #66b1ff;
    border-color: #66b1ff;
    color: #fff;
  }
}
.toolsOptions {
  display: flex;
  justify-content: space-between;
  height: 28px;
  line-height: 28px;
  span {
    display: inline-flex;
    line-height: 28px;
    margin: 0 0.4em;
  }
}
.toolsOptions label {
  border: 1px solid #6bf;
  font-weight: normal;
  color: #ffffff;
  border-radius: 5px;
  // padding: .3em 1em;
  line-height: 1.6em;
  cursor: pointer;
  margin-right: 0.4em;
  color: #409eff;
  background-color: #ecf5ff;
  border-color: #b3d8ff;
  height: 28px;
  width: 36px;
  background-position: 50% 50%;
  background-repeat: no-repeat;
  background-size: 80% 80%;
  &.line {
    background-image: url("./assets/line.svg");
  }
  &.pencil {
    background-image: url("./assets/pencil.svg");
  }

  &.eraser {
    background-image: url("./assets/eraser.svg");
  }
  &.circle {
    background-image: url("./assets/circle.svg");
  }
  &.rect {
    background-image: url("./assets/rect.svg");
  }
  &.undo {
    background-image: url("./assets/undo.svg");
  }
  &.redo {
    background-image: url("./assets/redo.svg");
  }
  &.save {
    background-image: url("./assets/save.svg");
  }
  &.clear {
    background-image: url("./assets/clear.svg");
  }
  &.text {
    background-image: url("./assets/text.svg");
  }
  &.uploadImg {
    background-image: url("./assets/uploadImg.svg");
  }
  &.arrow {
    background-image: url("./assets/arrow.svg");
  }
}
canvas {
  border: 2px dashed gray;
  // margin: 0 202px;
}

.container_pencil {
  cursor: url(./assets/PencilToolCursor.gif), pointer;
}

.container_eraser {
  cursor: url(./assets/test.png), pointer;
}

.container_font {
  cursor: crosshair;
}
.speed {
  top: 15px;
}

.ui-selectmenu-text {
  font-size: 14px;
}
.anyLine {
  position: absolute;
  top: auto;
  left: auto;
}
.fontTip {
  background: transparent;
  position: absolute;
  // overflow: hidden;
}
.arrowImg {
  position: absolute;
  margin-top: -17px;
}
</style>
