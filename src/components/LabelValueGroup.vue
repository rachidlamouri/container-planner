<template>
  <div class="group">
    <div class="label">
      <strong>{{ label }}</strong>
    </div>
    <div
      v-if="!isInput"
      class="value"
    >
      {{ value }}
    </div>
    <input
      v-if="isInput"
      :value="value"
      class="value"
      type="number"
      @input="$emit('input', $event.target.value)"
      @keydown.e="$event.target.blur()"
    >
  </div>
</template>

<style lang="scss" scoped>
  .group {
    display: flex;
    margin-bottom: 4px;

    &.invalid {
      color: #CC0000;
    }
  }

  .label {
    margin-right: 8px;
    width: 230px;
  }

  input {
    padding-left: 4px;
    padding-right: 4px;
    text-align: right;
    width: 40px;
  }
</style>

<script>
export default {
  props: {
    label: {
      type: String,
      required: true,
    },
    value: {
      type: [String, Number],
      required: true,
    },
    isInput: {
      type: Boolean,
      default: false,
    },
  },
};
</script>
