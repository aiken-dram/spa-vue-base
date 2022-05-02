<template>
  <v-menu
    v-model="showMenu"
    :close-on-content-click="false"
    transition="scale-transition"
    offset-y
    offset-x
    max-width="290px"
    min-width="290px"
  >
    <template v-slot:activator="{ on, attrs }">
      <v-text-field
        :value="dateRangeText"
        prepend-icon="fa-calendar-day"
        single-line
        solo
        dense
        hide-details
        class="ml-2 shrink"
        readonly
        v-bind="attrs"
        v-on="on"
      ></v-text-field>
    </template>
    <v-date-picker v-model="dateRange" locale="ru-RU" range no-title>
      <v-spacer></v-spacer>
      <v-btn text color="primary" @click="setDateRange">OK</v-btn>
    </v-date-picker>
  </v-menu>
</template>

<script>
export default {
  name: "DateRangeSearch",

  props: {
    value: String,
  },

  data: () => ({
    showMenu: false,
    dateRange: [],
  }),

  methods: {
    setDateRange() {
      this.showMenu = false;
      this.$emit("input", this.dateRangeText);
    },
  },

  computed: {
    dateRangeText() {
      return this.dateRange
        .map((p) => this.$options.filters.date(p))
        .join(" ~ ");
    },
  },
};
</script>
