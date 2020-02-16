<template>
  <div>
    <p class="section-title">
      Containers:
    </p>
    <div
      v-for="({ color, x, y, width, height, isOutOfBounds, isOverlapping }, index) in containers"
      :key="index"
      class="dimension-tuple"
    >
      <div
        class="color-swatch"
        :style="{ 'background-color': color }"
      />
      <span
        class="info"
        @click="$emit('selectContainer', index)"
      >
        <span :class="{ invalid: isOutOfBounds || isOverlapping }">{{ padDimension(x) }}</span>,
        <span :class="{ invalid: isOutOfBounds || isOverlapping }">{{ padDimension(y) }}</span>,
        <span :class="{ invalid: isOutOfBounds || isOverlapping }">{{ padDimension(width) }}</span>,
        <span :class="{ invalid: isOutOfBounds || isOverlapping }">{{ padDimension(height) }}</span>
      </span>
      <div
        class="delete-container"
        @click="$emit('deleteContainer', {index, color})"
      >
        X
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
  .dimension-tuple {
    align-items: center;
    display: flex;
    margin-left: 20px;

    .color-swatch {
      height: 10px;
      margin-right: 4px;
      width: 10px;
    }

    .info {
      cursor: pointer;

      .invalid {
        color: #CC0000;
      }

      &:hover {
        background-color: #ccc;
      }
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
export default {
  props: {
    containers: {
      type: Array,
      required: true,
    },
  },
  methods: {
    padDimension(dimension) {
      return dimension.toString().padStart(3, '_');
    },
  },
};
</script>
