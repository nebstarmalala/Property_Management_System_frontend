<template>
  <div class="lease-details">
    <!-- Invoices Section -->
    <v-card class="mb-5">
      <v-card-title>
        <v-row justify="space-between" align="center">
          <v-col cols="12" md="3">
            <h3>Invoices</h3>
          </v-col>
          <v-col cols="12" md="6">
            <search-input
              v-model="invoiceSearch"
              :filters="invoiceSearchFilters"
              :active-filter="invoiceSearchColumn"
              @filter-change="onInvoiceSearchFilterChange"
            />
          </v-col>
          <v-col cols="12" md="3" class="text-right">
            <refresh-button :loading="isInvoicesRefreshing" @click="refreshInvoices" />
          </v-col>
        </v-row>
      </v-card-title>
      <v-data-table
        :headers="invoiceHeaders"
        :items="invoices"
        :loading="isInvoicesLoading"
        :options.sync="invoiceOptions"
        :server-items-length="invoicePagination.totalItems"
        :footer-props="footerProps"
        item-key="id"
      >
        <template v-slot:item.billing_date="{ item }">
          {{ formatDate(item.billing_date) }}
        </template>
        <template v-slot:item.due_date="{ item }">
          {{ formatDate(item.due_date) }}
        </template>
        <template v-slot:item.grand_total="{ item }">
          {{ formatCurrency(item.grand_total) }}
        </template>
        <template v-slot:item.status="{ item }">
          {{ getInvoiceStatusLabel(item.status) }}
        </template>
        <template v-slot:item.action="{ item }">
          <v-btn class="ma-1" text icon @click="viewInvoice(item)">
            <v-icon>{{ icons.mdiEyeOutline }}</v-icon>
          </v-btn>
        </template>
      </v-data-table>
    </v-card>

    <!-- Repairs Section -->
    <v-card>
      <v-card-title>
        <v-row justify="space-between" align="center">
          <v-col cols="12" md="3">
            <h3>Repairs</h3>
          </v-col>
          <v-col cols="12" md="6">
            <search-input
              v-model="repairSearch"
              :filters="repairSearchFilters"
              :active-filter="repairSearchColumn"
              @filter-change="onRepairSearchFilterChange"
            />
          </v-col>
          <v-col cols="12" md="3" class="text-right">
            <refresh-button :loading="isRepairsRefreshing" @click="refreshRepairs" />
          </v-col>
        </v-row>
      </v-card-title>
      <v-data-table
        :headers="repairHeaders"
        :items="repairs"
        :loading="isRepairsLoading"
        :options.sync="repairOptions"
        :server-items-length="repairPagination.totalItems"
        :footer-props="footerProps"
        item-key="id"
      >
        <template v-slot:item.date="{ item }">
          {{ formatDate(item.date) }}
        </template>
        <template v-slot:item.cost="{ item }">
          {{ formatCurrency(item.cost) }}
        </template>
        <template v-slot:item.status="{ item }">
          {{ getRepairStatusLabel(item.status) }}
        </template>
        <template v-slot:item.action="{ item }">
          <v-btn class="ma-1" text icon @click="viewRepair(item)">
            <v-icon>{{ icons.mdiEyeOutline }}</v-icon>
          </v-btn>
        </template>
      </v-data-table>
    </v-card>
  </div>
</template>

<script>
import axios from 'axios'
import _ from 'lodash'
import { mdiEyeOutline } from '@mdi/js'
import SearchInput from '@/components/partials/SearchInput.vue'
import RefreshButton from '@/components/partials/RefreshButton.vue'

