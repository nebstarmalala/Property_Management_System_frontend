<template>
  <div class="leases">
    <v-row>
      <v-col cols="12" md="6">
        <breadcrumb :items="breadcrumbs" />
      </v-col>
      <v-col cols="12" md="6" class="text-md-right">
        <v-btn color="primary" class="mb-5" @click="showAddLeaseDialog = true">
          <v-icon left size="22">{{ icons.mdiPlus }}</v-icon>
          Add Lease
        </v-btn>
      </v-col>
    </v-row>
    <v-card>
      <v-card-title>
        <v-row justify="center" class="mb-0">
          <v-col cols="12" md="4">
            <refresh-button :loading="isLeasesRefreshing" @click="refreshLeases" />
          </v-col>
          <v-col cols="12" md="4" class="text-center">
            <search-input
              v-model="search"
              :filters="searchFilters"
              :active-filter="searchColumn"
              @filter-change="onSearchFilterChange"
            />
          </v-col>
          <v-col cols="12" md="4">
            <v-select
              v-model="selectedproperty_id"
              :items="propertyOptions"
              label="Filter by Property"
              item-text="name"
              item-value="id"
              outlined
              dense
              @change="applyFilter"
            />
          </v-col>
        </v-row>
      </v-card-title>
      <v-data-table
        v-model:selected="selected"
        :headers="headers"
        :items="leases"
        :loading="isLeasesLoading"
        :options.sync="options"
        :footer-props="footerProps"
        item-key="id"
      >
        <template v-slot:item.created_at="{ item }">
          {{ formatDate(item.created_at) }}
        </template>
        <template v-slot:item.unit.unit_number="{ item }">
          {{ item.unit.unit_number }}
        </template>
        <template v-slot:item.tenant.user.name="{ item }">
          {{ item.tenant.user.name }}
        </template>
        <template v-slot:item.status="{ item }">
          {{ getStatusLabel(item.status) }}
        </template>
        <template v-slot:item.action="{ item }">
          <v-btn class="ma-1" text icon @click="editLease(item)">
            <v-icon>{{ icons.mdiPencil }}</v-icon>
          </v-btn>
          <v-btn class="ma-1" text icon @click="showDeleteLeaseDialog(item.id)">
            <v-icon>{{ icons.mdiDelete }}</v-icon>
          </v-btn>
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-btn v-bind="attrs" text icon @click="viewLease(item)" v-on="on">
                <v-icon>{{ icons.mdiEyeOutline }}</v-icon>
              </v-btn>
            </template>
            <span>View Lease</span>
          </v-tooltip>
        </template>
      </v-data-table>
    </v-card>
    <add-lease-dialog
      :show-dialog="showAddLeaseDialog"
      :lease="selectedLease"
      @close="() => { showAddLeaseDialog = false; selectedLease = {}; }"
      @lease-added="getLeases"
    />
    <confirm-dialog
      :show-dialog="showConfirmDeleteDialog"
      :is-agree-button-loading="isConfirmDeleteDialogButtonLoading"
      agree-text="Delete"
      @cancel="() => { showConfirmDeleteDialog = false; toDelete = null; }"
      @agree="deleteLease"
    />
  </div>
</template>

<script>
import axios from 'axios'
import _ from 'lodash'
import { mdiEyeOutline, mdiPencil, mdiDelete, mdiPlus } from '@mdi/js'
import BreadCrumb from '@/components/partials/BreadCrumb.vue'
import SearchInput from '@/components/partials/SearchInput.vue'
import AddLeaseDialog from '@/components/dialogs/AddLeaseDialog.vue'
import ConfirmDialog from '@/components/dialogs/ConfirmDialog.vue'
import RefreshButton from '@/components/partials/RefreshButton.vue'

