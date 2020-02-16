<template>
  <div class="group">
    <div class="label">
      <strong>{{ label }}</strong>
    </div>
    <div class="value">
      {{ value }}
    </div>
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
</style>

<script>
export default {
  props: {
    label: {
      type: String,
      required: true,
    },
    value: {
      type: String,
      required: true,
    },
  },
};
</script>
