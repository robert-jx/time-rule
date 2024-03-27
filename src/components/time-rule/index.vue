<template>
  <div class="time-rule" id="time-rule">
    <canvas id="is-select-time" :width="ruleWidth" height="50"></canvas>
    <canvas id="time-line" :width="ruleWidth" height="50"></canvas>
    <canvas id="time-line-layer" :width="ruleWidth" height="50"></canvas>
  </div>
</template>

<script>
export default {
  name: "time-rule",
  props: {
    videoRecords: {
      type: Object,
      default: () => {},
    },
  },
  data() {
    return {
      ruleWidth: 1000,
      type: 1, // 1-月份，2-天，3-时,4-分
      currentTime: parseInt(new Date().getTime() / 1000),
      allowDarw: false,
    };
  },
  watch: {
    type: {
      handler(val) {
        this.setRule(this.currentTime);
      },
    },
    currentTime: {
      handler(val) {
        if (this.allowDarw) {
          this.timeChange(val);
        }
      },
    },
  },
  created() {
    this.init();
  },
  mounted() {
    let div = document.getElementById("time-rule");
    if (div.addEventListener) {
      div.addEventListener("DOMMouseScroll", this.changeType, false);
    } //W3C
    //滚动滑轮触发scrollFunc方法
    div.onmousewheel = div.onmousewheel = this.changeType;
    // this.setRule(this.currentTime);
    this.timeChange(this.currentTime);
    this.addCanvasListener();
  },
  methods: {
    init() {
      this.$nextTick(() => {
        this.ruleWidth = document.getElementById("time-rule").offsetWidth;
      });
    },
    // 鼠标滚轮滚动更改刻度单位
    changeType(e) {
      e = e || window.event;
      if (e.wheelDelta) {
        //IE/Opera/Chrome
        // +120表示向上；-120表示向下
        if (e.wheelDelta == 120) {
          if (this.type != 1) {
            this.type--;
          }
        } else {
          if (this.type != 4) {
            this.type++;
          }
        }
      } else if (e.detail) {
        //Firefox滑轮事件
        // +3表示向上；-3表示向下
        if (e.detail == 3) {
          if (this.type != 1) {
            this.type--;
          }
        } else {
          if (this.type != 4) {
            this.type++;
          }
        }
      }
    },
    setRule(currentTime) {
      let canvasId = document.getElementById("time-line");
      let ctx = canvasId.getContext("2d");
      ctx.clearRect(0, 0, this.ruleWidth, 50);
      ctx.font = "10px Arial";
      // 月份
      if (this.type == 1) {
        let unit = this.ruleWidth / 30;
        let startTime = parseInt(currentTime) - 60 * 60 * 24 * 31 * 15;
        for (let i = 0; i <= 30; i++) {
          this.drawLine(i * unit, 0, i * unit, 5, "white", 1);
          let d = this.getTime(parseInt(startTime) + i * 60 * 60 * 24 * 31);
          ctx.fillStyle = "white";
          ctx.fillText(`${d[1]}月`, i * unit - 12, 25);
          if (d[1] == "01") {
            ctx.fillStyle = "white";
            ctx.fillText(`${d[0]}年`, i * unit - 14, 45);
          }
        }
      }
      // 天
      if (this.type == 2) {
        let unit = this.ruleWidth / 30;
        let startTime = parseInt(currentTime) - 60 * 60 * 24 * 15;
        for (let i = 1; i <= 30; i++) {
          this.drawLine(i * unit, 0, i * unit, 5, "white", 1);
          let d = this.getTime(parseInt(startTime) + i * 60 * 60 * 24);
          ctx.fillStyle = "white";
          ctx.fillText(`${d[1]}-${d[2]}`, i * unit - 12, 25);
        }
      }
      //时
      if (this.type == 3) {
        let unit = this.ruleWidth / 30;
        let startTime = parseInt(currentTime) - 60 * 60 * 15;
        for (let i = 1; i <= 30; i++) {
          this.drawLine(i * unit, 0, i * unit, 5, "white", 1);
          let d = this.getTime(parseInt(startTime) + i * 60 * 60);
          ctx.fillStyle = "white";
          ctx.fillText(`${d[3]}H`, i * unit - 12, 25);
          if (d[3] == "00") {
            ctx.fillStyle = "white";
            ctx.fillText(`${d[1]}-${d[2]}`, i * unit - 14, 45);
          }
        }
      }
      // 分
      if (this.type === 4) {
        let unit = this.ruleWidth / 50;
        let startTime = parseInt(currentTime) - 60 * 25;
        for (let i = 1; i <= 50; i++) {
          this.drawLine(i * unit, 0, i * unit, 5, "white", 1);
          if (i % 5 == 0) {
            let d = this.getTime(parseInt(startTime) + i * 60);
            ctx.fillStyle = "white";
            ctx.fillText(`${d[3]}:${d[4]}`, i * unit - 12, 25);
          }
        }
      }
    },
    timeChange(time) {
      this.setRule(time);
      this.setCurrentLine(time);
      // this.carveVideoScope(time);
    },
    addCanvasListener() {
      let canvasId = document.getElementById("time-line-layer");
      // 鼠标按下时的位置；
      let start = 0;
      // 鼠标按下时的currentTime
      let oldTime = 0;
      canvasId.onmousedown = (e) => {
        this.allowDarw = true;
        start = e.layerX;
        oldTime = this.currentTime;
        canvasId.onmousemove = (e1) => {
          let end = e1.layerX;
          let difference = start - end;
          let current = 0;
          if (this.type === 1) {
            let unit = this.ruleWidth / 30;
            current = oldTime + (difference / unit) * 60 * 60 * 24 * 31;
          }
          if (this.type == 2) {
            let unit = this.ruleWidth / 30;
            current = oldTime + (difference / unit) * 60 * 60 * 24;
          }
          if (this.type === 3) {
            let unit = this.ruleWidth / 30;
            current = oldTime + (difference / unit) * 60 * 60;
          }
          if (this.type === 4) {
            let unit = this.ruleWidth / 50;
            current = oldTime + (difference / unit) * 60;
          }
          if (!this.allowDarw) return;
          this.currentTime = current;
        };
      };
      // 鼠标弹起向外传出当前时间
      canvasId.onmouseup = (e) => {
        this.allowDarw = false;
        let end = e.layerX;
        let current = (start - end) / 2 + oldTime;
        let d = this.getTime(current);
        let str = `${d[0]}-${d[1]}-${d[2]} ${d[3]}:${d[4]}:${d[5]}`;
        this.$emit("changCurrentTime", str);
      };
      canvasId.onmouseleave = (e) => {
        this.allowDarw = false;
      };
    },

    // 时间轴中间指示当前时间刻度
    setCurrentLine(time) {
      let canvasId = document.getElementById("is-select-time");
      let width = this.ruleWidth;
      let ctx = canvasId.getContext("2d");
      ctx.clearRect(0, 0, this.ruleWidth, 60);
      ctx.beginPath();
      ctx.moveTo(parseInt(width / 2), 0);
      ctx.lineTo(parseInt(width / 2), 35);
      //   ctx.strokeStyle = "blue";
      ctx.strokeStyle = "#009e4f";
      ctx.lineWidth = 1;
      ctx.stroke();
      ctx.font = "12px Arial";
      ctx.fillStyle = "white";
      let d = this.getTime(time);
      ctx.fillText(
        `${d[0]}-${d[1]}-${d[2]} ${d[3]}:${d[4]}:${d[5]}`,
        parseInt(width / 2) - 50,
        48
      );
    },

    // 有视频的区域渲染颜色
    carveVideoScope(currentTime) {
      let width = this.ruleWidth;
      let canvas = document.getElementById("is-select-time");
      let ctx = canvas.getContext("2d");
      ctx.clearRect(0, 0, width, 50);

      if (this.isEmpty(this.videoRecords)) {
        return;
      }
      let startPoint =
        parseInt(this.videoRecords.startTime) -
        currentTime +
        parseInt(width / 2);
      let endPoint =
        parseInt(this.videoRecords.endTime) - currentTime + parseInt(width / 2);

      // 起点不能为负数，
      startPoint = startPoint > 0 ? startPoint : 0;
      // 终点不能超出时间尺总长度。
      endPoint = endPoint < width ? endPoint : width;
      let w = endPoint - startPoint;
      ctx.fillStyle = "rgba(230,162,60, 0.7)";
      ctx.fillRect(startPoint, 0, w, 50);
    },
    drawLine(beginX, beginY, endX, endY, color, width) {
      let canvasId = document.getElementById("time-line");
      let ctx = canvasId.getContext("2d");
      ctx.beginPath();
      ctx.moveTo(beginX, beginY);
      ctx.lineTo(endX, endY);
      ctx.strokeStyle = color;
      ctx.lineWidth = width;
      ctx.stroke();
    },

    getTime(timeStamp) {
      let date = new Date(timeStamp * 1000);
      let year = date.getFullYear();
      let month =
        date.getMonth() + 1 < 10
          ? "0" + (date.getMonth() + 1)
          : date.getMonth() + 1;
      let currentDate =
        date.getDate() < 10 ? "0" + date.getDate() : date.getDate();
      let hour = date.getHours() < 10 ? "0" + date.getHours() : date.getHours();
      let minute =
        date.getMinutes() < 10 ? "0" + date.getMinutes() : date.getMinutes();
      let second =
        date.getSeconds() < 10 ? "0" + date.getSeconds() : date.getSeconds();
      return [year, month, currentDate, hour, minute, second];
    },

    isEmpty(value) {
      switch (typeof value) {
        case "undefined":
          return true;
        case "string":
          if (value.replace(/(^[ \t\n\r]*)|([ \t\n\r]*$)/g, "").length == 0)
            return true;
          break;
        case "boolean":
          if (!value) return true;
          break;
        case "number":
          if (0 === value || isNaN(value)) return true;
          break;
        case "object":
          if (null === value || value.length === 0) return true;
          for (var i in value) {
            return false;
          }
          return true;
      }
      return false;
    },
  },
};
</script>

<style lang="scss" scoped>
.time-rule {
  width: 100%;
  height: 100%;

  #time-line {
    background-color: rgb(0, 255, 234, 0);
    position: absolute;
  }
  #is-select-time {
    background-color: gray;
    background-color: rgb(143, 63, 63);
    background: #161616;
    position: absolute;
  }
  #time-line-layer {
    background-color: rgb(0, 255, 234, 0);
    position: absolute;
  }
}
</style>