export default {
  components: {
    SearchInput,
    RefreshButton,
  },
  data() {
    return {
      leaseId: null,
      // Shared
      footerProps: {
        itemsPerPageOptions: [5, 10, 25, 50, 100],
      },

      // Invoices Section
      invoiceSearch: '',
      invoiceSearchColumn: 'lease_id',
      invoiceSearchFilters: [
        { text: 'Invoice Number', value: 'invoice_number' },
        { text: 'Billing Date', value: 'billing_date' },
        { text: 'Due Date', value: 'due_date' },
      ],
      isInvoicesLoading: false,
      isInvoicesRefreshing: false,
      invoices: [],
      invoicesUrl: 'invoices',
      invoiceHeaders: [
        { text: 'Invoice Number', value: 'invoice_number' },
        { text: 'Billing Date', value: 'billing_date' },
        { text: 'Due Date', value: 'due_date' },
        { text: 'Grand Total', value: 'grand_total' },
        { text: 'Status', value: 'status' },
        { text: 'Action', value: 'action', sortable: false },
      ],
      invoiceOptions: {},
      invoicePagination: {
        page: 1,
        pageCount: 1,
        itemsPerPage: 10,
        totalItems: 0,
      },

      // Repairs Section
      repairSearch: '',
      repairSearchColumn: 'repair_number',
      repairSearchFilters: [
        { text: 'Repair Number', value: 'repair_number' },
        { text: 'Description', value: 'description' },
        { text: 'Date', value: 'date' },
      ],
      isRepairsLoading: false,
      isRepairsRefreshing: false,
      repairs: [],
      repairsUrls: 'repairs',
      repairHeaders: [
        { text: 'Repair Number', value: 'repair_number' },
        { text: 'Description', value: 'description' },
        { text: 'Date', value: 'date' },
        { text: 'Cost', value: 'cost' },
        { text: 'Status', value: 'status' },
        { text: 'Action', value: 'action', sortable: false },
      ],
      repairOptions: {},
      repairPagination: {
        page: 1,
        pageCount: 1,
        itemsPerPage: 10,
        totalItems: 0,
      },

      // Icons
      icons: {
        mdiEyeOutline,
      },
    }
  },
  computed: {
    refreshInvoiceTable() {
      return `invoices|${this.invoiceSearch}|${this.invoiceOptions.sortBy}|${this.invoiceOptions.sortDesc}|${this.invoiceSearchColumn}|${this.leaseId}`
    },
    refreshRepairTable() {
      return `repairs|${this.repairSearch}|${this.repairOptions.sortBy}|${this.repairOptions.sortDesc}|${this.repairSearchColumn}|${this.leaseId}`
    },
  },
  watch: {
    invoiceOptions: {
      handler() {
        this.getInvoices()
      },
      deep: true,
    },
    repairOptions: {
      handler() {
        this.getRepairs()
      },
      deep: true,
    },
    refreshInvoiceTable() {
      this.getInvoices()
    },
    refreshRepairTable() {
      this.getRepairs()
    },
  },
  created() {
    this.leaseId = this.$route.params.id
  },
  mounted() {
    this.getInvoices()
    this.getRepairs()
  },
  methods: {
    // Invoices Section Methods
    getInvoices: _.debounce(function() {
      this.isInvoicesLoading = true
      const { sortBy, sortDesc, page, itemsPerPage } = this.invoiceOptions
      const sortOrder = sortDesc[0] ? 'desc' : 'asc'
      axios
        .get(
          `${this.invoicesUrl}?paginate=true&sortBy=${sortBy[0]}&sortDirection=${sortOrder}&search=${this.invoiceSearch}&searchColumn=${this.invoiceSearchColumn}&page=${page}&perPage=${itemsPerPage}&lease_id=${this.leaseId}`,
        )
        .then(response => {
          this.invoices = response.data.data
          this.invoicePagination.page = response.data.current_page
          this.invoicePagination.pageCount = response.data.last_page
          this.invoicePagination.itemsPerPage = response.data.per_page
          this.invoicePagination.totalItems = response.data.total
        })
        .catch(error => {
          this.$toast.error(error.response.data.message)
        })
        .finally(() => {
          this.isInvoicesLoading = false
          this.isInvoicesRefreshing = false
        })
    }, 500),

    refreshInvoices() {
      this.isInvoicesRefreshing = true
      this.getInvoices()
    },

    onInvoiceSearchFilterChange(newFilter) {
      this.invoiceSearchColumn = newFilter
    },

    // Repairs Section Methods
    getRepairs: _.debounce(function() {
      this.isRepairsLoading = true
      const { sortBy, sortDesc, page, itemsPerPage } = this.repairOptions
      const sortOrder = sortDesc[0] ? 'desc' : 'asc'
      axios
        .get(
          `${this.repairsUrls}?paginate=true&sortBy=${sortBy[0]}&sortDirection=${sortOrder}&search=${this.repairSearch}&searchColumn=${this.repairSearchColumn}&page=${page}&perPage=${itemsPerPage}&lease_id=${this.leaseId}`,
        )
        .then(response => {
          this.repairs = response.data.data
          this.repairPagination.page = response.data.current_page
          this.repairPagination.pageCount = response.data.last_page
          this.repairPagination.itemsPerPage = response.data.per_page
          this.repairPagination.totalItems = response.data.total
        })
        .catch(error => {
          this.$toast.error(error.response.data.message)
        })
        .finally(() => {
          this.isRepairsLoading = false
          this.isRepairsRefreshing = false
        })
    }, 500),

    refreshRepairs() {
      this.isRepairsRefreshing = true
      this.getRepairs()
    },

    onRepairSearchFilterChange(newFilter) {
      this.repairSearchColumn = newFilter
    },

    getInvoiceStatusLabel(status) {
      return status === 1 ? 'Paid' : 'Unpaid'
    },

    getRepairStatusLabel(status) {
      return status === 1 ? 'Completed' : 'Pending'
    },

    formatCurrency(amount) {
      return new Intl.NumberFormat('en-KE', { style: 'currency', currency: 'KES' }).format(amount)
    },

    formatDate(date) {
      return new Intl.DateTimeFormat('en-KE', { dateStyle: 'medium' }).format(new Date(date))
    },

    viewInvoice(invoice) {
      console.log('View invoice:', invoice)
    },

    viewRepair(repair) {
      console.log('View repair:', repair)
    },
  },
}
</script>

<style scoped>
.lease-details {
  max-width: 100%;
}
</style>
