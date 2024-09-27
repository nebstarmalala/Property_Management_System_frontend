<template>
    <v-card :min-height="210">
      <v-card-title class="mb-0 pb-1">
        Lease Details
        <v-spacer></v-spacer>
      </v-card-title>
      <v-card-text>
        <v-container class="mt-0">
          <v-row>
            <v-col
              v-if="!lease && !isLoading"
              cols="12"
            >
              <h3 class="font-weight-light">
                Lease not found
              </h3>
            </v-col>
            <v-col
              v-for="detail in details"
              :key="detail.label"
              cols="12"
              md="3"
            >
              <details-text-shimmer v-show="isLoading" />
              <div
                v-if="lease"
                v-show="!isLoading"
              >
                <h4 class="font-weight-light">
                  {{ detail.label }}
                </h4>
                <h4 class="font-weight-medium">
                  <span v-if="isLoading">
                    <details-text-shimmer />
                  </span>
                  <span v-else>
                    <span v-if="detail.label === 'Lease Start Date' || detail.label === 'Lease End Date'">{{ detail.value | formatDate }}</span>
                    <span v-else-if="detail.label === 'Total Rent'">Ksh {{ detail.value | formatCurrency }}</span>
                    <span v-else>{{ detail.value }}</span>
                  </span>
                </h4>
              </div>
            </v-col>
          </v-row>
        </v-container>
      </v-card-text>
    </v-card>
  </template>
  
  <script>
  import DetailsTextShimmer from '@/components/partials/shimmers/DetailsTextShimmer.vue'
  
  export default {
    components: {
      'details-text-shimmer': DetailsTextShimmer,
    },
    props: {
      lease: {
        type: Object,
        default: null,
      },
      isLoading: {
        type: Boolean,
        default: true,
      },
    },
    computed: {
      details() {
        const details = [
          { label: 'Lease Number', value: this.lease ? this.lease.lease_number : 'N/A' },
          { label: 'Tenant Name', value: this.lease ? this.lease.tenant.name : 'N/A' },
          { label: 'Property Name', value: this.lease ? this.lease.property.name : 'N/A' },
          { label: 'Unit Number', value: this.lease ? this.lease.unit.number : 'N/A' },
          { label: 'Lease Start Date', value: this.lease ? this.lease.start_date : 'N/A' },
          { label: 'Lease End Date', value: this.lease ? this.lease.end_date : 'N/A' },
          { label: 'Total Rent', value: this.lease ? this.lease.total_rent : 'N/A' },
          { label: 'Payment Status', value: this.lease ? this.lease.payment_status : 'N/A' },
        ];
  
        return details;
      },
    },
  };
  </script>
  