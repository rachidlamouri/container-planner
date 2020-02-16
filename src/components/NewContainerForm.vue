<template>
  <form @keydown.enter="onEnterPressed">
    <p class="section-title">
      New Container
    </p>
    <div>
      <label>Width</label>
      <input
        ref="widthInput"
        v-model.number="inputs.width"
        type="number"
      >
    </div>
    <div>
      <label>Height</label>
      <input
        ref="heightInput"
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
</template>

<style lang="scss" scoped>
  form {
    margin-bottom: 8px;

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
</style>

<script>
import Joi from '@hapi/joi';

const requiredPositiveInteger = () => Joi.number().integer().positive().required();
const formSchema = Joi.object({
  width: requiredPositiveInteger(),
  height: requiredPositiveInteger(),
});

export default {
  props: {
    isOutOfColors: {
      type: Boolean,
      required: true,
    },
    maxDimensions: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      inputs: null,
      errorMessage: '',
    };
  },
  created() {
    this.resetInputs();
  },
  mounted() {
    this.$refs.widthInput.focus();
  },
  methods: {
    onEnterPressed() {
      this.errorMessage = '';
      if (document.activeElement !== this.$refs.heightInput) {
        this.$refs.heightInput.focus();
        return;
      }

      if (!this.validateForm()) {
        return;
      }

      const { width, height } = this.inputs;
      this.$emit('addContainer', { width, height });

      this.$refs.widthInput.focus();
      this.resetInputs();
    },
    resetInputs() {
      this.inputs = {
        width: '',
        height: '',
      };
    },
    validateForm() {
      try {
        if (this.isOutOfColors) {
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

        if (this.inputs.width > this.maxDimensions.width) {
          throw new Error('width is too big');
        }
        if (this.inputs.height > this.maxDimensions.height) {
          throw new Error('height is too big');
        }

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
