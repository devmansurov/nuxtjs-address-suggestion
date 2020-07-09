<template>
  <div>
    <v-layout row justify-center align-center>
      <v-flex xs12 sm8 md6>
        <v-card>
          <v-card-title class="headline">Сервис подсказок адресов</v-card-title>
          <v-card-text>Используется Yandex Maps API для поиска адресов.</v-card-text>
          <v-card-text>
            <v-autocomplete
              v-model="model"
              :items="items"
              :loading="isLoading"
              :search-input.sync="search"
              color="dark"
              :hide-no-data="!model || !items.length"
              item-text="name"
              label="Yandex Maps API"
              placeholder="Напишите адрес доставки товара"
              prepend-icon="mdi-map-marker"
              return-object
              eager
              :filter="filter"
            ></v-autocomplete>
          </v-card-text>
          <v-divider></v-divider>
          <v-expand-transition>
            <v-list v-if="model">
              <v-list-item v-for="(field, i) in fields" :key="i">
                <v-list-item-content>
                  <v-list v-if="Array.isArray(field.value)">
                    <v-list-item v-for="(field, i) in field.value" :key="i">
                      <v-list-item-content>
                        <v-list-item-title>{{field.name}}</v-list-item-title>
                        <v-list-item-subtitle v-text="field.kind"></v-list-item-subtitle>
                      </v-list-item-content>
                    </v-list-item>
                  </v-list>
                  <v-list-item-title v-else>{{field.value}}</v-list-item-title>
                  <v-list-item-subtitle v-text="field.key" v-show="field.key != 'meta'"></v-list-item-subtitle>
                </v-list-item-content>
              </v-list-item>
            </v-list>
          </v-expand-transition>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn :disabled="!model" color="dark lighten-3" @click="model = null">
              Очистить
              <v-icon right>mdi-close-circle</v-icon>
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-flex>
    </v-layout>
  </div>
</template>

<script>
import _ from "lodash";
import axios from "axios";

export default {
  head: {
    meta: [
      { charset: "utf-8" },
      { name: "viewport", content: "width=device-width, initial-scale=1" }
    ],
    title: "Home"
  },
  data: () => ({
    locations: [],
    model: null,
    search: null,
    isLoading: false
  }),

  computed: {
    apiUrl() {
      return process.env.apiUrl;
    },
    fields() {
      if (!this.model) return [];
      return Object.keys(this.model).map(key => {
        return {
          key,
          value: this.model[key] || "n/a"
        };
      });
    },
    items() {
      return this.locations.map(location => {
        const name = (location.name =
          location.name +
          (location.description ? ", " + location.description : ""));

        return Object.assign({}, location, { name });
      });
    }
  },

  watch: {
    search(val) {
      this.isLoading = true;
      if (val && val.length > 2) {
        this.performSearch(val);
      }
    }
  },

  methods: {
    performSearch: _.debounce(function(val) {
      axios.get(this.apiUrl + val).then(res => {
        this.isLoading = false;
        this.locations = res.data;
      });
    }, 300),

    filter(item, queryText, itemText) {
      /**
       * Return true because Yandex can return correct results
       * and we not need to custom filter
       */

      return true;
    }
  }
};
</script>
