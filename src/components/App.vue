<template>
  <div id="app">
    <h1>Container Planner</h1>
    <div class="editor">
      <planner-canvas
        :canvas-dimensions="canvasDimensions"
        :pixels-per-mm="pixelsPerMm"
        :max-container-dimensions="maxContainerDimensions"
        :containers="containers"
        :selected-index="selectedIndex"
        :bounding-container="boundingContainer"
        :fill-all-containers="fillAllContainers"
        :shim-groups="shimGroups"
      />
      <div class="side">
        <new-container-form
          class="section"
          :is-out-of-colors="isOutOfColors"
          :max-container-dimensions="maxContainerDimensions"
          @addContainer="addContainer"
        />
        <container-list
          class="section"
          :containers="containers"
          @selectContainer="selectContainer"
          @deleteContainer="deleteContainer"
        />
        <computed-info
          :canvas-dimensions="canvasDimensions"
          :pixels-per-mm="pixelsPerMm"
          :max-container-dimensions="maxContainerDimensions"
          :bounding-container="boundingContainer"
          :is-plan-valid="isPlanValid"
          :tolerance="tolerance"
          @toleranceChanged="tolerance = $event"
        />
        <button @click="toggleFillAllContainers">
          Toggle Fill
        </button>
        <button @click="computeShims">
          Compute Shims
        </button>
      </div>
    </div>
  </div>
</template>

<style lang="scss">
  input::-webkit-outer-spin-button,
  input::-webkit-inner-spin-button {
    -webkit-appearance: none;
    margin: 0;
  }

  * {
    box-sizing: border-box;
    font-family: sans-serif;
    font-size: 14px;
    margin: 0;
    padding: 0;
  }

  html {
    color: #333;
  }

  body {
    padding: 8px;
  }

  h1 {
    font-size: 32px;
  }

  .editor {
    display: flex;
  }

  .side {
    display: flex;
    flex-direction: column;
  }

  .section {
    margin-bottom: 16px;
  }

  .section-title {
    font-size: 18px;
    margin-bottom: 4px;
  }

  button {
    background-color: #eee;
    border: 1.5px solid transparent;
    height: 30px;
    margin-bottom: 8px;
    outline: none;
    width: 120px;

    &:hover {
      background-color: #ddd;
      cursor: pointer;
    }

    &:active {
      border-color: #ccc;
    }
  }
</style>

<script>
import _ from 'lodash';
import ContainerList from './ContainerList.vue';
import NewContainerForm from './NewContainerForm.vue';
import PlannerCanvas from './PlannerCanvas.vue';
import ContainerHelpers from './ContainerHelpers.vue';
import ComputedInfo from './ComputedInfo.vue';
import initialPlan from '../initialPlan.json';

