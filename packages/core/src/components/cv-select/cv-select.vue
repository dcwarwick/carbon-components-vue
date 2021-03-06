<template>
  <cv-wrapper :tag-type="formItem ? 'div' : ''" :class="`cv-select ${carbonPrefix}--form-item`">
    <div
      :data-test="value"
      :class="[
        `${carbonPrefix}--select`,
        {
          'cv-select': !formItem,
          [`${carbonPrefix}--select--inline`]: inline,
          [`${carbonPrefix}--select--light`]: theme === 'light',
          [`${carbonPrefix}--select--invalid`]: isInvalid,
          [`${carbonPrefix}--select--disabled`]: $attrs.disabled,
        },
      ]"
    >
      <label
        :for="uid"
        :class="[
          `${carbonPrefix}--label`,
          {
            [`${carbonPrefix}--visually-hidden`]: hideLabel,
            [`${carbonPrefix}--label--disabled`]: $attrs.disabled,
          },
        ]"
        >{{ label }}</label
      >

      <div v-if="!inline && isHelper" :class="`${carbonPrefix}--form__helper-text`">
        <slot name="helper-text">{{ helperText }}</slot>
      </div>

      <cv-wrapper :tag-type="inline ? 'div' : ''" :class="`${carbonPrefix}--select-input--inline__wrapper`">
        <div :class="`${carbonPrefix}--select-input__wrapper`" :data-invalid="isInvalid">
          <select
            v-bind="$attrs"
            :id="uid"
            :class="`${carbonPrefix}--select-input`"
            v-on="inputListeners"
            ref="select"
            :value="internalValue"
          >
            <slot></slot>
          </select>
          <chevron-down-glyph :class="`${carbonPrefix}--select__arrow`" />
          <warning-filled-16 v-if="isInvalid" :class="`${carbonPrefix}--select__invalid-icon`" />
        </div>

        <div v-if="isInvalid" :class="`${carbonPrefix}--form-requirement`">
          <slot name="invalid-message">{{ invalidMessage }}</slot>
        </div>
        <!-- cv-wrapper div ${carbonPrefix}--select-input--inline__wrapper -->
      </cv-wrapper>

      <div v-if="inline && isHelper" :class="`${carbonPrefix}--form__helper-text`">
        <slot name="helper-text">{{ helperText }}</slot>
      </div>
    </div>
    <!-- cv-wrapper div ${carbonPrefix}--form-item -->
  </cv-wrapper>
</template>

<script>
import uidMixin from '../../mixins/uid-mixin';
import themeMixin from '../../mixins/theme-mixin';
import CvWrapper from '../cv-wrapper/_cv-wrapper';
import ChevronDownGlyph from '@carbon/icons-vue/es/chevron--down';
import WarningFilled16 from '@carbon/icons-vue/es/warning--filled/16';
import carbonPrefixMixin from '../../mixins/carbon-prefix-mixin';

export default {
  name: 'CvSelect',
  inheritAttrs: false,
  components: { CvWrapper, ChevronDownGlyph, WarningFilled16 },
  mixins: [uidMixin, themeMixin, carbonPrefixMixin],
  props: {
    inline: Boolean,
    invalidMessage: { type: String, default: undefined },
    helperText: { type: String, default: undefined },
    formItem: { type: Boolean, default: true },
    hideLabel: Boolean,
    label: { type: String, required: true },
    // *********************
    // declare here to prevent the following from being added to the select
    // *********************
    // multiple does not work with styling from carbon-components 9.20
    multiple: {
      type: String,
      validator: () => {
        console.warn('property multiple not supported in CvSelect');
        return false;
      },
    },
    value: { type: String, default: undefined },
  },
  beforeCreate() {
    // *********************
    // delete here to prevent the following from being added to the select
    // *********************
    // multiple does not work with styling from carbon-components 9.20
    delete this.$attrs.multiple;
  },
  data() {
    return {
      dataValue: undefined,
      isHelper: false,
      isInvalid: false,
    };
  },
  mounted() {
    // this is needed to ensure selected for an option when no value is supplied
    if (this.value === undefined) {
      let options = this.$el.querySelectorAll('option');
      for (let i = 0; i < options.length; i++) {
        if (options[i].attributes.selected) {
          this.dataValue = options[i].value;
        }
      }
    }
    this.checkSlots();
  },
  beforeUpdate() {
    this.checkSlots();
  },
  watch: {
    value() {
      // this is needed to ensure selected for an option when no value is supplied
      if (this.value === undefined) {
        let options = this.$el.querySelectorAll('option');
        for (let i = 0; i < options.length; i++) {
          if (options[i].attributes.selected) {
            this.dataValue = options[i].value;
          }
        }
      }
    },
  },
  model: {
    prop: 'value',
    event: 'change',
  },
  computed: {
    // Bind listeners at the component level to the embedded input element and
    // add our own input listener to service the v-model. See:
    // https://vuejs.org/v2/guide/components-custom-events.html#Customizing-Component-v-model
    inputListeners() {
      return {
        ...this.$listeners,
        input: event => this.$emit('input', event.target.value),
        change: event => this.$emit('change', event.target.value), // use change event for ie11 compatibility
      };
    },
    internalValue() {
      return this.dataValue ? this.dataValue : this.value;
    },
  },
  methods: {
    checkSlots() {
      // NOTE: this.$slots is not reactive so needs to be managed on beforeUpdate
      this.isInvalid = !!(this.$slots['invalid-message'] || (this.invalidMessage && this.invalidMessage.length));
      this.isHelper = !!(this.$slots['helper-text'] || (this.helperText && this.helperText.length));
    },
  },
};
</script>
