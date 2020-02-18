<template>
  <div>
    <p class="section-title">
      Computed
    </p>
    <div class="info">
      <label-value-group
        label="Canvas Dimensions:"
        :value="`${canvasDimensions.width} px, ${canvasDimensions.height} px`"
      />
      <label-value-group
        label="Conversion:"
        :value="`${pixelsPerMm} px/mm`"
      />
      <label-value-group
        label="Max Container Inner Dimensions:"
        :value="`${maxContainerDimensions.width} mm, ${maxContainerDimensions.height} mm`"
      />
      <label-value-group
        :is-input="true"
        label="Tolerance:"
        :value="tolerance"
        @input="$emit('toleranceChanged', parseFloat($event))"
      />
      <label-value-group
        :class="{ invalid: !isPlanValid }"
        label="Container Inner Dimensions:"
        :value="boundingContainerDisplayDimensions"
      />
    </div>
  </div>
</template>

<style lang="scss" scoped>
  .info {
    display: flex;
    flex-direction: column;
    margin-bottom: 8px;
    padding-left: 20px;
  }
</style>

<script>
import LabelValueGroup from './LabelValueGroup.vue';

export default {
  components: {
    LabelValueGroup,
  },
  props: {
    canvasDimensions: {
      type: Object,
      required: true,
    },
    pixelsPerMm: {
      type: Number,
      required: true,
    },
    tolerance: {
      type: Number,
      required: true,
    },
    maxContainerDimensions: {
      type: Object,
      required: true,
    },
    boundingContainer: {
      type: Object,
      required: true,
    },
    isPlanValid: {
      type: Boolean,
      required: true,
    },
  },
  computed: {
    boundingContainerDisplayDimensions() {
      const { width, height } = this.boundingContainer.withTolerance;

      return `${width} mm, ${height} mm`;
    },
  },
};
</script>
