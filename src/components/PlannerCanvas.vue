<template>
  <!-- wrapping canvas in a div to prevent stretching -->
  <div class="canvas-wrapper">
    <canvas
      ref="canvas"
      width="600"
      height="400"
    />
  </div>
</template>

<style lang="scss" scoped>
  .canvas-wrapper {
    margin-right: 20px;
  }

  canvas {
    border: 1px solid #ccc;
  }
</style>

<script>
export default {
  props: {
    containers: {
      type: Array,
      required: true,
    },
    selectedIndex: {
      type: Number,
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
    selectedIndex() {
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
      this.containers.forEach((container, index) => {
        const {
          color,
          x,
          y,
          width: containerWidth,
          height: containerHeight,
        } = container;
        this.ctx.beginPath();
        this.ctx.strokeStyle = color;
        this.ctx.fillStyle = color;

        const dimensions = [x + 0.5, y + 0.5, containerWidth - 1, containerHeight - 1];
        this.ctx.strokeRect(...dimensions);
        if (index === this.selectedIndex) {
          this.ctx.fillRect(...dimensions);
        }
      });
    },
  },
};
</script>
