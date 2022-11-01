<template>
  <v-text-field
    v-model="val"
    type="number"
    :label="tLabel ? $t(tLabel) : label"
    :hint="tHint ? $t(tHint) : hint"
    :rules="rulePreset ? getRules : rules"
    placeholder=" "
    persistent-placeholder
    v-bind="$attrs"
    v-on="$listeners"
  ></v-text-field>
</template>

<script>
/**
 * Base number field component
 */
export default {
  name: "BaseNumberField",

  /** attrs are passed to child component */
  inheritAttrs: false,

  props: {
    /** Text field value */
    value: Number,

    /** Label */
    label: String,

    /** Translate label */
    tLabel: String,

    /** Hint */
    hint: String,

    /** Translate hint */
    tHint: String,

    /** List of rules */
    rules: Array,

    /** use preset rules */
    rulePreset: {
      type: Boolean,
      default: false,
    },

    /** field is required */
    ruleRequired: {
      type: Boolean,
      default: false,
    },
  },

  computed: {
    /** get and set value */
    val: {
      get() {
        return this.value;
      },
      set(value) {
        this.$emit("input", value);
      },
    },

    /** rules preset builder */
    getRules() {
      var res = [];
      if (this.ruleRequired)
        res.push(
          (v) =>
            !!v ||
            this.$i18n.t("forms.common.mustNotBeEmpty", {
              field: this.$i18n.t(this.tLabel),
            })
        );
      return res;
    },
  },
};
</script>

<style></style>
