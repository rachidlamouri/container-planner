<template>
  <canvas
    ref="canvas"
    width="600"
    height="400"
  />
</template>

<style lang="scss" scoped>
  canvas {
    border: 1px solid #ccc;
    margin-right: 20px;
  }
</style>

<script>
export default {
  props: {
    containers: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      ctx: null,
    };
  },
  watch: {
    containers() {
      this.draw();
    },
  },
  mounted() {
    this.ctx = this.$refs.canvas.getContext('2d');
    this.ctx.lineWidth = 1;
    this.draw();
  },
  methods: {
    draw() {
      const {
        width: canvasWidth,
        height: canvasHeight,
      } = this.$refs.canvas;
      this.ctx.clearRect(0, 0, canvasWidth, canvasHeight);
      this.containers.forEach((container) => {
        const {
          color,
          x,
          y,
          width: containerWidth,
          height: containerHeight,
        } = container;
        this.ctx.beginPath();
        this.ctx.strokeStyle = color;
        this.ctx.strokeRect(x + 0.5, y + 0.5, containerWidth - 1, containerHeight - 1);
      });
    },
  },
};
</script>
