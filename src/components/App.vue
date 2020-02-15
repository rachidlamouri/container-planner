<template>
  <div id="app">
    <h1>Container Planner</h1>
    <div class="editor">
      <planner-canvas
        :containers="containers"
      />
      <div class="side">
        <new-container-form
          :out-of-colors="colors.length === 0"
          @addContainer="addContainer"
        />
        <container-list
          :containers="containers"
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
  methods: {
    addContainer(dimensions) {
      this.containers.push([
        this.colors.shift(),
        ...dimensions,
      ]);
    },
    deleteContainer({ index, color }) {
      this.colors.push(color);
      this.containers.splice(index, 1);
    },
  },
};
</script>
