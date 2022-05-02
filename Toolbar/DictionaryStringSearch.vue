<template>
  <v-select
    v-model="val"
    :items="dict(dictionary)"
    :label="label"
    :loading="loading"
    solo
    dense
    single-line
    hide-details
    clearable
    no-data-text="Нет данных"
    class="shrink"
    style="width: 400px"
  ></v-select>
</template>

<script>
import { mapGetters } from "vuex";
import store from "@/store";
import { DICT_FETCH } from "@/store/actions.type";

export default {
  name: "DictionarySearch",

  props: {
    value: String,
    label: String,
    dictionary: String,
  },

  data: () => ({
    loading: true,
  }),

  mounted() {
    if (!this.dict[this.dictionary]) {
      store.dispatch(DICT_FETCH, this.dictionary).then(() => {
        this.loading = false;
      });
    } else this.loading = false;
  },

  computed: {
    ...mapGetters(["dict"]),
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
