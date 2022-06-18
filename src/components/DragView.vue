<template>
  <div
    class="heat-layout"
    :style="{
      width: `${1920 * scale}px`,
      height: `${1080 * scale}px`,
    }"
  >
    <div
      id="heatDragArea"
      class="heat-drag-area"
      :style="{
        transform: `translate(${heatDragAreaRects.x}px, ${heatDragAreaRects.y}px)`,
        width: `${heatDragAreaRects.w}px`,
        height: `${heatDragAreaRects.h}px`,
      }"
      @mousedown="handleAreaMousedown"
      @mousemove="handleAreaMousemove"
    >
      <span
        v-for="(item, index) in dragOptions"
        :key="item"
        :class="[item, 'heat-drag-point']"
        @mousedown.stop="handleDragPointMousedown($event, index)"
      ></span>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isDrag: false,
      isScale: false,
      scale: 0.6,
      heatDragAreaRects: {
        w: 200,
        h: 200,
        x: 0,
        y: 0,
      },
      currheatDragAreaRects: {
        w: 200,
        h: 200,
      },
      startPosition: {
        x: 0,
        y: 0,
      },
      startPointPosition: {
        x: 0,
        y: 0,
      },
      activePointIndex: 0,
      dragOptions: ["left-top", "right-top", "right-bottom", "left-bottom"],
    };
  },
  mounted() {
    document.addEventListener("mouseup", this.handleAreaMouseup);
    document.addEventListener("mousemove", this.handleDragPointMousemove);
  },
  methods: {
    limitPosition(val, min, max, size) {
      if (val < min) {
        return min;
      }
      if (val > max - size) {
        return max - size;
      }
      return val;
    },
    handleAreaMousedown(e) {
      this.startPosition = {
        x: e.offsetX,
        y: e.offsetY,
      };
      this.isDrag = true;
    },
    handleAreaMousemove(e) {
      if (this.isDrag) {
        const { clientX, clientY } = e;
        let x = this.limitPosition(
          clientX - this.startPosition.x,
          0,
          1920 * this.scale,
          this.heatDragAreaRects.w
        );
        let y = this.limitPosition(
          clientY - this.startPosition.y,
          0,
          1080 * this.scale,
          this.heatDragAreaRects.h
        );
        this.heatDragAreaRects.x = x;
        this.heatDragAreaRects.y = y;
      }
    },
    handleAreaMouseup() {
      this.isDrag = false;
      this.isScale = false;
    },
    handleDragPointMousedown(e, index) {
      this.isScale = true;
      this.activePointIndex = index;
      this.currheatDragAreaRects = { ...this.heatDragAreaRects };
      this.startPointPosition = {
        x: e.clientX,
        y: e.clientY,
      };
    },
    twoPointDistance(p1, p2) {
      let dep = Math.sqrt(Math.pow(p1.x - p2.x, 2) + Math.pow(p1.y - p2.y, 2));
      return dep;
    },

    handleDragPointMousemove(e) {
      if (this.isScale) {
        let { clientX, clientY } = e;
        clientX = this.limitPosition(clientX, 0, 1920 * this.scale, 0);
        clientY = this.limitPosition(clientY, 0, 1080 * this.scale, 0);

        const diffX = clientX - this.startPointPosition.x;
        const diffY = clientY - this.startPointPosition.y;
        let scaleVal = 0;
        const { x, y, w, h } = this.currheatDragAreaRects;
        const limitX = 1920 * this.scale;
        const limitY = 1080 * this.scale;
        console.log(clientX, limitX);
        if (clientX < limitX && clientY < limitY) return;
        switch (this.activePointIndex) {
          case 0:
            scaleVal = Math.min(diffX, diffY);
            this.heatDragAreaRects.w = -scaleVal + w;
            this.heatDragAreaRects.h = -scaleVal + h;
            this.heatDragAreaRects.x = scaleVal + x;
            this.heatDragAreaRects.y = scaleVal + y;
            break;
          case 1:
            if (
              clientX <=
                this.heatDragAreaRects.x + this.heatDragAreaRects.w + 10 &&
              clientY <= this.heatDragAreaRects.y + 10
            ) {
              this.heatDragAreaRects.w = -diffY + w;
              this.heatDragAreaRects.h = -diffY + h;
              this.heatDragAreaRects.y = diffY + y;
            } else {
              this.heatDragAreaRects.w = diffX + w;
              this.heatDragAreaRects.h = diffX + h;
              this.heatDragAreaRects.y = -diffX + y;
            }

            break;
          case 2:
            scaleVal = Math.max(diffX, diffY);
            this.heatDragAreaRects.w = scaleVal + w;
            this.heatDragAreaRects.h = scaleVal + h;
            break;
          case 3:
            if (
              clientX >= this.heatDragAreaRects.x - 10 &&
              clientY >=
                this.heatDragAreaRects.y + this.heatDragAreaRects.h - 10
            ) {
              this.heatDragAreaRects.w = diffY + w;
              this.heatDragAreaRects.h = diffY + h;
              this.heatDragAreaRects.x = -diffY + x;
            } else {
              this.heatDragAreaRects.w = -diffX + w;
              this.heatDragAreaRects.h = -diffX + h;
              this.heatDragAreaRects.x = diffX + x;
            }
            break;
          default:
            break;
        }
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.heat-layout {
  position: absolute;
  background-color: #383838;
  transform-origin: 0 0;
  .heat-drag-area {
    position: absolute;
    width: 200px;
    height: 200px;
    left: 0;
    top: 0;
    border: 2px solid violet;
    background-color: aquamarine;
    cursor: move;
  }
  .heat-drag-point {
    position: absolute;
    width: 10px;
    height: 10px;
    background-color: red;
  }
  .left-top {
    left: -5px;
    top: -5px;
    cursor: nw-resize;
  }
  .right-top {
    right: -5px;
    top: -5px;
    cursor: ne-resize;
  }
  .left-bottom {
    left: -5px;
    bottom: -5px;
    cursor: sw-resize;
  }
  .right-bottom {
    right: -5px;
    bottom: -5px;
    cursor: se-resize;
  }
}
</style>
