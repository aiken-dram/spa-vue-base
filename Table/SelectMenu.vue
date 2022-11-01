<template>
  <v-menu attach bottom left offset-y max-height="500">
    <template v-slot:activator="{ on, attrs }">
      <v-btn text v-bind="attrs" v-on="on">
        <span class="text-capitalize">{{ $t(title) }}</span>
        <v-icon x-small class="ml-1">fa-chevron-down</v-icon>
      </v-btn>
    </template>
    <v-list dense nav>
      <div v-for="(header, i) in items" :key="i">
        <div v-if="header.items">
          <v-subheader>{{ $t(header.title) }}</v-subheader>
          <v-list-item-group>
            <v-list-item
              v-for="(item, j) in header.items"
              :key="j"
              @click="select(item.action)"
            >
              <v-list-item-content>
                <v-list-item-title>{{ $t(item.title) }}</v-list-item-title>
              </v-list-item-content>
            </v-list-item>
          </v-list-item-group>
        </div>
        <div v-else>
          <v-list-item @click="select(header.action)">
            <v-list-item-content>
              <v-list-item-title>{{ $t(header.title) }}</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </div>
      </div>
    </v-list>
  </v-menu>
</template>

<script>
export default {
  name: "BaseSelectMenu",

  props: {
    items: Array,

    title: {
      type: String,
      default: "common.actions",
    },
  },

  methods: {
    select(action) {
      this.$emit("selected", action);
    },
  },
};
</script>

<style></style>
