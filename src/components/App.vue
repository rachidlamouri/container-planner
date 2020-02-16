<template>
  <div id="app">
    <h1>Container Planner</h1>
    <div class="editor">
      <planner-canvas
        :containers="containers"
        :selected-index="selectedIndex"
      />
      <div class="side">
        <new-container-form
          :out-of-colors="colors.length === 0"
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
      const nextX = (
        _(this.containers)
          .map(({ x, width }) => x + width)
          .max()
      ) || 0;

      this.containers.push({
        color: this.colors.shift(),
        x: nextX,
        y: 0,
        ...dimensions,
      });

      this.selectLastContainer();
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
          container.y -= 1;
          break;
        case 'down':
          container.y += 1;
          break;
        case 'left':
          container.x -= 1;
          break;
        case 'right':
          container.x += 1;
          break;
        default:
      }

      this.containers.splice(this.selectedIndex, 1, container);
    },
    selectContainer(index) {
      this.selectedIndex = index;
    },
    selectLastContainer() {
      this.selectContainer(this.containers.length - 1);
    },
  },
};
</script>
