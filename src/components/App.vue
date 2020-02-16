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
        />
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
</style>

<script>
import _ from 'lodash';
import ContainerList from './ContainerList.vue';
import NewContainerForm from './NewContainerForm.vue';
import PlannerCanvas from './PlannerCanvas.vue';
import ContainerHelpers from './ContainerHelpers.vue';
import ComputedInfo from './ComputedInfo.vue';

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
    };
  },
  computed: {
    boundingContainer() {
      if (!this.containersExist) {
        return {};
      }

      const { x: left } = _.minBy(this.containers, 'x');
      const { y: top } = _.minBy(this.containers, 'y');
      const right = _(this.containers)
        .map(({ x, width }) => x + width)
        .max();
      const bottom = _(this.containers)
        .map(({ y, height }) => y + height)
        .max();

      return {
        x: left,
        y: top,
        width: right - left,
        height: bottom - top,
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
    selectedContainer() {
      return this.containers[this.selectedIndex];
    },
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
    updateSelectedContainer(newContainer) {
      this.containers.splice(this.selectedIndex, 1, newContainer);
    },
  },
};
</script>
