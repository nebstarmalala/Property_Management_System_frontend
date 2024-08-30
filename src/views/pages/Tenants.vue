<template>
    <div class="tenants">
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
        <h1 class="text-center">Tenants</h1>
        <p class="text-center">Manage your tenants efficiently.</p>
        <v-row>
          <v-spacer></v-spacer>
          <v-col class="text-right mt-3 mr-8" cols="6">
            <v-btn color="primary" @click="showAddTenantDialog = true">
              <v-icon left size="22">{{ icons.mdiPlus }}</v-icon>
              Add Tenant
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
          :items="tenants"
          :loading="isTenantsLoading"
          :options.sync="options"
          :footer-props="footerProps"
          item-key="id"
        >
          <template v-slot:item.created_at="{ item }">
            {{ formatDate(item.created_at) }}
          </template>
          <template v-slot:item.user="{ item }">
            {{ item.user.name }}
          </template>
          <template v-slot:item.action="{ item }">
            <v-btn class="ma-1" text icon @click="editTenant(item)">
              <v-icon>{{ icons.mdiPencil }}</v-icon>
            </v-btn>
            <v-btn class="ma-1" text icon @click="showDeleteTenantDialog(item.id)">
              <v-icon>{{ icons.mdiDelete }}</v-icon>
            </v-btn>
            <v-tooltip bottom>
              <template v-slot:activator="{ on, attrs }">
                <v-btn ma-3 v-bind="attrs" text icon @click="viewTenant(item)" v-on="on">
                  <v-icon v-bind="attrs" v-on="on">{{ icons.mdiEyeOutline }}</v-icon>
                </v-btn>
              </template>
              <span>View Tenant</span>
            </v-tooltip>
          </template>
        </v-data-table>
      </v-card>
      <add-tenant-dialog
        :show-dialog="showAddTenantDialog"
        :tenant="selectedTenant"
        @close="
          showAddTenantDialog = false
          selectedTenant = {}
        "
        @tenant-added="getTenants"
      />
      <confirm-dialog
        :show-dialog="showConfirmDeleteDialog"
        :is-agree-button-loading="isConfirmDeleteDialogButtonLoading"
        :agree-text="'Delete'"
        @cancel="
          showConfirmDeleteDialog = false
          toDelete = null
        "
        @agree="deleteTenant"
      />
    </div>
  </template>
  
  <script>
  import axios from 'axios'
  import _ from 'lodash'
  import { mdiEyeOutline, mdiArrowLeft, mdiPencil, mdiDelete, mdiPlus } from '@mdi/js'
  import BreadCrumb from '@/components/partials/BreadCrumb.vue'
  import SearchInput from '@/components/partials/SearchInput.vue'
  import AddTenantDialog from '@/components/dialogs/AddTenantDialog.vue'
  import ConfirmDialog from '@/components/dialogs/ConfirmDialog.vue'
  
  export default {
    components: {
      breadcrumb: BreadCrumb,
      SearchInput,
      AddTenantDialog,
      ConfirmDialog,
    },
    data() {
      return {
        showAddTenantDialog: false,
        showConfirmDeleteDialog: false,
        isTenantsLoading: true,
        isTenantsRefreshing: false,
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
          { text: 'Tenant Number', value: 'tenant_number' },
          { text: 'Name', value: 'user.name' },
          { text: 'Email', value: 'user.email' },
          { text: 'Phone Number', value: 'user.phone_number' },
          { text: 'Created At', value: 'created_at' },
          { text: 'Action', value: 'action', sortable: false },
        ],
        searchColumn: 'tenant_number',
        searchFilters: [
          { text: 'Tenant Number', value: 'tenant_number' },
          { text: 'Tenant Name', value: 'user.name' },
          { text: 'Phone Number', value: 'user.phone_number' },
          { text: 'Email', value: 'user.email' },
        ],
        tenants: [],
        selected: [],
        toDelete: null,
        tenantsUrl: 'tenants',
        breadcrumbs: [
          { text: 'Dashboard', disabled: false, to: { name: 'dashboard' } },
          { text: 'Tenants', disabled: true },
        ],
        icons: {
          mdiEyeOutline,
          mdiArrowLeft,
          mdiPencil,
          mdiDelete,
          mdiPlus,
        },
        selectedTenant: {},
      }
    },
    computed: {
      refreshTable() {
        return `${this.tenantsUrl}|${this.search}|${this.sortBy}|${this.sortDesc}`
      },
    },
    watch: {
      options: {
        handler() {
          this.getTenants()
        },
        deep: true,
      },
      refreshTable() {
        this.getTenants()
      },
      searchColumn() {
        if (this.search !== '') {
          this.getTenants()
        }
      },
    },
    mounted() {
      this.getTenants()
    },
    methods: {
      getTenants: _.debounce(function() {
        this.isTenantsLoading = true
        const { sortBy, sortDesc, page, itemsPerPage } = this.options
        const sortOrder = sortDesc[0] ? 'desc' : 'asc'
        axios
          .get(
            `${this.tenantsUrl}?paginate=true&sortBy=${sortBy[0]}&sortDirection=${sortOrder}&search=${this.search}&searchColumn=${this.searchColumn}&page=${page}&perPage=${itemsPerPage}`,
          )
          .then(response => {
            this.tenants = response.data.data
            this.pagination.page = response.data.current_page
            this.pagination.pageCount = response.data.last_page
            this.pagination.itemsPerPage = response.data.per_page
            this.pagination.totalItems = response.data.total
          })
          .catch(error => {
            this.$toast.error(error.response.data.message)
          })
          .finally(() => {
            this.isTenantsLoading = false
            this.isTenantsRefreshing = false
          })
      }, 500),
      refreshTenants() {
        this.isTenantsRefreshing = true
        this.getTenants()
      },
      editTenant(tenant) {
        this.selectedTenant = tenant
        this.showAddTenantDialog = true
      },
      showDeleteTenantDialog(id) {
        this.showConfirmDeleteDialog = true
        this.toDelete = id
      },
      deleteTenant() {
        this.isConfirmDeleteDialogButtonLoading = true
        axios
          .delete(`tenants/${this.toDelete}`)
          .then(() => {
            this.refreshTenants()
            this.$toast.success('Tenant deleted successfully')
          })
          .catch(error => {
            this.$toast.error(error.response.data.message)
          })
          .finally(() => {
            this.showConfirmDeleteDialog = false
            this.isConfirmDeleteDialogButtonLoading = false
          })
      },
      onTenantAdded() {
        this.refreshTenants()
      },
      onTenantUpdated() {
        this.refreshTenants()
      },
      onPageChange() {
        this.getTenants()
      },
      onSearchFilterChange(filter) {
        this.searchColumn = filter
      },
      viewTenant(tenant) {
        this.$router.push({ name: 'tenant-details', params: { id: tenant.id } })
      },
      formatDate(date) {
        const options = { year: 'numeric', month: 'short', day: 'numeric' }
        return new Date(date).toLocaleDateString(undefined, options)
      },
    },
  }
  </script>
  