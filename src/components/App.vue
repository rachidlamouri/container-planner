<template>
  <div id="app">
    <h1>Container Planner</h1>
    <div class="editor">
      <canvas
        ref="canvas"
        width="600"
        height="400"
      />
      <div class="side">
        <form @keydown.enter="onEnterPressed">
          <p class="form-title">
            New Container
          </p>
          <div>
            <label>Width</label>
            <input
              ref="firstInput"
              v-model.number="inputs.width"
              type="number"
            >
          </div>
          <div>
            <label>Height</label>
            <input
              ref="secondInput"
              v-model.number="inputs.height"
              type="number"
            >
          </div>
          <p
            v-if="errorMessage"
            class="error"
          >
            {{ errorMessage }}
          </p>
        </form>
        <div>
          <p class="form-title">
            Containers:
          </p>
          <div
            v-for="([color, x, y, width, height], index) in containers"
            :key="index"
            class="dimension-tuple"
          >
            <div
              class="color-swatch"
              :style="{ 'background-color': color }"
            />
            <span>
              {{ padDimension(x) }},
              {{ padDimension(y) }},
              {{ padDimension(width) }},
              {{ padDimension(height) }}
            </span>
            <div
              class="delete-container"
              @click="deleteContainer(index, color)"
            >
              X
            </div>
          </div>
        </div>
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

  canvas {
    border: 1px solid #ccc;
    margin-right: 20px;
  }

  .side {
    display: flex;
    flex-direction: column;
  }

  form {
    > div {
      align-items: center;
      display: flex;
      margin-bottom: 8px;
    }

    label {
      display: flex;
      margin-right: 4px;
      width: 50px;
    }

    input {
      display: flex;
      padding-left: 2px;
      padding-right: 2px;
      width: 50px;
    }

    .error {
      color: #CC0000;
    }
  }

  .form-title {
    font-size: 18px;
    margin-bottom: 4px;
  }

  .dimension-tuple {
    align-items: center;
    display: flex;
    margin-left: 20px;

    .color-swatch {
      height: 10px;
      margin-right: 4px;
      width: 10px;
    }

    .delete-container {
      color: #CC0000;
      cursor: pointer;
      margin-left: 20px;
      padding-left: 4px;
      padding-right: 4px;

      &:hover {
        background-color: #ccc;
      }
    }
  }
</style>

<script>
import Joi from '@hapi/joi';

const requiredPositiveInteger = () => Joi.number().integer().positive().required();
const formSchema = Joi.object({
  width: requiredPositiveInteger(),
  height: requiredPositiveInteger(),
});

export default {
  name: 'App',
  data() {
    return {
      ctx: null,
      inputs: null,
      errorMessage: '',
      containers: [
      ],
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
    displayContainers() {
      return JSON.stringify(this.containers, null, 2);
    },
  },
  created() {
    this.resetInputs();
  },
  mounted() {
    this.$refs.firstInput.focus();
    this.ctx = this.$refs.canvas.getContext('2d');
    this.ctx.lineWidth = 2;
    this.draw();
  },
  methods: {
    onEnterPressed() {
      if (document.activeElement !== this.$refs.secondInput) {
        this.$refs.secondInput.focus();
        return;
      }

      if (!this.validateForm()) {
        return;
      }

      this.containers.push([
        this.colors.shift(),
        0,
        0,
        this.inputs.width,
        this.inputs.height,
      ]);

      this.draw();

      this.$refs.firstInput.focus();
      this.resetInputs();
    },
    deleteContainer(index, color) {
      this.colors.push(color);
      this.containers.splice(index, 1);
      this.draw();
    },
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
    padDimension(dimension) {
      return dimension.toString().padStart(2, '0');
    },
    resetInputs() {
      this.inputs = {
        width: '',
        height: '',
      };
    },
    validateForm() {
      try {
        if (this.colors.length === 0) {
          throw new Error('out of colors');
        }

        Joi.assert(
          this.inputs,
          formSchema,
          {
            abortEarly: false,
            convert: false,
          },
        );
        this.errorMessage = '';
        return true;
      } catch (error) {
        this.errorMessage = `Error: ${error.message}`;
        return false;
      }
    },
  },
};
</script>
