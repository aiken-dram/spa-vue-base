<template>
  <v-text-field
    v-model.number="val"
    type="number"
    single-line
    solo
    dense
    hide-details
    class="ml-2 shrink"
  ></v-text-field>
</template>

<script>
export default {
  name: "NumberSearch",

  props: {
    value: Number,
  },

  computed: {
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
