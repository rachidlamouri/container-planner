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
    this.ctx.lineWidth = 2;
    this.draw();
  },
  methods: {
    draw() {
      const { width, height } = this.$refs.canvas;
      this.ctx.clearRect(0, 0, width, height);
      this.containers.forEach((container) => {
        const [color, ...dimensions] = container;
        this.ctx.beginPath();
        this.ctx.strokeStyle = color;
        this.ctx.rect(...dimensions);
        this.ctx.stroke();
      });
    },
  },
};
</script>
