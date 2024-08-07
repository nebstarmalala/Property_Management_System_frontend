<template>
  <div class="properties">
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
      <h1 class="text-center">Properties</h1>
      <p class="text-center">Manage your properties efficiently.</p>
      <v-row>
        <v-spacer></v-spacer>
        <v-col class="text-right mt-3 mr-8" cols="6">
          <v-btn color="primary" @click="showAddPropertyDialog = true">
            <v-icon left size="22">{{ icons.mdiPlus }}</v-icon>
            Add Property
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
        :items="properties"
        :loading="isPropertiesLoading"
        :options.sync="options"
        :footer-props="footerProps"
        item-key="id"
      >
        <template v-slot:item.description="{ item }">
          {{ truncateDescription(item.description) }}
        </template>
        <template v-slot:item.created_at="{ item }">
          {{ formatDate(item.created_at) }}
        </template>
        <template v-slot:item.action="{ item }">
          <v-btn class="ma-1" text icon @click="editProperty(item)">
            <v-icon>{{ icons.mdiPencil }}</v-icon>
          </v-btn>
          <v-btn class="ma-1" text icon @click="showDeletePropertyDialog(item.id)">
            <v-icon>{{ icons.mdiDelete }}</v-icon>
          </v-btn>
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-btn ma-3 v-bind="attrs" text icon @click="viewProperty(item)" v-on="on">
                <v-icon v-bind="attrs" v-on="on">{{ icons.mdiEyeOutline }}</v-icon>
              </v-btn>
            </template>
            <span>View Property</span>
          </v-tooltip>
        </template>
      </v-data-table>
    </v-card>
    <add-property-dialog
      :show-dialog="showAddPropertyDialog"
      :property="selectedProperty"
      @close="
        showAddPropertyDialog = false
        selectedProperty = {}
      "
      @property-added="getProperties"
    />
    <confirm-dialog
      :show-dialog="showConfirmDeleteDialog"
      :is-agree-button-loading="isConfirmDeleteDialogButtonLoading"
      :agree-text="'Delete'"
      @cancel="
        showConfirmDeleteDialog = false
        toDelete = null
      "
      @agree="deleteProperty"
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
import AddPropertyDialog from '@/components/dialogs/AddPropertyDialog.vue'
import ConfirmDialog from '@/components/dialogs/ConfirmDialog.vue'

