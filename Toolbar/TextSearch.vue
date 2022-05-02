<template>
  <v-text-field
    v-model="val"
    single-line
    solo
    dense
    hide-details
    class="ml-2 shrink"
  ></v-text-field>
</template>

<script>
export default {
  name: "TextSearch",

  props: {
    value: String,
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
