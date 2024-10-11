<template>
  <v-row justify="center">
    <v-dialog v-model="showDialog" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="text-h5">Add Payment</span>
        </v-card-title>
        <v-form ref="form" v-model="formValid" lazy-validation @submit.prevent="savePayment">
          <v-card-text>
            <v-container>
              <v-row>
                <!-- Payment Amount -->
                <v-col cols="12" md="12">
                  <v-text-field
                    v-model="form.amount"
                    label="Payment Amount"
                    type="number"
                    :rules="[v => !!v || 'Amount is required', v => v > 0 || 'Amount must be positive']"
                    :error="form.errors.has('amount')"
                    :error-messages="form.errors.get('amount')"
                    outlined
                  ></v-text-field>
                </v-col>

                <!-- Payment Date -->
                <v-col cols="12" md="6">
                  <v-menu
                    v-model="datePickerMenu"
                    :close-on-content-click="false"
                    transition="scale-transition"
                    offset-y
                    min-width="auto"
                  >
                    <template v-slot:activator="{ on, attrs }">
                      <v-text-field
                        v-model="form.payment_date"
                        label="Payment Date"
                        readonly
                        v-bind="attrs"
                        v-on="on"
                        :rules="[v => !!v || 'Payment date is required']"
                        :error="form.errors.has('payment_date')"
                        :error-messages="form.errors.get('payment_date')"
                        outlined
                      ></v-text-field>
                    </template>
                    <v-date-picker v-model="form.payment_date" @input="datePickerMenu = false"></v-date-picker>
                  </v-menu>
                </v-col>

                <!-- Payment Method -->
                <v-col cols="12" md="6">
                  <v-select
                    v-model="form.payment_method"
                    :items="paymentMethods"
                    label="Payment Method"
                    item-text="text"
                    item-value="value"
                    :rules="[v => !!v || 'Payment method is required']"
                    :error="form.errors.has('payment_method')"
                    :error-messages="form.errors.get('payment_method')"
                    outlined
                  ></v-select>
                </v-col>

                <!-- Reference -->
                <v-col cols="12" md="12">
                  <v-text-field
                    v-model="form.reference"
                    label="Reference Number"
                    :rules="[v => !!v || 'Reference is required']"
                    :error="form.errors.has('reference')"
                    :error-messages="form.errors.get('reference')"
                    outlined
                  ></v-text-field>
                </v-col>

                <!-- Invoice ID (Hidden) -->
                <v-col cols="12" md="12">
                  <v-text-field
                    v-model="form.invoice_id"
                    label="Invoice ID"
                    :rules="[v => !!v || 'Invoice ID is required']"
                    :error="form.errors.has('invoice_id')"
                    :error-messages="form.errors.get('invoice_id')"
                    outlined
                    readonly
                  ></v-text-field>
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
    invoiceId: {
      type: [String, Number],
      required: true,
    },
  },
  data() {
    return {
      form: new Form({
        amount: '',
        payment_date: '',
        payment_method: '',
        reference: '',
        invoice_id: this.invoiceId,
      }),
      formValid: false,
      paymentMethods: [
        { text: 'Mpesa', value: 1 },
        { text: 'Credit Card', value: 2 },
        { text: 'Bank Transfer', value: 3 },
        { text: 'Cash', value: 4 },
      ],
      datePickerMenu: false,
    };
  },
  watch: {
    invoiceId: {
      handler(newInvoiceId) {
        this.form.invoice_id = newInvoiceId; // Update invoice ID when prop changes
      },
      immediate: true,
    },
  },
  methods: {
    closeDialog() {
      this.$emit('close');
      this.form.reset();
    },
    savePayment() {
      if (this.$refs.form.validate()) {
        this.form
          .post('/payments')
          .then(() => {
            this.$toast.success('Payment added successfully');
            this.$emit('payment-added');
            this.closeDialog();
          })
          .catch(error => {
            const errorMessage = error.response?.data?.message || 'An error occurred';
            this.$toast.error(errorMessage);
          })
          .finally(() => {
            this.form.busy = false;
          });
      }
    },
  },
};
</script>
