<template>
  <div class="units">
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
      <h1 class="text-center">Units</h1>
      <p class="text-center">Manage your units efficiently.</p>
      <v-row>
        <v-spacer></v-spacer>
        <v-col class="text-right mt-3 mr-8" cols="6">
          <v-btn color="primary" @click="showAddUnitDialog = true">
            <v-icon left size="22">{{ icons.mdiPlus }}</v-icon>
            Add Unit
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
        :items="units"
        :loading="isUnitsLoading"
        :options.sync="options"
        :footer-props="footerProps"
        item-key="id"
      >
        <template v-slot:item.created_at="{ item }">
          {{ formatDate(item.created_at) }}
        </template>
        <template v-slot:item.status="{ item }">
          {{ getStatusText(item.status) }}
        </template>
        <template v-slot:item.action="{ item }">
          <v-btn class="ma-1" text icon @click="editUnit(item)">
            <v-icon>{{ icons.mdiPencil }}</v-icon>
          </v-btn>
          <v-btn class="ma-1" text icon @click="showDeleteUnitDialog(item.id)">
            <v-icon>{{ icons.mdiDelete }}</v-icon>
          </v-btn>
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-btn ma-3 v-bind="attrs" text icon @click="viewUnit(item)" v-on="on">
                <v-icon v-bind="attrs" v-on="on">{{ icons.mdiEyeOutline }}</v-icon>
              </v-btn>
            </template>
            <span>View Unit</span>
          </v-tooltip>
        </template>
      </v-data-table>
    </v-card>
    <add-unit-dialog
      :show-dialog="showAddUnitDialog"
      :unit="selectedUnit"
      :property-id="propertyId"
      @close="
        showAddUnitDialog = false
        selectedUnit = {}
      "
      @unit-added="getUnits"
    />
    <confirm-dialog
      :show-dialog="showConfirmDeleteDialog"
      :is-agree-button-loading="isConfirmDeleteDialogButtonLoading"
      :agree-text="'Delete'"
      @cancel="
        showConfirmDeleteDialog = false
        toDelete = null
      "
      @agree="deleteUnit"
    />
  </div>
</template>

<script>
import axios from 'axios'
import _ from 'lodash'
import { mdiEyeOutline, mdiArrowLeft, mdiPencil, mdiDelete, mdiPlus } from '@mdi/js'
import BreadCrumb from '@/components/partials/BreadCrumb.vue'
import SearchInput from '@/components/partials/SearchInput.vue'
import RefreshButton from '@/components/partials/RefreshButton.vue'
import AddUnitDialog from '@/components/dialogs/AddUnitDialog.vue'
import ConfirmDialog from '@/components/dialogs/ConfirmDialog.vue'

export default {
  components: {
    breadcrumb: BreadCrumb,
    SearchInput,
    RefreshButton,
    AddUnitDialog,
    ConfirmDialog,
  },
  data() {
    return {
      showAddUnitDialog: false,
      showConfirmDeleteDialog: false,
      isUnitsLoading: true,
      isUnitsRefreshing: false,
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
        { text: 'Unit Number', value: 'unit_number' },
        { text: 'Size', value: 'size' },
        { text: 'Bedrooms', value: 'bedrooms' },
        { text: 'Rent', value: 'rent' },
        { text: 'Status', value: 'status' },
        { text: 'Created At', value: 'created_at' },
        { text: 'Action', value: 'action', sortable: false },
      ],
      searchColumn: 'unit_number',
      searchFilters: [
        { text: 'Unit Number', value: 'unit_number' },
        { text: 'Size', value: 'size' },
      ],
      units: [],
      selected: [],
      toDelete: null,
      unitsUrl: 'units',
      breadcrumbs: [
        { text: 'Dashboard', disabled: false, to: { name: 'dashboard' } },
        { text: 'Properties', disabled: false, to: { name: 'properties' } },
        { text: 'Units', disabled: true },
      ],
      icons: {
        mdiEyeOutline,
        mdiArrowLeft,
        mdiPencil,
        mdiDelete,
        mdiPlus,
      },
      selectedUnit: {},
      propertyId: null,
    }
  },
  computed: {
    refreshTable() {
      return `${this.unitsUrl}|${this.search}|${this.sortBy}|${this.sortDesc}`
    },
  },
  watch: {
    options: {
      handler() {
        this.getUnits()
      },
      deep: true,
    },
    refreshTable() {
      this.getUnits()
    },
    searchColumn() {
      if (this.search !== '') {
        this.getUnits()
      }
    },
  },
  mounted() {
    this.propertyId = this.$route.params.id
    this.getUnits()
  },
  methods: {
    getUnits: _.debounce(function() {
      this.isUnitsLoading = true
      const { sortBy, sortDesc, page, itemsPerPage } = this.options
      const sortOrder = sortDesc[0] ? 'desc' : 'asc'
      axios
        .get(
          `${this.unitsUrl}?paginate=true&sortBy=${sortBy[0]}&sortDirection=${sortOrder}&search=${this.search}&searchColumn=${this.searchColumn}&page=${page}&perPage=${itemsPerPage}&property_id=${this.propertyId}`,
        )
        .then(response => {
          this.units = response.data.data
          this.pagination.page = response.data.current_page
          this.pagination.pageCount = response.data.last_page
          this.pagination.itemsPerPage = response.data.per_page
          this.pagination.totalItems = response.data.total
        })
        .catch(error => {
          this.$toast.error(error.response.data.message)
        })
        .finally(() => {
          this.isUnitsLoading = false
          this.isUnitsRefreshing = false
        })
    }, 500),
    refreshUnits() {
      this.isUnitsRefreshing = true
      this.getUnits()
    },
    editUnit(unit) {
      this.selectedUnit = unit
      this.showAddUnitDialog = true
    },
    showDeleteUnitDialog(id) {
      this.showConfirmDeleteDialog = true
      this.toDelete = id
    },
    deleteSelectedUnits() {
      this.isConfirmBulkDeleteDialogButtonLoading = true
      const selectedIds = this.selected.map(unit => unit.id)
      axios
        .delete('bulk-delete/units', { data: { ids: selectedIds } })
        .then(() => {
          this.selected = []
          this.getUnits()
        })
        .catch(error => {
          this.$toast.error(error.response.data.message)
        })
        .finally(() => {
          this.showConfirmBulkDeleteDialog = false
          this.isConfirmBulkDeleteDialogButtonLoading = false
        })
    },
    deleteUnit() {
      this.isConfirmDeleteDialogButtonLoading = true
      axios
        .delete(`units/${this.toDelete}`)
        .then(() => {
          this.refreshUnits()
          this.$toast.success('Unit deleted successfully')
        })
        .catch(error => {
          this.$toast.error(error.response.data.message)
        })
        .finally(() => {
          this.showConfirmDeleteDialog = false
          this.isConfirmDeleteDialogButtonLoading = false
        })
    },
    onUnitAdded() {
      this.refreshUnits()
    },
    onUnitUpdated() {
      this.refreshUnits()
    },
    onPageChange() {
      this.getUnits()
    },
    onSearchFilterChange(filter) {
      this.searchColumn = filter
    },
    viewUnit(unit) {
      this.$router.push({ name: 'unit-details', params: { id: unit.id } })
    },
    formatDate(date) {
      const options = { year: 'numeric', month: 'short', day: 'numeric' }
      return new Date(date).toLocaleDateString(undefined, options)
    },
    getStatusText(status) {
      if (status === 1) return 'Available'
      if (status === 2) return 'Leased'
      if (status === 3) return 'Reserved'
      return ''
    }
  },
}
</script>
