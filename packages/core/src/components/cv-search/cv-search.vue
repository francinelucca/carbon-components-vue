<template>
  <cv-wrapper :tag-type="formItem ? 'div' : ''" :class="`cv-search ${carbonPrefix}--form-item`">
    <div
      :aria-labelledby="uid"
      :class="[
        `${carbonPrefix}--search`,
        {
          [`${carbonPrefix}--search--light`]: isLight,
          [`${carbonPrefix}--search--${internalSize}`]: internalSize,
          [`${carbonPrefix}--toolbar-search`]: isToolbarKind,
          [`${carbonPrefix}--toolbar-search--active`]: toolbarActive,
          'cv-search': !formItem,
        },
      ]"
      role="search"
      ref="search"
    >
      <button
        type="button"
        v-if="isToolbarKind"
        :class="`${carbonPrefix}--toolbar-search__btn`"
        :aria-label="toolbarAriaLabel"
        @click="toggleActive(true)"
        @blur="checkFocus"
      >
        <component :is="icon" :class="`${carbonPrefix}--search-magnifier`" />
      </button>

      <div v-if="!isToolbarKind" :class="`${carbonPrefix}--search-magnifier`">
        <Search20 :class="`${carbonPrefix}--search-magnifier-icon`" />
      </div>

      <label :for="uid" :class="`${carbonPrefix}--label`">{{ label }}</label>

      <input
        :id="uid"
        :class="`${carbonPrefix}--search-input`"
        v-bind="$attrs"
        v-model="internalValue"
        v-on="inputListeners"
        type="text"
        ref="input"
        :placeholder="placeholder"
        :aria-labelledby="uid"
        @blur="checkFocus"
      />

      <button
        type="button"
        :class="[`${carbonPrefix}--search-close`, { [`${carbonPrefix}--search-close--hidden`]: !clearVisible }]"
        :title="clearAriaLabel"
        :aria-label="clearAriaLabel"
        @click="onClearClick"
      >
        <Close16 />
      </button>
    </div>
  </cv-wrapper>
</template>

<script>
import { uidMixin, themeMixin, carbonPrefixMixin, methodsMixin } from '../../mixins';
import Search16 from '@carbon/icons-vue/es/search/16';
import Search20 from '@carbon/icons-vue/es/search/20';
import Close16 from '@carbon/icons-vue/es/close/16';
import CvWrapper from '../cv-wrapper/_cv-wrapper';

export default {
  name: 'CvSearch',
  mixins: [uidMixin, themeMixin, carbonPrefixMixin, methodsMixin({ input: ['blur', 'focus'] })],
  components: { Close16, CvWrapper, Search20 },
  inheritAttrs: false,
  props: {
    clearAriaLabel: { type: String, default: 'Clear search input' },
    formItem: { type: Boolean, default: true },
    kind: { type: String, default: undefined },
    label: String,
    size: { type: String, default: undefined },
    small: {
      type: Boolean,
      default: undefined,
      validator(val) {
        if (val !== undefined && process.env.NODE_ENV === 'development') {
          console.warn('DEPRECARTED: Prefer size property: small, large or xl (default)');
        }
        return true;
      },
    },
    large: {
      type: Boolean,
      default: undefined,
      validator(val) {
        if (val !== undefined && process.env.NODE_ENV === 'development') {
          console.warn('DEPRECARTED: Prefer size property: small, large or xl (default)');
        }
        return true;
      },
    },
    value: String,
    placeholder: { type: String, default: 'Search' },
    toolbarAriaLabel: { type: String, default: 'Toolbar search' },
  },
  data() {
    return {
      clearVisible: this.value ? this.value.length : false,
      internalValue: this.value,
      toolbarActive: false,
    };
  },
  watch: {
    value() {
      this.clearVisible = this.value ? this.value.length : false;
      this.internalValue = this.value;
    },
  },
  computed: {
    // Bind listeners at the component level to the embedded input element and
    // add our own input listener to service the v-model. See:
    // https://vuejs.org/v2/guide/components-custom-events.html#Customizing-Component-v-model
    inputListeners() {
      return {
        ...this.$listeners,
        input: this.onInput,
      };
    },
    internalSize() {
      let size;

      if (this.size !== undefined && (this.size || (this.small === undefined && this.large === undefined))) {
        switch (this.size) {
          case 'small':
            size = 'sm';
            break;
          case 'large':
            size = 'lg';
            break;
          default:
            size = 'xl';
            break;
        }
      } else {
        size = this.small ? 'sm' : 'xl';
      }
      return size;
    },
    isToolbarKind() {
      return this.kind === 'toolbar';
    },
    icon() {
      return this.size === 'xl' ? Search20 : Search16;
    },
  },
  methods: {
    onClearClick() {
      this.internalValue = '';
      this.clearVisible = false;
      this.$emit('input', this.internalValue);
    },
    onInput() {
      this.clearVisible = this.internalValue.length > 0;
      return this.$emit('input', this.internalValue);
    },
    toggleActive(force) {
      if (typeof force === 'boolean') {
        this.toolbarActive = force;
      } else {
        this.toolbarActive = !this.toolbarActive;
      }
      if (this.toolbarActive) {
        this.$refs.input.focus();
      }
    },
    checkFocus(ev) {
      if (this.isToolbarKind) {
        if (!this.$el.contains(ev.relatedTarget)) {
          this.toolbarActive = false;
        }
      }
    },
  },
};
</script>