export default {
  components: {
    breadcrumb: BreadCrumb,
    SearchInput,
    RefreshButton,
    AddPropertyDialog,
    ConfirmDialog,
  },
  data() {
    return {
      showAddPropertyDialog: false,
      showConfirmDeleteDialog: false,
      isPropertiesLoading: true,
      isPropertiesRefreshing: false,
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
        { text: 'Name', value: 'name' },
        { text: 'Location', value: 'location' },
        { text: 'Description', value: 'description' },
        { text: 'Created At', value: 'created_at' },
        { text: 'Action', value: 'action', sortable: false },
      ],
      searchColumn: 'name',
      searchFilters: [
        { text: 'Name', value: 'name' },
        { text: 'Location', value: 'location' },
      ],
      properties: [],
      selected: [],
      toDelete: null,
      propertiesUrl: 'properties',
      breadcrumbs: [
        { text: 'Dashboard', disabled: false, to: { name: 'dashboard' } },
        { text: 'Properties', disabled: true },
      ],
      icons: {
        mdiEyeOutline,
        mdiArrowLeft,
        mdiPencil,
        mdiDelete,
        mdiPlus,
      },
      selectedProperty: {},
    }
  },
  computed: {
    refreshTable() {
      return `${this.propertiesUrl}|${this.search}|${this.sortBy}|${this.sortDesc}`
    },
  },
  watch: {
    options: {
      handler() {
        this.getProperties()
      },
      deep: true,
    },
    refreshTable() {
      this.getProperties()
    },
    searchColumn() {
      if (this.search !== '') {
        this.getProperties()
      }
    },
  },
  mounted() {
    this.getProperties()
  },
  methods: {
    getProperties: _.debounce(function() {
      this.isPropertiesLoading = true
      const { sortBy, sortDesc, page, itemsPerPage } = this.options
      const sortOrder = sortDesc[0] ? 'desc' : 'asc'
      axios
        .get(
          `${this.propertiesUrl}?paginate=true&sortBy=${sortBy[0]}&sortDirection=${sortOrder}&search=${this.search}&searchColumn=${this.searchColumn}&page=${page}&perPage=${itemsPerPage}`,
        )
        .then(response => {
          this.properties = response.data.data
          this.pagination.page = response.data.current_page
          this.pagination.pageCount = response.data.last_page
          this.pagination.itemsPerPage = response.data.per_page
          this.pagination.totalItems = response.data.total
        })
        .catch(error => {
          this.$toast.error(error.response.data.message)
        })
        .finally(() => {
          this.isPropertiesLoading = false
          this.isPropertiesRefreshing = false
        })
    }, 500),
    refreshProperties() {
      this.isPropertiesRefreshing = true
      this.getProperties()
    },
    editProperty(property) {
      this.selectedProperty = property
      this.showAddPropertyDialog = true
    },
    showDeletePropertyDialog(id) {
      this.showConfirmDeleteDialog = true
      this.toDelete = id
    },
    deleteSelectedProperties() {
      this.isConfirmBulkDeleteDialogButtonLoading = true
      const selectedIds = this.selected.map(property => property.id)
      axios
        .delete('bulk-delete/properties', { data: { ids: selectedIds } })
        .then(() => {
          this.selected = []
          this.getProperties()
        })
        .catch(error => {
          this.$toast.error(error.response.data.message)
        })
        .finally(() => {
          this.showConfirmBulkDeleteDialog = false
          this.isConfirmBulkDeleteDialogButtonLoading = false
        })
    },
    deleteProperty() {
      this.isConfirmDeleteDialogButtonLoading = true
      axios
        .delete(`properties/${this.toDelete}`)
        .then(() => {
          this.refreshProperties()
          this.$toast.success('Property deleted successfully')
        })
        .catch(error => {
          this.$toast.error(error.response.data.message)
        })
        .finally(() => {
          this.showConfirmDeleteDialog = false
          this.isConfirmDeleteDialogButtonLoading = false
        })
    },
    onPropertyAdded() {
      this.refreshProperties()
    },
    onPropertyUpdated() {
      this.refreshProperties()
    },
    onPageChange() {
      this.getProperties()
    },
    onSearchFilterChange(filter) {
      this.searchColumn = filter
    },
    viewProperty(property) {
      this.$router.push({ name: 'property-details', params: { id: property.id } })
    },
    truncateDescription(description) {
      return description.length > 100 ? description.slice(0, 100) + '...' : description
    },
    formatDate(date) {
      const options = { year: 'numeric', month: 'short', day: 'numeric' }
      return new Date(date).toLocaleDateString(undefined, options)
    },
  },
}
</script>

<style lang="scss" scoped>
.app-title {
  font-size: 1.25rem;
  font-weight: 700;
  font-stretch: normal;
  font-style: normal;
  line-height: normal;
  letter-spacing: 0.3px;
}

.app-logo {
  transition: all 0.18s ease-in-out;
  .v-navigation-drawer--mini-variant & {
    transform: translateX(-4px);
  }
}

@include theme(app-navigation-menu) using ($material) {
  background-color: map-deep-get($material, 'background');
}

.app-navigation-menu {
  .v-list-item {
    &.vertical-nav-menu-link {
      ::v-deep .v-list-item__icon {
        .v-icon {
          transition: none !important;
        }
      }
    }
  }
}

.settings-menu {
  position: absolute;
  bottom: 20px;
  left: 50%;
  width: 100%;
  transform: translateX(-50%);
}
</style>
