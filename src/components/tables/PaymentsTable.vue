<template>
    <v-card>
      <v-card-title>
        Payments
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
        :items="filteredPayments"
        :search="search"
        item-value="id"
        class="elevation-1"
        :loading="isLoading"
      >
        <template v-slot:item.actions="{ item }">
          <v-btn icon @click="editPayment(item)">
            <v-icon>mdi-pencil</v-icon>
          </v-btn>
        </template>
      </v-data-table>
    </v-card>
  </template>
  
  <script>
  export default {
    props: {
      payments: {
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
          { text: 'Amount', value: 'amount' },
          { text: 'Date', value: 'date' },
          { text: 'Method', value: 'method' },
          { text: 'Actions', value: 'actions', sortable: false },
        ],
      };
    },
    computed: {
      filteredPayments() {
        return this.payments.filter(
          (payment) =>
            payment.amount.toString().includes(this.search) ||
            payment.method.toLowerCase().includes(this.search.toLowerCase())
        );
      },
    },
    methods: {
      editPayment(payment) {
        this.$emit('edit-payment', payment);
      },
    },
  };
  </script>
  