<template>
  <!-- wrapping canvas in a div to prevent stretching -->
  <div class="canvas-wrapper">
    <canvas
      ref="canvas"
      width="1200"
      height="800"
    />
  </div>
</template>

<style lang="scss" scoped>
  .canvas-wrapper {
    margin-right: 20px;
  }

  canvas {
    border: 1px solid #aaa;
  }
</style>

<script>
import _ from 'lodash';

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
    boundingContainer: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      ctx: null,
      pixelsPerMm: 10,
    };
  },
  computed: {
    canvasWidth() {
      return this.$refs.canvas.width;
    },
    canvasHeight() {
      return this.$refs.canvas.height;
    },
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
    this.$emit('maxDimensions', {
      width: this.canvasWidth / this.pixelsPerMm,
      height: this.canvasHeight / this.pixelsPerMm,
    });
    this.ctx = this.$refs.canvas.getContext('2d');
    this.ctx.lineWidth = 1;
    this.draw();
  },
  methods: {
    draw() {
      this.clearCanvas();
      this.drawGrid();
      this.drawContainers();
      this.drawBoundingContainer();
    },
    clearCanvas() {
      this.ctx.clearRect(0, 0, this.canvasWidth, this.canvasHeight);
    },
    drawGrid() {
      const shadedGridCellCount = this.canvasWidth / this.pixelsPerMm / 2;
      const rowCount = this.canvasHeight / this.pixelsPerMm;

      const gridColor = '#efefef';
      this.ctx.strokeStyle = gridColor;
      this.ctx.fillStyle = gridColor;
      _.range(shadedGridCellCount).forEach((xIndex) => {
        _.range(rowCount).forEach((yIndex) => {
          this.ctx.beginPath();

          const isEvenRow = yIndex % 2 === 0;
          const xStagger = isEvenRow ? this.pixelsPerMm : 0;

          const dimensions = [
            xStagger + xIndex * 2 * this.pixelsPerMm + 0.5,
            yIndex * this.pixelsPerMm + 0.5,
            this.pixelsPerMm - 1,
            this.pixelsPerMm - 1,
          ];

          this.ctx.strokeRect(...dimensions);
          this.ctx.fillRect(...dimensions);
        });
      });
    },
    drawContainers() {
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

        const dimensions = [
          x * this.pixelsPerMm + 0.5,
          y * this.pixelsPerMm + 0.5,
          containerWidth * this.pixelsPerMm - 1,
          containerHeight * this.pixelsPerMm - 1,
        ];
        this.ctx.strokeRect(...dimensions);
        if (index === this.selectedIndex) {
          this.ctx.fillRect(...dimensions);
        }
      });
    },
    drawBoundingContainer() {
      this.ctx.strokeStyle = '#0f0';

      const {
        x,
        y,
        width,
        height,
      } = this.boundingContainer;
      const dimensions = [
        x * this.pixelsPerMm,
        y * this.pixelsPerMm,
        width * this.pixelsPerMm,
        height * this.pixelsPerMm,
      ];
      this.ctx.strokeRect(...dimensions);
    },
  },
};
</script>
