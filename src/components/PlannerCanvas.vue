<template>
  <!-- wrapping canvas in a div to prevent stretching -->
  <div class="canvas-wrapper">
    <canvas
      ref="canvas"
      :width="canvasDimensions.width"
      :height="canvasDimensions.height"
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
    canvasDimensions: {
      type: Object,
      required: true,
    },
    pixelsPerMm: {
      type: Number,
      required: true,
    },
    maxContainerDimensions: {
      type: Object,
      required: true,
    },
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
      backgroundCanvas: document.createElement('canvas'),
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

    this.backgroundCanvas.width = this.canvasDimensions.width;
    this.backgroundCanvas.height = this.canvasDimensions.height;
    this.saveGrid();

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
      this.ctx.clearRect(0, 0, this.canvasDimensions.width, this.canvasDimensions.height);
    },
    drawGrid() {
      this.ctx.drawImage(this.backgroundCanvas, 0, 0);
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
    saveGrid() {
      const shadedGridCellCount = this.maxContainerDimensions.width / 2;
      const rowCount = this.maxContainerDimensions.height;

      const backgroundCtx = this.backgroundCanvas.getContext('2d');

      const gridColor = '#efefef';
      backgroundCtx.strokeStyle = gridColor;
      backgroundCtx.fillStyle = gridColor;
      _.range(shadedGridCellCount).forEach((xIndex) => {
        _.range(rowCount).forEach((yIndex) => {
          backgroundCtx.beginPath();

          const isEvenRow = yIndex % 2 === 0;
          const xStagger = isEvenRow ? this.pixelsPerMm : 0;

          const dimensions = [
            xStagger + xIndex * 2 * this.pixelsPerMm + 0.5,
            yIndex * this.pixelsPerMm + 0.5,
            this.pixelsPerMm - 1,
            this.pixelsPerMm - 1,
          ];

          backgroundCtx.strokeRect(...dimensions);
          backgroundCtx.fillRect(...dimensions);
        });
      });
    },
  },
};
</script>
