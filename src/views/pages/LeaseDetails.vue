<template>
  <div class="lease-details">
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

    <!-- Lease Details Card -->
    <v-card class="mb-4">
      <v-card-title>Lease Details</v-card-title>
      <v-card-text v-if="leaseDetails">
        <v-row>
          <v-col md="6" sm="12">
            <p><strong>Lease Number:</strong> {{ leaseDetails.lease_number }}</p>
            <p><strong>Property:</strong> {{ leaseDetails.unit.property.name }}</p>
            <p><strong>Unit:</strong> {{ leaseDetails.unit.unit_number }}</p>
            <p><strong>Status:</strong> {{ formatStatus(leaseDetails.status) }}</p>
          </v-col>
          <v-col md="6" sm="12">
            <p><strong>Tenant:</strong> {{ leaseDetails.tenant.user.name }}</p>
            <p><strong>Tenant Phone Number:</strong> {{ leaseDetails.tenant.user.phone_number }}</p>
            <p><strong>Start Date:</strong> {{ formatDate(leaseDetails.start_date) }}</p>
            <p><strong>End Date:</strong> {{ formatDate(leaseDetails.end_date) }}</p>
          </v-col>
        </v-row>
      </v-card-text>
      <v-card-text v-else>
        <v-progress-circular indeterminate color="primary"></v-progress-circular>
        Loading lease details...
      </v-card-text>
    </v-card>

    <!-- Invoices Section -->
    <v-card class="mb-4">
      <v-card-title>
        Invoices
        <v-spacer></v-spacer>
        <v-text-field
          v-model="invoiceSearch"
          append-icon="mdi-magnify"
          label="Search invoices"
          single-line
          hide-details
          class="mr-4"
        ></v-text-field>
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" v-bind="attrs" v-on="on" @click="showAddInvoiceDialog = true">
              <v-icon left>{{ icons.mdiPlus }}</v-icon>
              Add Invoice
            </v-btn>
          </template>
          <span>Add a new invoice</span>
        </v-tooltip>
      </v-card-title>
      <v-data-table
        :headers="invoiceHeaders"
        :items="invoices"
        :search="invoiceSearch"
        :loading="isInvoicesLoading"
        :options.sync="invoiceOptions"
        :footer-props="footerProps"
        item-key="id"
      >
        <template v-slot:item.billing_date="{ item }">
          {{ formatDate(item.billing_date) }}
        </template>
        <template v-slot:item.due_date="{ item }">
          {{ formatDate(item.due_date) }}
        </template>
        <template v-slot:item.action="{ item }">
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-btn class="ma-1" text icon v-bind="attrs" v-on="on" @click="editInvoice(item)">
                <v-icon>{{ icons.mdiPencil }}</v-icon>
              </v-btn>
            </template>
            <span>Edit Invoice</span>
          </v-tooltip>
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-btn class="ma-1" text icon v-bind="attrs" v-on="on" @click="showDeleteInvoiceDialog(item.id)">
                <v-icon>{{ icons.mdiDelete }}</v-icon>
              </v-btn>
            </template>
            <span>Delete Invoice</span>
          </v-tooltip>
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-btn class="ma-1" text icon v-bind="attrs" v-on="on" @click="viewInvoice(item)">
                <v-icon>{{ icons.mdiEyeOutline }}</v-icon>
              </v-btn>
            </template>
            <span>View Invoice</span>
          </v-tooltip>
        </template>
      </v-data-table>
    </v-card>

    <!-- Repairs Section -->
    <v-card>
      <v-card-title>
        Repairs
        <v-spacer></v-spacer>
        <v-text-field
          v-model="repairSearch"
          append-icon="mdi-magnify"
          label="Search repairs"
          single-line
          hide-details
          class="mr-4"
        ></v-text-field>
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" v-bind="attrs" v-on="on" @click="addRepair">
              <v-icon left>{{ icons.mdiPlus }}</v-icon>
              Add Repair
            </v-btn>
          </template>
          <span>Add a new repair</span>
        </v-tooltip>
      </v-card-title>
      <v-data-table
        :headers="repairHeaders"
        :items="repairs"
        :search="repairSearch"
        :loading="isRepairsLoading"
        :options.sync="repairOptions"
        :footer-props="footerProps"
        item-key="id"
      >
        <template v-slot:item.repair_date="{ item }">
          {{ formatDate(item.repair_date) }}
        </template>
        <template v-slot:item.action="{ item }">
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-btn class="ma-1" text icon v-bind="attrs" v-on="on" @click="editRepair(item)">
                <v-icon>{{ icons.mdiPencil }}</v-icon>
              </v-btn>
            </template>
            <span>Edit Repair</span>
          </v-tooltip>
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-btn class="ma-1" text icon v-bind="attrs" v-on="on" @click="showDeleteRepairDialog(item.id)">
                <v-icon>{{ icons.mdiDelete }}</v-icon>
              </v-btn>
            </template>
            <span>Delete Repair</span>
          </v-tooltip>
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-btn class="ma-1" text icon v-bind="attrs" v-on="on" @click="viewRepair(item)">
                <v-icon>{{ icons.mdiEyeOutline }}</v-icon>
              </v-btn>
            </template>
            <span>View Repair</span>
          </v-tooltip>
        </template>
      </v-data-table>
    </v-card>
    <add-invoice-dialog
    :show-dialog.sync="showAddInvoiceDialog"
    :invoice="selectedInvoice"
    :lease-id="$route.params.id"
    @close="() => { showAddInvoiceDialog = false; selectedInvoice = {}; }"
    @invoice-added="fetchInvoices"
  />
  
  <add-repair-dialog
    :show-dialog.sync="showAddRepairDialog"
    :repair="selectedRepair"
    :lease-id="$route.params.id"
    @close="() => { showAddRepairDialog = false; selectedRepair = {}; }"
    @repair-added="fetchRepairs"
  />
  </div>
