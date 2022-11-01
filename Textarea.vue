<template>
  <v-textarea
    v-model="val"
    :label="tLabel ? $t(tLabel) : label"
    :hint="tHint ? $t(tHint) : hint"
    :rules="rulePreset ? getRules : rules"
    placeholder=" "
    persistent-placeholder
    auto-grow
    v-bind="$attrs"
    v-on="$listeners"
  ></v-textarea>
</template>

<script>
/**
 * Base text area component
 */
export default {
  name: "BaseTextarea",

  /** attrs are passed to child component */
  inheritAttrs: false,

  props: {
    /** Text field value */
    value: String,

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

    ruleMax: {
      type: Number,
      default: 0,
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
      if (this.ruleMax > 0)
        res.push(
          (v) =>
            (v || "").length <= this.ruleMax ||
            this.$i18n.t("forms.common.maxLength", { length: this.ruleMax })
        );
      return res;
    },
  },
};
</script>

<style></style>
