<template>
    <v-card>
      <v-card-title>
        Invoices
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
        :items="filteredInvoices"
        :search="search"
        item-value="id"
        class="elevation-1"
        :loading="isLoading"
      >
        <template v-slot:item.actions="{ item }">
          <v-btn icon @click="editInvoice(item)">
            <v-icon>mdi-pencil</v-icon>
          </v-btn>
        </template>
      </v-data-table>
    </v-card>
  </template>
  
  <script>
  export default {
    props: {
      invoices: {
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
          { text: 'Invoice Number', value: 'invoice_number' },
          { text: 'Amount', value: 'grand_total' },
          { text: 'Billing Date', value: 'billing_date' },
          { text: 'Status', value: 'status' },
          { text: 'Actions', value: 'actions', sortable: false },
        ],
      };
    },
    computed: {
      filteredInvoices() {
        return this.invoices.filter(
          (invoice) =>
            invoice.invoice_number.includes(this.search) ||
            invoice.status.toLowerCase().includes(this.search.toLowerCase())
        );
      },
    },
    methods: {
      editInvoice(invoice) {
        this.$emit('edit-invoice', invoice);
      },
    },
  };
  </script>
  