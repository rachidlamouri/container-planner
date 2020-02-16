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
    padding-left: 20px;
  }
</style>

<script>
import _ from 'lodash';
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
      const width = _.get(this.boundingContainer, 'width', '0');
      const height = _.get(this.boundingContainer, 'height', '0');

      return `${width}mm, ${height}mm`;
    },
  },
};
</script>
