<template>
    <div class="leases">
      <v-row>
        <v-col md="10" sm="12">
          <breadcrumb :items="breadcrumbs" />
        </v-col>
        <v-col md="2" sm="12">
          <v-btn class="go-back-button" @click="$router.go(-1)">
            <v-icon class="mr-1">{{ icons.mdiArrowLeft }}</v-icon>
            Go back
          </v-btn>
        </v-col>
      </v-row>
      <v-card flat class="pa-3 mt-2">
        <h1 class="text-center">Leases</h1>
        <p class="text-center">Manage your leases efficiently.</p>
        <v-row>
          <v-spacer></v-spacer>
          <v-col class="text-right mt-3 mr-8" cols="6">
            <v-btn color="primary" @click="showAddLeaseDialog = true">
              <v-icon left size="22">{{ icons.mdiPlus }}</v-icon>
              Add Lease
            </v-btn>
          </v-col>
        </v-row>
        <v-row justify="center" class="mb-0">
          <v-col class="col-md-6 text-center">
            <search-input
              v-model="search"
              :filters="searchFilters"
              :active-filter="searchColumn"
              @filter-change="onSearchFilterChange"
            />
          </v-col>
        </v-row>
        <v-data-table
          v-model="selected"
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
          <template v-slot:item.action="{ item }">
            <v-btn class="ma-1" text icon @click="editLease(item)">
              <v-icon>{{ icons.mdiPencil }}</v-icon>
            </v-btn>
            <v-btn class="ma-1" text icon @click="showDeleteLeaseDialog(item.id)">
              <v-icon>{{ icons.mdiDelete }}</v-icon>
            </v-btn>
            <v-tooltip bottom>
              <template v-slot:activator="{ on, attrs }">
                <v-btn ma-3 v-bind="attrs" text icon @click="viewLease(item)" v-on="on">
                  <v-icon v-bind="attrs" v-on="on">{{ icons.mdiEyeOutline }}</v-icon>
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
        @close="
          showAddLeaseDialog = false
          selectedLease = {}
        "
        @lease-added="getLeases"
      />
      <confirm-dialog
        :show-dialog="showConfirmDeleteDialog"
        :is-agree-button-loading="isConfirmDeleteDialogButtonLoading"
        :agree-text="'Delete'"
        @cancel="
          showConfirmDeleteDialog = false
          toDelete = null
        "
        @agree="deleteLease"
      />
    </div>
  </template>
  
  <script>
  import axios from 'axios'
  import _ from 'lodash'
  import { mdiEyeOutline, mdiArrowLeft, mdiPencil, mdiDelete, mdiPlus } from '@mdi/js'
  import BreadCrumb from '@/components/partials/BreadCrumb.vue'
  import SearchInput from '@/components/partials/SearchInput.vue'
  import AddLeaseDialog from '@/components/dialogs/AddLeaseDialog.vue'
  import ConfirmDialog from '@/components/dialogs/ConfirmDialog.vue'
  
  export default {
    components: {
      breadcrumb: BreadCrumb,
      SearchInput,
      AddLeaseDialog,
      ConfirmDialog,
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
          { text: 'Lease Number', value: 'lease_number' },
          { text: 'Unit', value: 'unit.unit_number' },
          { text: 'Tenant', value: 'tenant.user.name' },
          { text: 'Start Date', value: 'start_date' },
          { text: 'End Date', value: 'end_date' },
          { text: 'Security Deposit', value: 'security_deposit' },
          { text: 'Status', value: 'status' },
          { text: 'Action', value: 'action', sortable: false },
        ],
        searchColumn: 'lease_number',
        searchFilters: [
          { text: 'Lease Number', value: 'lease_number' },
          { text: 'Tenant Name', value: 'tenant.user.name' },
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
          mdiArrowLeft,
          mdiPencil,
          mdiDelete,
          mdiPlus,
        },
        selectedLease: {},
      }
    },
    computed: {
      refreshTable() {
        return `${this.leasesUrl}|${this.search}|${this.sortBy}|${this.sortDesc}`
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
    },
    methods: {
      getLeases: _.debounce(function() {
        this.isLeasesLoading = true
        const { sortBy, sortDesc, page, itemsPerPage } = this.options
        const sortOrder = sortDesc[0] ? 'desc' : 'asc'
        axios
          .get(
            `${this.leasesUrl}?paginate=true&sortBy=${sortBy[0]}&sortDirection=${sortOrder}&search=${this.search}&searchColumn=${this.searchColumn}&page=${page}&perPage=${itemsPerPage}`,
          )
          .then(response => {
            this.leases = response.data.data
            this.pagination.page = response.data.current_page
            this.pagination.pageCount = response.data.last_page
            this.pagination.itemsPerPage = response.data.per_page
            this.pagination.totalItems = response.data.total
          })
          .catch(error => {
            this.$toast.error(error.response.data.message)
          })
          .finally(() => {
            this.isLeasesLoading = false
            this.isLeasesRefreshing = false
          })
      }, 500),
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
            this.$toast.error(error.response.data.message)
          })
          .finally(() => {
            this.showConfirmDeleteDialog = false
            this.isConfirmDeleteDialogButtonLoading = false
          })
      },
      onLeaseAdded() {
        this.refreshLeases()
      },
      onLeaseUpdated() {
        this.refreshLeases()
      },
      onPageChange() {
        this.getLeases()
      },
      onSearchFilterChange(filter) {
        this.searchColumn = filter
      },
      viewLease(lease) {
        this.$router.push({ name: 'lease-details', params: { id: lease.id } })
      },
      formatDate(date) {
        const options = { year: 'numeric', month: 'short', day: 'numeric' }
        return new Date(date).toLocaleDateString(undefined, options)
      },
    },
  }
  </script>
  