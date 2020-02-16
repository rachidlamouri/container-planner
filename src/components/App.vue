<template>
  <div id="app">
    <h1>Container Planner</h1>
    <div class="editor">
      <planner-canvas
        :containers="containers"
        :selected-index="selectedIndex"
        @maxDimensions="setMaxDimensions"
      />
      <div class="side">
        <new-container-form
          :out-of-colors="colors.length === 0"
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

export default {
  name: 'App',
  components: {
    ContainerList,
    NewContainerForm,
    PlannerCanvas,
  },
  data() {
    return {
      selectedIndex: 0,
      containers: [],
      maxDimensions: {},
      colors: Object.values({
        aqua: '#00ffff',
        brown: '#a52a2a',
        darkblue: '#00008b',
        darkcyan: '#008b8b',
        darkgrey: '#a9a9a9',
        darkgreen: '#006400',
        darkkhaki: '#bdb76b',
        darkmagenta: '#8b008b',
        darkolivegreen: '#556b2f',
        darkorange: '#ff8c00',
        darkorchid: '#9932cc',
        darkred: '#8b0000',
        darksalmon: '#e9967a',
        green: '#008000',
        indigo: '#4b0082',
        khaki: '#f0e68c',
        lightblue: '#add8e6',
        lightgreen: '#90ee90',
        lightgrey: '#d3d3d3',
        lightpink: '#ffb6c1',
        lime: '#00ff00',
        maroon: '#800000',
        navy: '#000080',
        olive: '#808000',
        orange: '#ffa500',
        pink: '#ffc0cb',
        purple: '#800080',
        red: '#ff0000',
        silver: '#c0c0c0',
      }),
    };
  },
  computed: {
    containersExist() {
      return this.containers.length > 0;
    },
  },
  mounted() {
    document.addEventListener('keydown', ({ key }) => {
      switch (key) {
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
      let nextX = (
        _(this.containers)
          .map(({ x, width }) => x + width)
          .max()
      );

      if (!nextX || nextX >= this.maxDimensions.width) {
        nextX = 0;
      }

      this.containers.push({
        color: this.colors.shift(),
        x: nextX,
        y: 0,
        ...dimensions,
        isOutOfBounds: nextX + dimensions.width > this.maxDimensions.width,
        isOverlapping: false,
      });

      this.selectLastContainer();
      this.moveSelectedContainer('no-op');
    },
    deleteContainer({ index, color }) {
      this.colors.push(color);
      this.containers.splice(index, 1);

      if (index === this.selectedIndex) {
        this.selectLastContainer();
      }
    },
    moveSelectedContainer(direction) {
      if (!this.containersExist) {
        return;
      }

      const container = this.containers[this.selectedIndex];
      switch (direction) {
        case 'up':
          if (container.y > 0) {
            container.y -= 1;
          }
          break;
        case 'down':
          if (container.y + container.height < this.maxDimensions.height) {
            container.y += 1;
          }
          break;
        case 'left':
          if (container.x > 0) {
            container.x -= 1;
            container.isOutOfBounds = container.x + container.width > this.maxDimensions.width;
          }
          break;
        case 'right':
          if (container.x + container.width < this.maxDimensions.width) {
            container.x += 1;
          }
          break;
        default:
      }

      const {
        x: containerLeft,
        y: containerTop,
        width: containerWidth,
        height: containerHeight,
      } = container;
      const containerRight = containerLeft + containerWidth;
      const containerBottom = containerTop + containerHeight;
      container.isOverlapping = this.containers.reduce(
        (isOverlapping, nextContainer, index) => {
          if (isOverlapping || index === this.selectedIndex) {
            return isOverlapping;
          }

          const {
            x: left,
            y: top,
            width,
            height,
          } = nextContainer;
          const right = left + width;
          const bottom = top + height;

          return (
            containerRight > left
            && containerLeft < right
            && containerBottom > top
            && containerTop < bottom
          );
        },
        false,
      );

      this.containers.splice(this.selectedIndex, 1, container);
    },
    selectContainer(index) {
      this.selectedIndex = index;
    },
    selectLastContainer() {
      this.selectContainer(this.containers.length - 1);
    },
    setMaxDimensions(maxDimensions) {
      this.maxDimensions = maxDimensions;
    },
  },
};
</script>
