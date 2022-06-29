<template>
  <div class="canvas-component">
    <canvas class="waves" ref="curve">
      Your browser does not support canvas.
    </canvas>
  </div>
</template>

<script>
export default {
  name: "canvas-wave",
  props: {
    scatterStart: {
      type: Number,
      default: 0,
    },
    scatterEnd: {
      type: Number,
      default: 10,
    },
    lines: {
      type: Number,
      default: 2,
    },
    colors: {
      type: Array,
      default: () => {
        return [
          { color: "#0ff", colorStop: 0.1 },
          { color: "#00f", colorStop: 1 },
        ];
      },
    },
    gradientMode: {
      type: Object,
      default: () => {
        return {
          enabled: true,
          vertical: false,
        };
      },
    },
    yFrom: {
      type: Number,
      default: 0.5,
    },
    yTo: {
      type: Number,
      default: 0,
    },
    speed: {
      type: Number,
      default: 90,
    },
    reversed: {
      type: Boolean,
      default: false,
    },
    opacity: {
      type: Number,
      default: 0.8,
    },
    amplitude: {
      type: Object,
      default: () => {
        return {
          type: 0,
          value: 100,
          x: 0,
          y: 0,
        };
      },
    },
    mouseMode: {
      type: Boolean,
      default: false,
    },
    mouseSensivity: {
      type: Number,
      default: 8,
    },
  },
  data() {
    return {
      context: null,
      width: null,
      height: null,
      t: 0,
      x: null,
      y: null,
      gradient: null,
      dataColors: [],
      mouseCoords: {
        x: 0,
        y: 0,
      },
    };
  },
  mounted() {
    this.context = this.$refs.curve.getContext("2d");

    if (this.mouseMode) {
      window.addEventListener("mousemove", this.setMouseCoords);
    }

    window.addEventListener("resize", this.setCanvasSize);

    this.setCanvasSize();
    this.createColor();

    requestAnimationFrame(this.draw);
  },
  methods: {
    setCanvasSize() {
      if (!this.$refs.curve) {
        return;
      }

      let width = this.$refs.curve.parentElement.clientWidth;
      let height = this.$refs.curve.parentElement.clientHeight;

      this.$refs.curve.width = this.width = width;
      this.$refs.curve.height = this.height = height;

      this.createColor();
    },
    drawCurve(color, index, scatterStart, scatterEnd) {
      let fillHeight;
      let fillScatterStart;
      let fillScatterEnd;

      scatterStart = scatterStart * index;
      scatterEnd = scatterEnd * index;

      this.context.fillStyle = color;
      this.context.globalAlpha = this.opacity;

      if (this.reversed) {
        fillHeight = -this.height;
        fillScatterStart = this.yFrom - scatterStart;
        fillScatterEnd = this.yTo - scatterEnd;
      } else {
        fillHeight = this.height;
        fillScatterStart = this.yFrom + scatterStart;
        fillScatterEnd = this.yTo + scatterEnd;
      }

      this.context.moveTo(0, this.yFrom);
      this.context.beginPath();

      this.context.bezierCurveTo(
        0,
        fillScatterStart,
        this.x,
        this.y,
        this.width,
        fillScatterEnd
      );
      this.context.lineTo(this.width, fillHeight);
      this.context.lineTo(0, fillHeight);
      this.context.fill();

      this.context.closePath();
    },
    draw() {
      this.clear();
      this.returnToePoints();

      if (this.height !== this.$refs.curve.parentElement.clientHeight) {
        this.setCanvasSize();
      }

      for (let i = 0; i < this.lines; i++) {
        let color = this.gradientMode.enabled
          ? this.gradient
          : this.dataColors[i];

        this.drawCurve(
          color,
          i,
          this.scatterStart,
          this.scatterEnd,
          this.yFrom,
          this.yTo
        );
      }
      requestAnimationFrame(this.draw);
    },
    createColor() {
      let colors = this.colors;

      if (this.gradientMode.enabled) {
        let gradient = this.context.createLinearGradient(
          0,
          this.gradientMode.vertical ? this.height : 0,
          this.gradientMode.vertical ? this.width : 0,
          this.height
        );
        for (let i = 0; i < colors.length; i++) {
          gradient.addColorStop(colors[i].colorStop, colors[i].color);
        }
        this.gradient = gradient;
      } else {
        for (let i = 0; i < colors.length; i++) {
          this.dataColors.push(colors[i].color);
        }
      }
    },
    clear() {
      this.context.clearRect(0, 0, this.width, this.height);
    },
    setMouseCoords(e) {
      this.mouseCoords.x = e.clientX;
      this.mouseCoords.y = e.clientY;
    },
    returnToePoints() {
      let relativeX = this.amplitude.x ? this.amplitude.x : this.width / 2;
      let relativeY = this.amplitude.y ? this.amplitude.y : this.height / 2;

      let radius = this.amplitude.value;

      let x = relativeX + radius * Math.cos(this.t);
      let y = this.amplitude.type
        ? relativeY + radius * Math.sin(this.t)
        : relativeY + (radius / 2) * Math.sin(this.t);

      this.context.fillStyle = "black";
      this.context.fillRect(x, y, 10, 10);

      this.t = (this.t - 1 / this.speed).toFixed(3);

      this.x =
        this.mouseCoords && this.mouseCoords.x
          ? x + (this.mouseCoords.x - x) / this.mouseSensivity
          : x;
      this.y =
        this.mouseCoords && this.mouseCoords.y
          ? y + (this.mouseCoords.y - y) / this.mouseSensivity
          : y;
    },
  },
};
</script>

<style lang="scss" scoped>
.canvas-component {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  overflow: hidden;
}
</style>
