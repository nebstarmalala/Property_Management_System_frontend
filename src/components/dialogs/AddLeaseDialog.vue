<template>
  <v-row justify="center">
    <v-dialog v-model="showDialog" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="text-h5">{{ isEdit ? 'Edit Lease' : 'Add Lease' }}</span>
        </v-card-title>
        <v-form ref="form" method="post" action="/" lazy-validation @submit.prevent="saveLease">
          <v-card-text>
            <v-container>
              <v-row>
                <v-col cols="12">
                  <v-select
                    v-model="form.unit_id"
                    :items="units"
                    label="Unit"
                    item-text="unit_number"
                    item-value="id"
                    :rules="[v => !!v || 'Unit is required']"
                    :error="form.errors.has('unit_id')"
                    :error-messages="form.errors.get('unit_id')"
                    outlined
                  ></v-select>
                </v-col>
                <v-col cols="12">
                  <v-select
                    v-model="form.tenant_id"
                    :items="tenants"
                    label="Tenant"
                    item-text="user.name"
                    item-value="id"
                    :rules="[v => !!v || 'Tenant is required']"
                    :error="form.errors.has('tenant_id')"
                    :error-messages="form.errors.get('tenant_id')"
                    outlined
                  ></v-select>
                </v-col>
                <v-col cols="12">
                  <v-text-field
                    v-model="form.start_date"
                    label="Start Date"
                    placeholder="Enter start date"
                    :rules="startDateRules"
                    :error="form.errors.has('start_date')"
                    :error-messages="form.errors.get('start_date')"
                    outlined
                    type="date"
                  ></v-text-field>
                </v-col>
                <v-col cols="12">
                  <v-text-field
                    v-model="form.end_date"
                    label="End Date"
                    placeholder="Enter end date"
                    :rules="endDateRules"
                    :error="form.errors.has('end_date')"
                    :error-messages="form.errors.get('end_date')"
                    outlined
                    type="date"
                  ></v-text-field>
                </v-col>
                <v-col cols="12">
                  <v-text-field
                    v-model="form.security_deposit"
                    label="Security Deposit"
                    placeholder="Enter security deposit amount"
                    :rules="securityDepositRules"
                    :error="form.errors.has('security_deposit')"
                    :error-messages="form.errors.get('security_deposit')"
                    outlined
                    type="number"
                  ></v-text-field>
                </v-col>
                <v-col cols="12">
                  <v-select
                    v-model="form.status"
                    :items="statusOptions"
                    label="Status"
                    item-text="text"
                    item-value="value"
                    :rules="[v => !!v || 'Status is required']"
                    :error="form.errors.has('status')"
                    :error-messages="form.errors.get('status')"
                    outlined
                  ></v-select>
                </v-col>
              </v-row>
            </v-container>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" text @click="closeDialog">
              Close
            </v-btn>
            <v-btn color="blue darken-1" text type="submit" :loading="form.busy">
              Save
            </v-btn>
          </v-card-actions>
        </v-form>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
import axios from 'axios'
import Form from 'vform'
import validationRules from '@/mixins/validationRules'

export default {
  mixins: [validationRules],
  props: {
    showDialog: {
      type: Boolean,
      required: true,
    },
    lease: {
      type: Object,
      default: () => ({}),
    },
  },
  data() {
    return {
      form: new Form({
        unit_id: '',
        tenant_id: '',
        start_date: '',
        end_date: '',
        security_deposit: '',
        status: '',
      }),
      startDateRules: [v => !!v || 'Start date is required'],
      endDateRules: [v => !!v || 'End date is required'],
      securityDepositRules: [
        v => !!v || 'Security deposit is required',
        v => /^\d+(\.\d{1,2})?$/.test(v) || 'Security deposit must be a valid number',
      ],
      statusOptions: [
        { text: 'Active', value: 1 },
        { text: 'Expired', value: 2 },
        { text: 'Terminated', value: 3 },
      ],
      units: [],
      tenants: [],
    }
  },
  computed: {
    isEdit() {
      return !!this.lease.id
    },
  },
  watch: {
    lease: {
      handler(newLease) {
        if (newLease && newLease.id) {
          this.form.unit_id = newLease.unit_id
          this.form.tenant_id = newLease.tenant_id
          this.form.start_date = newLease.start_date
          this.form.end_date = newLease.end_date
          this.form.security_deposit = newLease.security_deposit
          this.form.status = newLease.status
        }
      },
      immediate: true,
    },
  },
  methods: {
    closeDialog() {
      this.$emit('close')
      this.form.reset()
    },
    saveLease() {
      this.$refs.form.validate().then(success => {
        if (success) {
          const request = this.isEdit ? this.form.put(`leases/${this.lease.id}`) : this.form.post(`leases/`)

          request
            .then(() => {
              this.$toast.success(`${this.isEdit ? 'Lease updated' : 'Lease added'} successfully`)
              this.$emit('close')
              this.$emit('lease-added')
              this.form.reset()
            })
            .catch(error => {
              this.$toast.error(error.response.data.message)
            })
        }
      })
    },
    fetchUnits() {
      axios
        .get('units')
        .then(response => {
          this.units = response.data
        })
        .catch(error => {
          console.error('Error fetching units:', error)
        })
    },
    fetchTenants() {
      axios
        .get('tenants')
        .then(response => {
          this.tenants = response.data
        })
        .catch(error => {
          console.error('Error fetching tenants:', error)
        })
    },
  },
  created() {
    this.fetchUnits()
    this.fetchTenants()
  },
}
</script>
