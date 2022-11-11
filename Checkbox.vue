<template>
  <v-checkbox
    v-model="val"
    :label="tLabel ? $t(tLabel) : label"
    true-value="T"
    false-value="F"
    v-bind="$attrs"
    v-on="$listeners"
  ></v-checkbox>
</template>

<script>
export default {
  name: "BaseCheckbox",

  /** attrs are passed to child component */
  inheritAttrs: false,

  props: {
    /** checkbox value */
    value: String,

    /** Label */
    label: String,

    /** Translate label */
    tLabel: String,
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
  },
};
</script>

<style></style>