export default {
  name: 'App',
  components: {
    ContainerList,
    NewContainerForm,
    PlannerCanvas,
    ComputedInfo,
  },
  mixins: [ContainerHelpers],
  data() {
    return {
      canvasDimensions: {
        width: 1200,
        height: 800,
      },
      pixelsPerMm: 10,
      selectedIndex: 0,
      containers: [],
      fillAllContainers: false,
      shimGroups: {},
      tolerance: 0,
    };
  },
  computed: {
    boundingContainer() {
      const { x: left = 0 } = _.minBy(this.containers, 'x') || {};
      const { y: top = 0 } = _.minBy(this.containers, 'y') || {};
      const right = _(this.containers)
        .map(({ x, width }) => x + width)
        .max() || 0;
      const bottom = _(this.containers)
        .map(({ y, height }) => y + height)
        .max() || 0;

      const width = right - left;
      const height = bottom - top;

      return {
        x: left,
        y: top,
        width,
        height,
        withTolerance: {
          width: width + this.tolerance,
          height: height + this.tolerance,
        },
      };
    },
    containersExist() {
      return this.containers.length > 0;
    },
    isPlanValid() {
      return this.containers.every(
        ({ isOverlapping, isOutOfBounds }) => !isOverlapping && !isOutOfBounds,
      );
    },
    maxContainerDimensions() {
      return {
        width: this.canvasDimensions.width / this.pixelsPerMm,
        height: this.canvasDimensions.height / this.pixelsPerMm,
      };
    },
    plan() {
      return {
        containers: this.containers.map(_.partialRight(_.pick, ['x', 'y', 'width', 'height'])),
        tolerance: this.tolerance,
        boundingContainer: this.boundingContainer,
        shimGroups: _.mapValues(this.shimGroups, ((group) => {
          const leftEdge = _.minBy(group, 'x').x;
          const topEdge = _.minBy(group, 'y').y;

          return group.map(({ x, y }) => ({
            x: x - leftEdge,
            y: y - topEdge,
          }));
        })),
      };
    },
    selectedContainer() {
      return this.containers[this.selectedIndex];
    },
  },
  created() {
    const { containers, tolerance } = initialPlan;

    this.tolerance = tolerance;

    containers.forEach((dimensions) => {
      this.addContainer(dimensions);
    });
  },
  mounted() {
    document.addEventListener('keydown', ({ key }) => {
      switch (key.toLowerCase()) {
        case 'w':
          this.moveSelectedContainer('up');
          break;
        case 'a':
          this.moveSelectedContainer('left');
          break;
        case 's':
          this.moveSelectedContainer('down');
          break;
        case 'd':
          this.moveSelectedContainer('right');
          break;
        case 'q':
        case 'e':
          this.rotateSelectedContainer();
          break;
        default:
      }
    });
  },
  methods: {
    addContainer(dimensions) {
      const newContainer = this.createNewContainer(
        this.containers,
        dimensions,
        this.maxContainerDimensions,
      );
      this.containers.push(newContainer);

      this.selectLastContainer();
    },
    computeShims() {
      const points = [];

      {
        const {
          x,
          y,
          width,
          height,
        } = this.boundingContainer;

        _.range(height).forEach((rowIndex) => {
          points.push([]);
          _.range(width).forEach((colIndex) => {
            points[rowIndex][colIndex] = {
              x: x + colIndex,
              y: y + rowIndex,
              inContainer: false,
              group: null,
            };
          });
        });
      }

      this.containers.forEach((container) => {
        const {
          x,
          y,
          width,
          height,
        } = container;

        _.range(y, y + height).forEach((rowIndex) => {
          _.range(x, x + width).forEach((colIndex) => {
            const adjustedRowIndex = rowIndex - this.boundingContainer.y;
            const adjustedColIndex = colIndex - this.boundingContainer.x;
            points[adjustedRowIndex][adjustedColIndex].inContainer = true;
          });
        });
      });

      const spreadGroup = (rowIndex, colIndex, groupNumber) => {
        const row = points[rowIndex];

        if (!row) {
          return;
        }

        const point = row[colIndex];

        if (!point || point.inContainer || point.group !== null) {
          return;
        }

        point.group = groupNumber;
        spreadGroup(rowIndex - 1, colIndex, groupNumber); // up
        spreadGroup(rowIndex + 1, colIndex, groupNumber); // down
        spreadGroup(rowIndex, colIndex - 1, groupNumber); // left
        spreadGroup(rowIndex, colIndex + 1, groupNumber); // right
      };

      let groupCount = 0;
      _.range(points.length).forEach((rowIndex) => {
        _.range(points[rowIndex].length).forEach((colIndex) => {
          const point = points[rowIndex][colIndex];

          if (point.inContainer || point.group !== null) {
            return;
          }

          groupCount += 1;
          spreadGroup(rowIndex, colIndex, groupCount);
        });
      });

      this.shimGroups = _(points)
        .flatten()
        .filter(({ inContainer }) => !inContainer)
        .groupBy('group')
        .value();

      // eslint-disable-next-line no-console
      console.log(JSON.stringify(this.plan, null, 2));
    },
    deleteContainer({ index, color }) {
      this.replaceColor(color);
      this.containers.splice(index, 1);

      if (!this.containersExist) {
        this.selectedIndex = 0;
      } else if (index === this.selectedIndex || this.selectedIndex >= this.containers.length) {
        this.selectLastContainer();
      }
    },
    moveSelectedContainer(direction) {
      if (!this.containersExist) {
        return;
      }

      const repositionedContainer = {
        ...this.selectedContainer,
        ...this.getContainerPositionAfterMove(
          this.selectedContainer,
          direction,
          this.maxContainerDimensions,
        ),
      };

      const isOverlapping = this.getIsContainerOverlapping(
        this.containers,
        repositionedContainer,
        this.selectedIndex,
      );

      this.updateSelectedContainer({
        ...repositionedContainer,
        isOverlapping,
      });
    },
    rotateSelectedContainer() {
      if (!this.containersExist) {
        return;
      }

      const rotatedContainer = {
        ...this.selectedContainer,
        ...this.getContainerDimensionsAfterRotation(this.selectedContainer),
      };

      const isOverlapping = this.getIsContainerOverlapping(
        this.containers,
        rotatedContainer,
        this.selectedIndex,
      );

      const isOutOfBounds = this.getIsContainerOutOfBounds(
        rotatedContainer,
        this.maxContainerDimensions,
      );

      this.updateSelectedContainer({
        ...rotatedContainer,
        isOverlapping,
        isOutOfBounds,
      });
    },
    selectContainer(index) {
      this.selectedIndex = index;

      const isOverlapping = this.getIsContainerOverlapping(
        this.containers,
        this.selectedContainer,
        this.selectedIndex,
      );

      this.updateSelectedContainer({
        ...this.selectedContainer,
        isOverlapping,
      });
    },
    selectLastContainer() {
      this.selectContainer(this.containers.length - 1);
    },
    toggleFillAllContainers() {
      this.fillAllContainers = !this.fillAllContainers;
    },
    updateSelectedContainer(newContainer) {
      this.containers.splice(this.selectedIndex, 1, newContainer);
      this.shimGroups = {};
    },
  },
};
</script>