export default {
  components: {
    BreadCrumb,
    SearchInput,
    AddLeaseDialog,
    ConfirmDialog,
    RefreshButton,
  },
  data() {
    return {
      showAddLeaseDialog: false,
      showConfirmDeleteDialog: false,
      isLeasesLoading: true,
      isLeasesRefreshing: false,
      isConfirmDeleteDialogButtonLoading: false,
      search: '',
      pagination: {
        page: 1,
        pageCount: 1,
        itemsPerPage: 10,
        totalItems: 0,
      },
      footerProps: {
        itemsPerPageOptions: [5, 10, 25, 50, 100],
      },
      options: {},
      headers: [
        { text: 'Unit', value: 'unit.unit_number' },
        { text: 'Tenant', value: 'tenant.user.name' },
        { text: 'Start Date', value: 'start_date' },
        { text: 'End Date', value: 'end_date' },
        { text: 'Security Deposit', value: 'security_deposit' },
        { text: 'Status', value: 'status' },
        { text: 'Action', value: 'action', sortable: false },
      ],
      searchColumn: 'tenant.user.name',
      searchFilters: [
        { text: 'Tenant Name', value: 'tenant.user.name' },
        { text: 'Unit Number', value: 'unit.unit_number' },
        { text: 'Status', value: 'status' },
      ],
      leases: [],
      selected: [],
      toDelete: null,
      leasesUrl: 'leases',
      breadcrumbs: [
        { text: 'Dashboard', disabled: false, to: { name: 'dashboard' } },
        { text: 'Leases', disabled: true },
      ],
      icons: {
        mdiEyeOutline,
        mdiPencil,
        mdiDelete,
        mdiPlus,
      },
      selectedLease: {},
      selectedproperty_id: null,
      propertyOptions: [{ id: null, name: 'All Properties' }],
    }
  },
  computed: {
    refreshTable() {
      return `${this.leasesUrl}|${this.search}|${this.sortBy}|${this.sortDesc}|${this.selectedproperty_id}`
    },
  },
  watch: {
    options: {
      handler() {
        this.getLeases()
      },
      deep: true,
    },
    refreshTable() {
      this.getLeases()
    },
    searchColumn() {
      if (this.search !== '') {
        this.getLeases()
      }
    },
  },
  mounted() {
    this.getLeases()
    this.getProperties()
  },
  methods: {
    getLeases: _.debounce(function() {
      this.isLeasesLoading = true
      const { sortBy, sortDesc, page, itemsPerPage } = this.options
      const sortOrder = sortDesc[0] ? 'desc' : 'asc'

      let url = `${this.leasesUrl}?paginate=true&sortBy=${sortBy[0]}&sortDirection=${sortOrder}&search=${this.search}&searchColumn=${this.searchColumn}&page=${page}&perPage=${itemsPerPage}`

      if (this.selectedproperty_id) {
        url += `&property_id=${this.selectedproperty_id}`
      }

      axios
        .get(url)
        .then(response => {
          this.leases = response.data.data
          this.pagination.page = response.data.current_page
          this.pagination.pageCount = response.data.last_page
          this.pagination.itemsPerPage = response.data.per_page
          this.pagination.totalItems = response.data.total
        })
        .catch(error => {
          this.$toast.error(error.response?.data?.message || 'Failed to fetch leases')
        })
        .finally(() => {
          this.isLeasesLoading = false
          this.isLeasesRefreshing = false
        })
    }, 500),
    getProperties() {
      axios
        .get('properties')
        .then(response => {
          this.propertyOptions = [
            { id: null, name: 'All Properties' },
            ...response.data.data.map(property => ({
              id: property.id,
              name: property.name,
            })),
          ]
        })
        .catch(error => {
          this.$toast.error('Failed to load properties')
        })
    },
    refreshLeases() {
      this.isLeasesRefreshing = true
      this.getLeases()
    },
    editLease(lease) {
      this.selectedLease = lease
      this.showAddLeaseDialog = true
    },
    showDeleteLeaseDialog(id) {
      this.showConfirmDeleteDialog = true
      this.toDelete = id
    },
    deleteLease() {
      this.isConfirmDeleteDialogButtonLoading = true
      axios
        .delete(`leases/${this.toDelete}`)
        .then(() => {
          this.refreshLeases()
          this.$toast.success('Lease deleted successfully')
        })
        .catch(error => {
          this.$toast.error(error.response?.data?.message || 'Failed to delete lease')
        })
        .finally(() => {
          this.showConfirmDeleteDialog = false
          this.isConfirmDeleteDialogButtonLoading = false
        })
    },
    getStatusLabel(status) {
      switch (status) {
        case 1:
          return 'Active'
        case 2:
          return 'Expired'
        case 3:
          return 'Terminated'
        default:
          return 'Unknown'
      }
    },
    formatDate(date) {
      return this.$moment(date).format('YYYY-MM-DD')
    },
    applyFilter() {
      this.getLeases()
    },
    viewLease(lease) {
      this.$router.push({ name: 'lease-details', params: { id: lease.id } })
    },
    onSearchFilterChange(newFilter) {
      this.searchColumn = newFilter
    },
  },
}
</script>
