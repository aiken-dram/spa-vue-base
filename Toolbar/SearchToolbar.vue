<template>
  <v-card flat>
    <v-menu
      v-model="menu"
      :close-on-content-click="false"
      offset-y
      max-height="500"
    >
      <template v-slot:activator="{ on, attrs }">
        <v-btn
          color="primary"
          dark
          v-bind="attrs"
          v-on="on"
          small
          absolute
          right
          fab
          class="mt-1"
        >
          <v-icon>fa-plus</v-icon>
        </v-btn>
      </template>
      <v-list>
        <v-subheader>Выберите поле из списка:</v-subheader>
        <template v-for="item in search">
          <v-list-group v-if="item.children" :key="item.title">
            <template v-slot:activator>
              <v-list-item-content>
                <v-list-item-title>
                  {{ item.title }}
                </v-list-item-title>
              </v-list-item-content>
            </template>
            <v-list-item
              v-for="child in item.children"
              :key="child.name"
              @click="addExtendedSearch(child)"
            >
              <v-list-item-icon>
                <v-icon small> {{ iconType(child.type) }}</v-icon>
              </v-list-item-icon>
              <v-list-item-title>{{ child.title }}</v-list-item-title>
            </v-list-item>
          </v-list-group>
          <v-list-item
            v-else
            :key="item.title"
            @click="addExtendedSearch(item)"
          >
            <v-list-item-title>{{ item.title }}</v-list-item-title>
          </v-list-item>
        </template>
      </v-list>
    </v-menu>

    <slot> </slot>

    <div v-if="extended || extended.length > 0">
      <v-toolbar
        v-for="ext in extended"
        :key="ext.id"
        dense
        flat
        max-width="800"
      >
        <v-btn icon @click="removeExtendedSearch(ext)">
          <v-icon color="error">fa-minus-square</v-icon>
        </v-btn>

        <span class="mr-2"> {{ ext.title }} </span>

        <text-search v-if="ext.type == 'text'" v-model="ext.filter">
        </text-search>

        <date-range-search
          v-else-if="ext.type == 'date'"
          v-model="ext.filter"
        ></date-range-search>

        <number-search v-else-if="ext.type == 'number'" v-model="ext.filter">
        </number-search>

        <dictionary-search
          v-else-if="ext.type == 'dictionary'"
          v-model="ext.filter"
          :dictionary="ext.dict"
        ></dictionary-search>

        <dictionary-string-search
          v-else-if="ext.type == 'dictionaryString'"
          v-model="ext.filter"
          :dictionary="ext.dict"
        ></dictionary-string-search>
      </v-toolbar>
    </div>

    <v-card-actions>
      <slot name="actions"></slot>
    </v-card-actions>
  </v-card>
</template>

<script>
import TextSearch from "./TextSearch";
import NumberSearch from "./NumberSearch";
import DateRangeSearch from ".//DateRangeSearch";
import DictionarySearch from "./DictionarySearch";
import DictionaryStringSearch from "./DictionaryStringSearch"; //KOSTYL

/**
 * Extended search toolbar component
 *
 * Insert component where you want to use it:
 * <search-toolbar :search="extended" ref="extended">
 * ...
 *   <template v-slot:actions>
 *   ...
 *   </template>
 * </search-toolbar>
 *
 * Get extended filter value:
 * this.$refs.extended.get();
 */
export default {
  name: "ExtendedSearch",

  props: {
    search: Array,
  },

  data: () => ({
    menu: false,
    id: 1,
    extended: [],
  }),

  methods: {
    addExtendedSearch(item) {
      var ext = {
        id: this.id++,
        title: item.title,
        name: item.name,
        type: item.type,
        dict: item.dict,
      };
      this.extended.push(ext);
      this.menu = false;
    },
    removeExtendedSearch(item) {
      for (var i = 0; i < this.extended.length; i++) {
        if (this.extended[i].id === item.id) {
          this.extended.splice(i, 1);
        }
      }
    },
    getFilter() {
      var ext = [];
      if (this.extended)
        this.extended.forEach((e) => {
          if (e.filter || e.filter === 0) {
            switch (e.type) {
              case "text":
                ext.push(`${e.name}|like|${e.filter}`);
                return;
              case "number":
                ext.push(`${e.name}|==|${e.filter}`);
                return;
              case "date":
                ext.push(`${e.name}|date|${e.filter}`);
                return;
              case "dictionary":
              case "dictionaryString":
                ext.push(`${e.name}|==|${e.filter}`);
                return;
            }
          }
        });
      return ext;
    },

    clear() {
      this.extended = [];
    },

    iconType(t) {
      switch (t) {
        case "text":
          return "fa-terminal";
        case "date":
          return "fa-calendar-alt";
        case "number":
          return "fa-sort";
        case "dictionary":
        case "dictionaryString":
          return "fa-list";
      }
      return "";
    },
  },

  components: {
    TextSearch,
    NumberSearch,
    DateRangeSearch,
    DictionarySearch,
    DictionaryStringSearch,
  },
};
</script>
