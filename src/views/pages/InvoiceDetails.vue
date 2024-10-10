<template>
  <div class="invoice-details">
    <v-row>
      <v-col md="10" sm="12">
        <breadcrumb :items="breadcrumbs" />
      </v-col>
      <v-col md="2" sm="12">
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-btn class="go-back-button" v-bind="attrs" v-on="on" @click="$router.go(-1)">
              <v-icon left>{{ icons.mdiArrowLeft }}</v-icon>
              Go back
            </v-btn>
          </template>
          <span>Go back to the previous page</span>
        </v-tooltip>
      </v-col>
    </v-row>

    <!-- Error Alert -->
    <v-alert v-if="errorMessage" type="error" dismissible>
      {{ errorMessage }}
    </v-alert>

    <!-- Invoice Details Card -->
    <v-card class="mb-4">
      <v-card-title>INVOICE DETAILS</v-card-title>
      <v-card-text v-if="invoiceDetails">
        <v-row>
          <v-col md="4" sm="12">
            <p><strong>Invoice Number:</strong> {{ invoiceDetails.invoice_number }}</p>
            <p><strong>Ref:</strong> {{ invoiceDetails.ref }}</p>
            <p><strong>Status:</strong> {{ formatInvoiceStatus(invoiceDetails.status) }}</p>
          </v-col>
          <v-col md="4" sm="12">
            <p><strong>Amount:</strong> {{ invoiceDetails.amount }}</p>
            <p><strong>Tax:</strong> {{ invoiceDetails.tax }}</p>
            <p><strong>Discount:</strong> {{ invoiceDetails.discount }}</p>
          </v-col>
          <v-col md="4" sm="12">
            <p><strong>Grand Total:</strong> {{ invoiceDetails.grand_total }}</p>
            <p><strong>Billing Date:</strong> {{ formatDate(invoiceDetails.billing_date) }}</p>
            <p><strong>Due Date:</strong> {{ formatDate(invoiceDetails.due_date) }}</p>
          </v-col>
        </v-row>
      </v-card-text>
      <v-card-text v-else>
        <v-progress-circular indeterminate color="primary"></v-progress-circular>
        Loading invoice details...
      </v-card-text>
    </v-card>

    <!-- Payments Section -->
    <v-card class="mb-4">
      <v-card-title>
        Payments
        <v-spacer></v-spacer>
        <v-text-field
          v-model="paymentSearch"
          append-icon="mdi-magnify"
          label="Search payments"
          single-line
          hide-details
          class="mr-4"
        ></v-text-field>
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" v-bind="attrs" v-on="on" @click="showAddPaymentDialog = true">
              <v-icon left>{{ icons.mdiPlus }}</v-icon>
              Add Payment
            </v-btn>
          </template>
          <span>Add a new payment</span>
        </v-tooltip>
      </v-card-title>
      <v-data-table
        :headers="paymentHeaders"
        :items="payments"
        :search="paymentSearch"
        :loading="isPaymentsLoading"
        :options.sync="paymentOptions"
        :footer-props="footerProps"
        item-key="id"
      >
        <template v-slot:item.payment_date="{ item }">
          {{ formatDate(item.payment_date) }}
        </template>
        <template v-slot:item.payment_method="{ item }">
          {{ formatPaymentMethod(item.payment_method) }}
        </template>
        <template v-slot:item.action="{ item }">
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-btn class="ma-1" text icon v-bind="attrs" v-on="on" @click="editPayment(item)">
                <v-icon>{{ icons.mdiPencil }}</v-icon>
              </v-btn>
            </template>
            <span>Edit Payment</span>
          </v-tooltip>
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-btn class="ma-1" text icon v-bind="attrs" v-on="on" @click="showDeletePaymentDialog(item.id)">
                <v-icon>{{ icons.mdiDelete }}</v-icon>
              </v-btn>
            </template>
            <span>Delete Payment</span>
          </v-tooltip>
        </template>
      </v-data-table>
    </v-card>

    <add-payment-dialog
      :show-dialog.sync="showAddPaymentDialog"
      :payment="selectedPayment"
      :invoice-id="$route.params.id"
      @close="resetPaymentDialog"
      @payment-added="fetchPayments"
    />

    <v-dialog v-model="deletePaymentDialog" max-width="400">
      <v-card>
        <v-card-title class="headline">Delete Payment</v-card-title>
        <v-card-text>Are you sure you want to delete this payment?</v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="red" @click="deletePayment()">Delete</v-btn>
          <v-btn @click="deletePaymentDialog = false">Cancel</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import axios from 'axios'