</template>


<script>
import axios from 'axios';
import { mdiEyeOutline, mdiArrowLeft, mdiPencil, mdiDelete, mdiPlus } from '@mdi/js';
import BreadCrumb from '@/components/partials/BreadCrumb.vue';
import AddInvoiceDialog from '@/components/dialogs/AddInvoiceDialog.vue';

export default {
  components: {
    breadcrumb: BreadCrumb,
    AddInvoiceDialog
  },
  data() {
    return {
      showAddInvoiceDialog: false,
      showAddRepairDialog: false,
      selectedInvoice: {},
      selectedRepair: {},
      leaseDetails: null,
      invoiceSearch: '',
      repairSearch: '',
      invoiceOptions: {},
      repairOptions: {},
      isInvoicesLoading: true,
      isRepairsLoading: true,
      invoices: [],
      repairs: [],
      errorMessage: '',
      invoiceHeaders: [
        { text: 'Invoice Number', value: 'invoice_number' },
        { text: 'Amount', value: 'amount' },
        { text: 'Tax', value: 'tax' },
        { text: 'Discount', value: 'discount' },
        { text: 'Total Amount', value: 'grand_total' },
        { text: 'Billing Date', value: 'billing_date' },
        { text: 'Due Date', value: 'due_date' },
        { text: 'Action', value: 'action', sortable: false },
      ],
      repairHeaders: [
        { text: 'category', value: 'category' },
        { text: 'Description', value: 'description' },
        { text: 'Status', value: 'status' },
        { text: 'Priority', value: 'priority' },
        { text: 'Date', value: 'repair_date' },
        { text: 'Action', value: 'action', sortable: false },
      ],
      footerProps: {
        itemsPerPageOptions: [5, 10, 25, 50, 100],
      },
      icons: {
        mdiEyeOutline,
        mdiArrowLeft,
        mdiPencil,
        mdiDelete,
        mdiPlus,
      },
      breadcrumbs: [
        { text: 'Dashboard', disabled: false, to: { name: 'dashboard' } },
        { text: 'Leases', disabled: false, to: { name: 'leases' } },
        { text: 'Lease Details', disabled: true },
      ],
    };
  },
  methods: {
    formatDate(date) {
      if (!date) return '';
      return new Date(date).toLocaleDateString();
    },
    formatStatus(status) {
      switch (status) {
        case 1:
          return 'ACTIVE';
        case 2:
          return 'EXPIRED';
        case 3:
          return 'TERMINATED';
        default:
          return 'UNKNOWN';
      }
    },
    async fetchInvoices(page = 1, itemsPerPage = 10, sortBy = 'billing_date', sortOrder = 'desc', search = '', searchColumn = '') {
      this.isInvoicesLoading = true;
      this.errorMessage = '';
      try {
        const response = await axios.get(
          `/invoices?paginate=true&sortBy=${sortBy}&sortDirection=${sortOrder}&search=${search}&searchColumn=${searchColumn}&page=${page}&perPage=${itemsPerPage}&lease_id=${this.$route.params.id}`,
        );
        this.invoices = response.data.data;
        console.log('Invoices loaded:', this.invoices);
      } catch (error) {
        this.errorMessage = 'Failed to load invoices. Please try again later.';
        console.error('Failed to fetch invoices:', error);
      } finally {
        this.isInvoicesLoading = false;
      }
    },
    async fetchRepairs(page = 1, itemsPerPage = 10, sortBy = 'repair_date', sortOrder = 'desc', search = '', searchColumn = '') {
      this.isRepairsLoading = true;
      this.errorMessage = ''; 
      try {
        const response = await axios.get(
          `/repairs?paginate=true&sortBy=${sortBy}&sortDirection=${sortOrder}&search=${search}&searchColumn=${searchColumn}&page=${page}&perPage=${itemsPerPage}&lease_id=${this.$route.params.id}`,
        );
        this.repairs = response.data.data;
        console.log('Repairs loaded:', this.repairs);
      } catch (error) {
        this.errorMessage = 'Failed to load repairs. Please try again later.';
        console.error('Failed to fetch repairs:', error);
      } finally {
        this.isRepairsLoading = false;
      }
    },
    async getLeaseDetails() {
      try {
        const response = await axios.get(`/leases/${this.$route.params.id}`);
        this.leaseDetails = response.data;
      } catch (error) {
        console.error('Error fetching lease details:', error);
        this.errorMessage = 'Failed to load lease details. Please try again later.';
      }
    },
    addInvoice() {
      this.selectedInvoice = { lease_id: this.$route.params.id };
      this.showAddInvoiceDialog = true;
    },
    editInvoice(invoice) {
      this.selectedInvoice = { ...invoice, lease_id: this.$route.params.id };
      this.showAddInvoiceDialog = true;
    },
    addRepair() {
      this.selectedRepair = { lease_id: this.$route.params.id };
      this.showAddRepairDialog = true;
    },
    editRepair(repair) {
      this.selectedRepair = { ...repair, lease_id: this.$route.params.id };
      this.showAddRepairDialog = true;
    },
    showDeleteRepairDialog(id) {
      // Implement delete repair dialog logic
      console.log('Delete repair:', id);
    },
  },
  async mounted() {
    await this.getLeaseDetails();
    this.fetchInvoices();
    this.fetchRepairs();
  },
};
</script>

<style scoped>
.go-back-button {
  margin-top: 10px;
}

.lease-details {
  margin: 20px;
}
</style>