<template>
  <v-tooltip bottom max-width="500">
    <template v-slot:activator="{ on, attrs }">
      <v-icon
        :small="small"
        :large="large"
        :color="color"
        class="mr-2"
        @click.stop="$emit('click')"
        v-bind="attrs"
        v-on="on"
      >
        {{ icon }}
      </v-icon>
    </template>
    <span v-html="tooltip" />
  </v-tooltip>
</template>

<script>
export default {
  name: "BaseTableAction",

  props: {
    icon: String,
    tooltip: String,
    small: {
      type: Boolean,
      default: true,
    },
    large: {
      type: Boolean,
      default: false,
    },
    color: String,
  },
};
</script>

<style></style>