import { mdiPlus, mdiPencil, mdiDelete, mdiArrowLeft } from '@mdi/js'
import BreadCrumb from '@/components/partials/BreadCrumb.vue'
import AddPaymentDialog from '@/components/dialogs/AddPaymentDialog.vue'

export default {
  components: {
    breadcrumb: BreadCrumb,
    AddPaymentDialog,
  },
  data() {
    return {
      showAddPaymentDialog: false,
      selectedPayment: {},
      deletePaymentDialog: false,
      invoiceDetails: null,
      payments: [],
      paymentSearch: '',
      paymentOptions: {},
      isPaymentsLoading: true, // loading state for payments
      isLoading: true, // loading state for invoice details
      errorMessage: '',
      paymentHeaders: [
        { text: 'Payment Number', value: 'payment_number' },
        { text: 'Amount', value: 'amount' },
        { text: 'Reference', value: 'reference' },
        { text: 'Payment Method', value: 'payment_method' },
        { text: 'Payment Date', value: 'payment_date' },
        { text: 'Action', value: 'action', sortable: false },
      ],
      footerProps: {
        itemsPerPageOptions: [5, 10, 25, 50],
      },
      icons: {
        mdiPlus,
        mdiPencil,
        mdiDelete,
        mdiArrowLeft,
      },
      breadcrumbs: [
        { text: 'Dashboard', disabled: false, to: { name: 'dashboard' } },
        { text: 'Invoices', disabled: false},
        { text: 'Invoice Details', disabled: true },
      ],
    }
  },
  methods: {
    formatDate(date) {
      return new Date(date).toLocaleDateString()
    },
    formatInvoiceStatus(status) {
      switch (status) {
        case 1:
          return 'UNPAID'
        case 2:
          return 'PAID'
        case 3:
          return 'OVERDUE'
        case 4:
          return 'PARTIALLY PAID'
        default:
          return 'UNKNOWN'
      }
    },
    formatPaymentMethod(method) {
      switch (method) {
        case 1:
          return 'MPESA'
        case 2:
          return 'CREDIT CARD'
        case 3:
          return 'BANK TRANSFER'
        case 4:
          return 'CASH'
        case 5:
          return 'PAYPAL'
        default:
          return 'UNKNOWN'
      }
    },
    async getInvoiceDetails() {
      this.isLoading = true // set loading state
      try {
        const response = await axios.get(`/invoices/${this.$route.params.id}`)
        this.invoiceDetails = response.data
      } catch (error) {
        this.errorMessage = 'Failed to load invoice details. Please try again later.'
        console.error('Error fetching invoice details:', error)
      } finally {
        this.isLoading = false // stop loading
      }
    },
    async fetchPayments(
      page = 1,
      itemsPerPage = 10,
      sortBy = 'payment_date',
      sortOrder = 'desc',
      search = '',
      searchColumn = ''
    ) {
      this.isPaymentsLoading = true // set loading state
      try {
        const response = await axios.get(
          `/payments?paginate=true&sortBy=${sortBy}&sortDirection=${sortOrder}&search=${search}&searchColumn=${searchColumn}&page=${page}&perPage=${itemsPerPage}&invoice_id=${this.$route.params.id}`
        )
        this.payments = response.data.data
      } catch (error) {
        this.errorMessage = 'Failed to load payments. Please try again later.'
        console.error('Failed to fetch payments:', error)
      } finally {
        this.isPaymentsLoading = false // stop loading
      }
    },
    resetPaymentDialog() {
      this.showAddPaymentDialog = false
      this.selectedPayment = {}
    },
    editPayment(payment) {
      this.selectedPayment = payment
      this.showAddPaymentDialog = true
    },
    showDeletePaymentDialog(paymentId) {
      this.selectedPayment.id = paymentId
      this.deletePaymentDialog = true
    },
    async deletePayment() {
      try {
        await axios.delete(`/payments/${this.selectedPayment.id}`)
        this.fetchPayments()
      } catch (error) {
        console.error('Error deleting payment:', error)
      } finally {
        this.deletePaymentDialog = false
      }
    },
  },
  async created() {
    this.getInvoiceDetails()
    this.fetchPayments()
  },
}
</script>

<style scoped>
.go-back-button {
  float: right;
  margin-top: 10px;
}
</style>
