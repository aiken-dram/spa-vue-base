<template>
  <v-menu
    v-model="show"
    :close-on-content-click="false"
    :nudge-width="300"
    offset-x
    offset-y
  >
    <template v-slot:activator="{ on, attrs }">
      <v-btn icon x-small @click.stop="" v-bind="attrs" v-on="on">
        <v-icon :color="ButtonColor" x-small>fa-filter</v-icon>
      </v-btn>
    </template>

    <v-list dense>
      <v-list-item v-if="showFilter()">
        <v-checkbox
          v-if="showFilter('sign')"
          v-model="sign"
          true-value="!"
          false-value=""
          :label="$t('common.filters.sign')"
          dense
          class="mr-2 mb-4"
        ></v-checkbox>
        <v-select
          v-if="showFilter('select')"
          v-model="operator"
          :items="getOperators()"
          solo
          dense
        >
          <template slot="selection" slot-scope="data">
            {{ $t("common.filters.operators." + data.item) }}
          </template>
          <template slot="item" slot-scope="data">
            {{ $t("common.filters.operators." + data.item) }}
          </template>
        </v-select>
      </v-list-item>

      <v-list-item v-if="filter.type == 'text'">
        <v-text-field v-model="text" outlined dense></v-text-field>
      </v-list-item>

      <v-list-item v-if="filter.type == 'number'">
        <v-text-field
          v-model.number="number"
          type="number"
          outlined
          dense
        ></v-text-field>
      </v-list-item>

      <v-list-item v-if="filter.type == 'mask'">
        <v-text-field
          v-model="text"
          v-facade="filter.mask"
          t-label="common.filters.mask"
          outlined
          dense
        ></v-text-field>
      </v-list-item>

      <v-list-item v-if="filter.type == 'date'">
        <base-date-range-picker
          v-model="dateRange"
          t-label="common.filters.date"
        ></base-date-range-picker>
      </v-list-item>

      <v-list-item v-if="filter.type == 'dict'">
        <base-select
          v-model="dict"
          t-label="common.filters.dict"
          :dictionary="filter.dictionary"
          dense
        ></base-select>
      </v-list-item>

      <v-list-item v-if="filter.type == 'checkbox'">
        <base-checkbox
          v-model="checkbox"
          :t-label="filter.label"
          dense
        ></base-checkbox>
      </v-list-item>

      <div v-if="filter.type == 'flags'">
        <div v-for="(f, i) in filter.flags" :key="i">
          <base-table-filter-flag-group
            v-if="f.group"
            v-model="flags[f.name]"
            :flag="f"
          ></base-table-filter-flag-group>
          <base-table-filter-flag
            v-else
            v-model="flags[f.name]"
            :flag="f"
          ></base-table-filter-flag>
        </div>
      </div>

      <v-list-item>
        <v-spacer />
        <v-btn small @click="applyFilter">{{ $t("common.search") }}</v-btn>
        <v-btn small text @click="cancelFilter">
          {{ $t("common.cancel") }}
        </v-btn>
      </v-list-item>
    </v-list>
  </v-menu>
</template>

<script>
import FLAGS from "@/common/flags";
import TABLE from "@/common/table";

import BaseDateRangePicker from "./../DateTime/DateRangePicker";
import BaseSelect from "./../Select";
import BaseCheckbox from "./../Checkbox";
import BaseTableFilterFlag from "./FilterFlag";
import BaseTableFilterFlagGroup from "./FilterFlagGroup";

/**
 * Base api table filter component
 * This is used by TableAPI to display and apply filters in column headers
 */
export default {
  name: "TableFilter",

  props: {
    /** filter object */
    filter: Object,
  },

  data: () => ({
    /** show filter menu */
    show: false,

    /** operator for filter */
    operator: null,

    /** boolean sign for filter */
    sign: "",

    /** filter is active */
    active: false,

    /** filter value for text */
    text: null,

    /** filter value for number */
    number: null,

    /** filter value for dictionary */
    dict: null,

    /** filter value for checkbox */
    checkbox: null,

    /** filter value for date range */
    dateRange: [],

    /** filter value for flags */
    flags: {},
  }),

  methods: {
    getOperators() {
      return TABLE.filters.operators(this.filter.type);
    },
    showFilter(element) {
      return TABLE.filters.show(this.filter.type, element);
    },

    /** applies this filter to parent table component */
    applyFilter() {
      this.active = true;
      this.show = false;
      this.$emit(
        "set-filter",
        this.filter.name,
        TABLE.filters.getFilter(
          `${this.sign}${this.operator}`,
          this.filter.type,
          this.filter.name,
          this.text,
          this.number,
          this.dateRange,
          this.dict,
          this.checkbox,
          this.filter.flags,
          this.flags,
          FLAGS
        )
      );
    },

    /** cancel this filter from parent table component */
    cancelFilter() {
      this.active = false;
      this.show = false;
      this.$emit("remove-filter", this.filter.name);
    },
  },

  mounted() {
    /** default operator */
    this.operator = TABLE.filters.initial(this.filter.type);
  },

  computed: {
    /** filter button icon color, based on if filter is active or not */
    ButtonColor() {
      return this.active ? "black" : "grey lighten-1";
    },
  },

  components: {
    BaseDateRangePicker,
    BaseCheckbox,
    BaseSelect,
    BaseTableFilterFlag,
    BaseTableFilterFlagGroup,
  },
};
</script>
