<template>
  <cv-wrapper :tag-type="formItem ? 'div' : ''" class="cv-number-input" :class="`${carbonPrefix}--form-item`">
    <div
      :class="[
        `${carbonPrefix}--number`,
        {
          [`${carbonPrefix}--number--light`]: isLight,
          [`${carbonPrefix}--number--helpertext`]: isHelper,
          [`cv-number-input`]: !formItem,
          [`${carbonPrefix}--number--mobile`]: mobile,
        },
      ]"
      :data-invalid="isInvalid"
    >
      <label :for="uid" :class="`${carbonPrefix}--label`">{{ label }}</label>
      <div
        :class="[
          `${carbonPrefix}--number__input-wrapper`,
          { [`${carbonPrefix}--number__input-wrapper--warning`]: isWarn },
        ]"
      >
        <input
          :data-invalid="isInvalid"
          :id="uid"
          type="number"
          :value="internalValue"
          v-bind="$attrs"
          v-on="inputListeners"
          :disabled="disabled"
          :step="step"
          :min="min"
          :max="max"
          ref="input"
        />
        <WarningFilled16 v-if="isInvalid" :class="iconClasses" />
        <WarningAltFilled16 v-if="isWarn && !isInvalid" :class="iconClasses" />
        <div :class="`${carbonPrefix}--number__controls`">
          <button
            :class="`${carbonPrefix}--number__control-btn down-icon`"
            @click="doDown"
            type="button"
            :aria-label="ariaLabelForDownButton"
            :disabled="disabled"
            aria-live="polite"
            aria-atomic="true"
          >
            <Subtract16 class="down-icon" />
          </button>
          <div :class="`${carbonPrefix}--number__rule-divider`" />
          <button
            :class="`${carbonPrefix}--number__control-btn up-icon`"
            @click="doUp"
            type="button"
            :aria-label="ariaLabelForUpButton"
            :disabled="disabled"
            aria-live="polite"
            aria-atomic="true"
          >
            <Add16 class="up-icon" />
          </button>
          <div :class="`${carbonPrefix}--number__rule-divider`" />
        </div>
      </div>
      <div :class="`${carbonPrefix}--form-requirement`" v-if="isInvalid">
        <slot name="invalid-message">{{ invalidMessage }}</slot>
      </div>
      <div :class="`${carbonPrefix}--form-requirement`" v-if="isWarn && !isInvalid">
        <slot name="warn-text">{{ warnText }}</slot>
      </div>
      <div :class="`${carbonPrefix}--form__helper-text`" v-if="!(isInvalid || isWarn) && isHelper">
        <slot name="helper-text">{{ helperText }}</slot>
      </div>
    </div>
  </cv-wrapper>
</template>

<script>
import { uidMixin, themeMixin, carbonPrefixMixin, methodsMixin } from '../../mixins';
import Add16 from '@carbon/icons-vue/es/add/16';
import Subtract16 from '@carbon/icons-vue/es/subtract/16';
import WarningFilled16 from '@carbon/icons-vue/es/warning--filled/16';
import WarningAltFilled16 from '@carbon/icons-vue/es/warning--alt--filled/16';
import CvWrapper from '../cv-wrapper/_cv-wrapper';

export default {
  name: 'CvNumberInput',
  mixins: [uidMixin, themeMixin, carbonPrefixMixin, methodsMixin({ input: ['blur', 'focus'] })],
  components: { Add16, Subtract16, WarningFilled16, WarningAltFilled16, CvWrapper },
  inheritAttrs: false,
  props: {
    disabled: Boolean,
    formItem: { type: Boolean, default: true },
    helperText: { type: String, default: undefined },
    invalidMessage: { type: String, default: undefined },
    label: { type: String, default: '' },
    value: { type: [String, Number], default: '' },
    invalid: /* deprecate */ {
      type: Boolean,
      default: undefined,
      deprecated: true,
      validator(val) {
        if (val !== undefined && process.env.NODE_ENV === 'development') {
          console.warn('CvNumberInput: invalid prop deprecated in favour of invalidMessage');
        }
        return true;
      },
    },
    ariaLabelForUpButton: { type: String, default: 'Increase number input' },
    ariaLabelForDownButton: { type: String, default: 'Decrease number input' },
    min: { type: [String, Number], default: undefined },
    max: { type: [String, Number], default: undefined },
    step: { type: [String, Number], default: undefined },
    mobile: /* deprecate */ {
      type: Boolean,
      default: undefined,
      deprecated: true,
      validator(val) {
        if (val !== undefined) {
          console.warn('CvNumberInput: mobile prop deprecated as no longer different to standard number input.');
        }
        return true;
      },
    },
    warnText: { type: String, default: undefined },
  },
  data() {
    return {
      internalValue: 0,
      isHelper: false,
      isInvalid: false,
      isWarn: false,
    };
  },
  mounted() {
    this.internalValue = this.valueAsString(this.value);
    this.checkSlots();
  },
  updated() {
    this.checkSlots();
  },
  watch: {
    value() {
      // NOTE: DELIBERATE USE OF != TO COMPARE this.interanlValue and this.value
      if (typeof this.value !== 'number' || this.internalValue != this.value) {
        // prevents this.value of 1 updating this.internalValue of 1.0
        // which improves the typing experience
        // does not matter if this.value is string or number
        this.internalValue = this.valueAsString(this.value);
      }
    },
  },
  computed: {
    // Bind listeners at the component level to the embedded input element and
    // add our own input listener to service the v-model. See:
    // https://vuejs.org/v2/guide/components-custom-events.html#Customizing-Component-v-model
    inputListeners() {
      return Object.assign({}, this.$listeners, {
        input: ev => this.onInput(ev.target.value),
      });
    },
    iconClasses() {
      const classes = [];
      (this.isInvalid || this.isWarn) && classes.push(`${this.carbonPrefix}--number__invalid`);
      this.isWarn && !this.isInvalid && classes.push(`${this.carbonPrefix}--number__invalid--warning`);
      this.readOnly && classes.push(`${this.carbonPrefix}--number__readonly-icon`);

      return classes.join(' ');
    },
  },
  methods: {
    onInput(val) {
      this.internalValue = val;
      this.emitValue();
    },
    checkSlots() {
      // NOTE: this.$slots is not reactive so needs to be managed on updated
      this.isInvalid = !!(this.$slots['invalid-message'] || (this.invalidMessage && this.invalidMessage.length));
      this.isWarn = !!(this.$slots['warn-text'] || (this.warnText && this.warnText.length));
      this.isHelper = !!(this.$slots['helper-text'] || (this.helperText && this.helperText.length));
    },
    doUp() {
      this.$refs.input.stepUp();
      this.onInput(this.$refs.input.value);
    },
    doDown() {
      this.$refs.input.stepDown();
      this.onInput(this.$refs.input.value);
    },
    emitValue() {
      if (typeof this.value === 'number') {
        if (this.internalValue != this.value) {
          const ePos = this.internalValue.indexOf('e-');
          const dotPos = this.internalValue.indexOf('.');
          if (ePos > -1 || dotPos > -1) {
            this.$emit('input', parseFloat(this.internalValue));
          } else {
            this.$emit('input', parseInt(this.internalValue));
          }
        }
      } else {
        this.$emit('input', this.internalValue);
      }
    },
    valueAsString(val) {
      let strVal;
      if (typeof val === 'number') {
        strVal = Number.isFinite(val) ? val.toString() : '';
      } else {
        strVal = val;
      }
      return strVal;
    },
  },
};
</script>
