<template>
  <v-row justify="center">
    <v-dialog v-model="showDialog" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="text-h5">{{ isEdit ? 'Edit Invoice' : 'Add Invoice' }}</span>
        </v-card-title>
        <v-form ref="form" v-model="formValid" lazy-validation @submit.prevent="saveInvoice">
          <v-card-text>
            <v-container>
              <v-row>
                <!-- Invoice Number -->
                <v-col cols="12" md="6">
                  <v-text-field
                    v-model="form.invoice_number"
                    label="Invoice Number"
                    :rules="[v => !!v || 'Invoice number is required']"
                    :error="form.errors.has('invoice_number')"
                    :error-messages="form.errors.get('invoice_number')"
                    outlined
                  ></v-text-field>
                </v-col>
                <!-- Reference -->
                <v-col cols="12" md="6">
                  <v-text-field
                    v-model="form.ref"
                    label="Reference"
                    :error="form.errors.has('ref')"
                    :error-messages="form.errors.get('ref')"
                    outlined
                  ></v-text-field>
                </v-col>
                <!-- Amount -->
                <v-col cols="12" md="4">
                  <v-text-field
                    v-model="form.amount"
                    label="Amount"
                    type="number"
                    :rules="[v => !!v || 'Amount is required']"
                    :error="form.errors.has('amount')"
                    :error-messages="form.errors.get('amount')"
                    outlined
                  ></v-text-field>
                </v-col>
                <!-- Tax -->
                <v-col cols="12" md="4">
                  <v-text-field
                    v-model="form.tax"
                    label="Tax"
                    type="number"
                    :error="form.errors.has('tax')"
                    :error-messages="form.errors.get('tax')"
                    outlined
                  ></v-text-field>
                </v-col>
                <!-- Discount -->
                <v-col cols="12" md="4">
                  <v-text-field
                    v-model="form.discount"
                    label="Discount"
                    type="number"
                    :error="form.errors.has('discount')"
                    :error-messages="form.errors.get('discount')"
                    outlined
                  ></v-text-field>
                </v-col>
                <!-- Billing Date -->
                <v-col cols="12" md="6">
                  <v-menu
                    v-model="billingDateMenu"
                    :close-on-content-click="false"
                    transition="scale-transition"
                    offset-y
                    min-width="auto"
                  >
                    <template v-slot:activator="{ on, attrs }">
                      <v-text-field
                        v-model="form.billing_date"
                        label="Billing Date"
                        readonly
                        v-bind="attrs"
                        v-on="on"
                        :rules="[v => !!v || 'Billing date is required']"
                        :error="form.errors.has('billing_date')"
                        :error-messages="form.errors.get('billing_date')"
                        outlined
                      ></v-text-field>
                    </template>
                    <v-date-picker
                      v-model="form.billing_date"
                      @input="billingDateMenu = false"
                    ></v-date-picker>
                  </v-menu>
                </v-col>
                <!-- Due Date -->
                <v-col cols="12" md="6">
                  <v-menu
                    v-model="dueDateMenu"
                    :close-on-content-click="false"
                    transition="scale-transition"
                    offset-y
                    min-width="auto"
                  >
                    <template v-slot:activator="{ on, attrs }">
                      <v-text-field
                        v-model="form.due_date"
                        label="Due Date"
                        readonly
                        v-bind="attrs"
                        v-on="on"
                        :rules="[v => !!v || 'Due date is required']"
                        :error="form.errors.has('due_date')"
                        :error-messages="form.errors.get('due_date')"
                        outlined
                      ></v-text-field>
                    </template>
                    <v-date-picker
                      v-model="form.due_date"
                      @input="dueDateMenu = false"
                    ></v-date-picker>
                  </v-menu>
                </v-col>
                <!-- Status -->
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
import Form from 'vform'

export default {
  props: {
    showDialog: {
      type: Boolean,
      required: true,
    },
    invoice: {
      type: Object,
      default: () => ({}),
    },
    leaseId: {
      type: [String, Number],
      default: null,
    },
  },
  data() {
    return {
      form: new Form({
        invoice_number: '',
        lease_id: '',
        amount: '',
        tax: '',
        discount: '',
        ref: '',
        due_date: '',
        billing_date: '',
        status: '',
      }),
      formValid: false,
      statusOptions: [
        { text: 'Paid', value: 1 },
        { text: 'Pending', value: 2 },
        { text: 'Overdue', value: 3 },
      ],
      billingDateMenu: false,
      dueDateMenu: false,
    }
  },
  computed: {
    isEdit() {
      return !!this.invoice.id
    },
  },
  watch: {
    invoice: {
      handler(newInvoice) {
        if (newInvoice && newInvoice.id) {
          this.form.fill(newInvoice)
        }
      },
      immediate: true,
    },
    leaseId: {
      handler(newLeaseId) {
        if (newLeaseId) {
          this.form.lease_id = newLeaseId
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
    saveInvoice() {
      if (this.$refs.form.validate()) {
        const request = this.isEdit
          ? this.form.put(`invoices/${this.invoice.id}`)
          : this.form.post('invoices')

        request
          .then(() => {
            this.$toast.success(`${this.isEdit ? 'Invoice updated' : 'Invoice added'} successfully`)
            this.$emit('close')
            this.$emit('invoice-saved')
            this.form.reset()
          })
          .catch(error => {
            this.$toast.error(error.response.data.message)
          })
      }
    },
  },
}
</script>