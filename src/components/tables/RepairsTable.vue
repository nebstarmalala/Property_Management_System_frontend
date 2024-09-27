<template>
    <v-card>
      <v-card-title>
        Repairs
        <v-spacer></v-spacer>
        <v-text-field
          v-model="search"
          append-icon="mdi-magnify"
          label="Search"
          single-line
          hide-details
        />
      </v-card-title>
      <v-data-table
        :headers="headers"
        :items="filteredRepairs"
        :search="search"
        item-value="id"
        class="elevation-1"
        :loading="isLoading"
      >
        <template v-slot:item.actions="{ item }">
          <v-btn icon @click="editRepair(item)">
            <v-icon>mdi-pencil</v-icon>
          </v-btn>
        </template>
      </v-data-table>
    </v-card>
  </template>
  
  <script>
  export default {
    props: {
      repairs: {
        type: Array,
        default: () => [],
      },
      isLoading: {
        type: Boolean,
        default: false,
      },
    },
    data() {
      return {
        search: '',
        headers: [
          { text: 'Description', value: 'description' },
          { text: 'Cost', value: 'cost' },
          { text: 'Date', value: 'date' },
          { text: 'Actions', value: 'actions', sortable: false },
        ],
      };
    },
    computed: {
      filteredRepairs() {
        return this.repairs.filter(
          (repair) =>
            repair.description.toLowerCase().includes(this.search.toLowerCase())
        );
      },
    },
    methods: {
      editRepair(repair) {
        this.$emit('edit-repair', repair);
      },
    },
  };
  </script>
  