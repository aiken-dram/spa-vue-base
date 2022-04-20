<template>
  <v-data-iterator
    :items="items"
    :search="search"
    disable-pagination
    hide-default-footer
  >
    <template v-slot:header>
      <v-toolbar flat dense>
        <v-select
          v-model="search"
          :items="states"
          :label="$t('common.filter')"
          :no-results-text="$t('common.table.noResult')"
          dense
          solo
          class=""
        ></v-select>
        <v-spacer></v-spacer>
      </v-toolbar>
    </template>

    <template v-slot:default="props">
      <v-list flat dense>
        <v-list-item
          v-for="(p, i) in props.items"
          :key="i"
          :color="p.state"
          dense
        >
          <v-list-item-icon v-if="p.state == 'success'">
            <v-icon color="success">fa-check</v-icon>
          </v-list-item-icon>
          <v-list-item-icon v-if="p.state == 'error'">
            <v-icon color="error">fa-exclamation-circle</v-icon>
          </v-list-item-icon>
          <v-list-item-icon v-if="p.state == 'warning'">
            <v-icon color="warning">fa-exclamation-triangle</v-icon>
          </v-list-item-icon>
          <v-list-item-icon v-if="p.state == 'info'">
            <v-icon color="info">fa-info-circle</v-icon>
          </v-list-item-icon>
          <div v-html="p.body"></div>
        </v-list-item>
      </v-list>
    </template>
  </v-data-iterator>
</template>

<script>
export default {
  name: "BaseSignalRMessages",

  props: {
    /** Api function that starts request */
    items: Array,
  },

  data() {
    return {
      search: null,
      states: [
        { value: "success", text: this.$i18n.t("common.state.success") },
        { value: "info", text: this.$i18n.t("common.state.info") },
        { value: "warning", text: this.$i18n.t("common.state.warning") },
        { value: "error", text: this.$i18n.t("common.state.error") },
      ],
    };
  },
};
</script>

<style></style>
