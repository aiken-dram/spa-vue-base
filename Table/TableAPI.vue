<template>
  <v-data-table
    v-model="selected"
    :headers="headers"
    :loading="loading"
    :items="items"
    :options.sync="options"
    :server-items-length="totalLength"
    :item-class="itemClass"
    :loading-text="$t('common.table.loading')"
    :no-data-text="$t('common.table.noData')"
    :no-results-text="$t('common.table.noResult')"
    :footer-props="{
      //showFirstLastPage: true,
      itemsPerPageOptions: [10, 50, 100],
      itemsPerPageText: $t('common.table.page'),
    }"
    :class="hidePointer ? null : 'table-cursor'"
    v-bind="$attrs"
    v-on="$listeners"
  >
    <!-- Displaying error from API -->
    <template v-if="error" v-slot:no-data>
      <v-alert
        border="top"
        colored-border
        type="error"
        class="mt-2"
        elevation="2"
      >
        {{ error }}
      </v-alert>
    </template>

    <!-- Displaying table filter in headers that have filter enabled -->
    <template v-for="h in headers" v-slot:[`header.${h.value}`]="{ header }">
      <base-table-filter
        v-if="h.filter"
        :key="h.name"
        :filter="h.filter"
        @set-filter="setFilter"
        @remove-filter="removeFilter"
      ></base-table-filter>
      {{ $t(header.text) }}
    </template>

    <!-- Displaying footer with page information -->
    <template v-slot:footer.page-text="props">
      {{
        $t("common.table.records", {
          start: props.pageStart,
          stop: props.pageStop,
          length: props.itemsLength,
        })
      }}
    </template>

    <!-- Displaying button for refreshing data from API -->
    <template slot="footer">
      <div style="position: absolute; margin-left: 10px; margin-top: 10px">
        <v-btn icon @click="load()"><v-icon>fa-spinner</v-icon></v-btn>
      </div>
    </template>

    <template v-slot:top>
      <!-- Default slot for displaying content above table -->
      <slot></slot>

      <!-- Optional toolbar that appears above table at right side
           and contains actions for selected rows, along with toolbar and download buttons -->
      <v-toolbar
        v-if="!hideToolbar"
        flat
        :floating="!fixedToolbar"
        dense
        :class="fixedToolbar ? 'fixed-toolbar' : 'mt-n10 float-toolbar'"
      >
        <v-spacer></v-spacer>

        <!-- Toolbar table buttons -->
        <slot name="toolbar-buttons"></slot>

        <!-- selected menu to display in toolbar if any rows have been selected -->
        <v-toolbar-items v-show="selected.length > 0" class="mr-2">
          <base-select-menu :items="actions" @selected="onSelected" />
        </v-toolbar-items>

        <v-tooltip bottom v-if="exportPageIcon">
          <template v-slot:activator="{ on, attrs }">
            <v-btn icon @click="exportPage()" v-bind="attrs" v-on="on">
              <v-icon v-text="exportPageIcon" />
            </v-btn>
          </template>
          <span>{{ $t("common.exportPage") }}</span>
        </v-tooltip>

        <v-tooltip bottom v-if="exportAllIcon">
          <template v-slot:activator="{ on, attrs }">
            <v-btn icon @click="exportAll()" v-bind="attrs" v-on="on">
              <v-icon v-text="exportAllIcon" />
            </v-btn>
          </template>
          <span>{{ $t("common.exportAll") }}</span>
        </v-tooltip>
      </v-toolbar>
    </template>

    <!-- Supported formats for columns -->
    <!-- date filter -->
    <template v-for="d in formatDate" v-slot:[`item.${d.value}`]="{ item }">
      {{ item[d.value] | date }}
    </template>

    <!-- datetime filter -->
    <template
      v-for="dt in formatDatetime"
      v-slot:[`item.${dt.value}`]="{ item }"
    >
      {{ item[dt.value] | datetime }}
    </template>

    <!-- Pass on all named slots -->
    <slot v-for="slot in Object.keys($slots)" :name="slot" :slot="slot" />

    <!-- Pass on all scoped slots -->
    <template
      v-for="slot in Object.keys($scopedSlots)"
      :slot="slot"
      slot-scope="scope"
    >
      <slot :name="slot" v-bind="scope" />
    </template>
  </v-data-table>
</template>

<script>
import BaseTableFilter from "./Filter";
import BaseSelectMenu from "./SelectMenu";
import { mapMutations } from "vuex";

/**
 * Base api table component
 *
 * Supports column header filters:
 *
 * to use, add filter object to headers array element:
 * filter: {
 *  name: "FieldName"
 *  label: "Placeholder text in search field"
 *  type: "filterType"
 * },
 * where filterType can be:
 * - 'text':
 *  text field input that adds 'FieldName|like|value' to Filters array in API query
 * - 'mask'
 *  text field with mask input that adds 'FieldName|==|value' to Filters array in API query
 * - 'date'
 *  BaseDateRagePicker that adds 'FieldName|date|value' to Filters array in API query
 * - 'dict'
 *  BaseSelect that adds 'FieldName|==|value' to Filters array in API query
 * - 'flags'
 *  TableFilterFlagGroup that adds 'FieldName|in|value' to Filters array in API query
 *
 *  2D: write about remaining fuctionality
 *  also add example
 */
