<template>
  <button :class="`${carbonPrefix}--overflow-menu ${carbonPrefix}--toolbar-action`" v-on="$listeners" type="button">
    <slot></slot>
  </button>
</template>

<script>
import carbonPrefixMixin from '../../mixins/carbon-prefix-mixin';
export default {
  name: 'cv-data-table-action',
  mixins: [carbonPrefixMixin],
  mounted() {
    // ensure SVG has
    if (this.$el.children[0].classList) {
      this.$el.children[0].classList.add(`${this.carbonPrefix}--toolbar-action__icon`);
    } else if (this.$el.children[0].className) {
      // IE 11 and Edge pre-crhomium
      this.$el.children[0].className += ` ${this.carbonPrefix}--toolbar-action__icon`;
    }
  },
};
</script>
