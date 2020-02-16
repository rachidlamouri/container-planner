<template>
  <div id="app">
    <h1>Container Planner</h1>
    <div class="editor">
      <planner-canvas
        :containers="containers"
        :selected-index="selectedIndex"
        :bounding-container="boundingContainer"
        @maxDimensions="setMaxDimensions"
      />
      <div class="side">
        <new-container-form
          :is-out-of-colors="isOutOfColors"
          :max-dimensions="maxDimensions"
          @addContainer="addContainer"
        />
        <container-list
          :containers="containers"
          @selectContainer="selectContainer"
          @deleteContainer="deleteContainer"
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
    color: #333;
    font-family: sans-serif;
    font-size: 14px;
    margin: 0;
    padding: 0;
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

export default {
  name: 'App',
  components: {
    ContainerList,
    NewContainerForm,
    PlannerCanvas,
  },
  mixins: [ContainerHelpers],
  data() {
    return {
      selectedIndex: 0,
      containers: [],
      maxDimensions: {},
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
        default:
      }
    });
  },
  methods: {
    addContainer(dimensions) {
      const newContainer = this.createNewContainer(
        this.containers,
        dimensions,
        this.maxDimensions.width,
      );
      this.containers.push(newContainer);

      this.selectLastContainer();
    },
    deleteContainer({ index, color }) {
      this.replaceColor(color);
      this.containers.splice(index, 1);

      if (index === this.selectedIndex) {
        this.selectLastContainer();
      }
    },
    moveSelectedContainer(direction) {
      if (!this.containersExist) {
        return;
      }

      const newContainer = {
        ...this.selectedContainer,
        ...this.getContainerPositionAfterMove(this.selectedContainer, direction),
      };

      const isOverlapping = this.getIsContainerOverlapping(
        this.containers,
        newContainer,
        this.selectedIndex,
      );

      this.updateSelectedContainer({
        ...newContainer,
        isOverlapping,
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
    setMaxDimensions(maxDimensions) {
      this.maxDimensions = maxDimensions;
    },
    updateSelectedContainer(newContainer) {
      this.containers.splice(this.selectedIndex, 1, newContainer);
    },
  },
};
</script>