export default {
  name: "BaseTableAPI",

  /** attrs are passed to child component */
  inheritAttrs: false,

  props: {
    /** table headers array */
    headers: Array,

    /** actions for selected items */
    actions: {
      type: Array,
      default: () => [],
    },

    /** function to get data from api */
    dataTable: Function,

    /** function to set table row class */
    itemClass: {
      type: Function,
      default: () => {},
    },

    /** Show toolbar */
    hideToolbar: {
      type: Boolean,
      default: false,
    },

    /** Fixed toolbar */
    fixedToolbar: {
      type: Boolean,
      default: false,
    },

    /** icon for exporting current page records */
    exportPageIcon: {
      type: String,
      default: null,
    },

    /** icon for exporting all records through message query system */
    exportAllIcon: {
      type: String,
      default: null,
    },

    /** hmm */
    hidePointer: {
      type: Boolean,
      default: false,
    },
  },

  data: () => ({
    /** array of table items */
    items: [],

    /** array of selected table items */
    selected: [],

    /** total amount of unpaginated records from api */
    totalLength: null,

    /** error while loading items from api */
    error: null,

    /** table data is loading from api */
    loading: true,

    /** sync for table options */
    options: {},

    /** table filters */
    filters: {},
  }),

  methods: {
    ...mapMutations({
      setOverlay: "SET_OVERLAY",
    }),

    /** load table items from api */
    load(resetPage = false) {
      this.loading = true;
      this.selected = [];
      if (resetPage) {
        this.options.page = 1;
      }
      var params = {
        Page: this.options.page,
        ItemsPerPage: this.options.itemsPerPage,
        SortBy: this.options.sortBy.find(Boolean),
        SortDesc: this.options.sortDesc.find(Boolean),
        Filters: this.getFilters(),
      };
      this.dataTable(params)
        .then(({ data }) => {
          //console.log(data.items);
          this.error = null;
          this.items = data.items;
          this.totalLength = data.total;
          this.loading = false;
        })
        .catch((err) => {
          this.items = [];
          this.error = err.Error;
          this.loading = false;
        });
    },

    /**
     * Set filter
     * @param {string} filterName - name of filter
     * @param {Object} filterBody - filter object
     */
    setFilter(filterName, filterBody) {
      this.filters[filterName] = filterBody;
      this.load();
    },

    /**
     * Remove filter
     * @param {string} filterName - name of filter
     */
    removeFilter(filterName) {
      this.filters[filterName] = null;
      this.load();
    },

    /** get filters */
    getFilters() {
      //need to transform values of properties of filters object into array
      var res = [];
      this.headers
        .filter((h) => h.filter)
        .forEach((h) => {
          if (this.filters[h.filter.name])
            res.push(this.filters[h.filter.name]);
        });
      return res;
    },

    /** export current table page */
    exportPage() {
      var params = {
        Page: this.options.page,
        ItemsPerPage: this.options.itemsPerPage,
        SortBy: this.options.sortBy.find(Boolean),
        SortDesc: this.options.sortDesc.find(Boolean),
        Filters: this.getFilters(),
      };
      //console.log(params);
      this.$emit("export-page", params);
    },

    /** export all table */
    exportAll() {
      var params = {
        SortBy: this.options.sortBy.find(Boolean),
        SortDesc: this.options.sortDesc.find(Boolean),
        Filters: this.getFilters(),
      };
      //console.log(params);
      this.$emit("export-all", params, this.totalLength);
    },

    /** alter data from signalR message */
    signalr(idField, idVal, item) {
      var i = this.items.findIndex((p) => p[idField] == idVal);
      if (i) {
        var fields = Object.getOwnPropertyNames(item);
        fields.forEach((p) => {
          this.items[i][p] = item[p];
        });
      }
    },

    onSelected(action) {
      this.$emit("selected", action);
    },
  },

  mounted() {},

  watch: {
    /** table options watch */
    options: {
      handler() {
        //reload table from api on option changes
        this.load();
      },
      deep: true,
    },
  },

  computed: {
    /** return headers which have filter property  */
    filterHeaders: function () {
      return this.headers.filter((h) => h.filter);
    },

    /** get row formats from headers */
    formatDate: function () {
      return this.headers.filter((h) => h.format == "date");
    },
    formatDatetime: function () {
      return this.headers.filter((h) => h.format == "datetime");
    },
  },

  components: {
    BaseTableFilter,
    BaseSelectMenu,
  },
};
</script>

<style>
.table-cursor tbody tr:hover {
  cursor: pointer;
}

.float-toolbar {
  position: absolute;
  right: 10px;
  z-index: 3;
}

.fixed-toolbar {
  z-index: 3;
}
</style>
