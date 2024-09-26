<template>
    <v-dialog v-model="dialog" max-width="600px">
      <v-card>
        <v-card-title>
          Update Lease
          <v-spacer></v-spacer>
          <v-btn icon @click="dialog = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-card-title>
        <v-card-text>
          <v-form ref="form" v-model="isFormValid">
            <v-container>
              <v-row>
                <v-col cols="12">
                  <v-text-field
                    label="Lease Term (in months)"
                    v-model="lease.term"
                    :rules="[rules.required, rules.isPositiveNumber]"
                    prepend-icon="mdi-calendar-clock"
                  ></v-text-field>
                </v-col>
  
                <v-col cols="12">
                  <v-menu
                    v-model="startDatePicker"
                    :close-on-content-click="false"
                    transition="scale-transition"
                    offset-y
                    min-width="290px"
                  >
                    <template v-slot:activator="{ on, attrs }">
                      <v-text-field
                        v-model="lease.start_date"
                        label="Start Date"
                        prepend-icon="mdi-calendar"
                        readonly
                        v-bind="attrs"
                        v-on="on"
                        :rules="[rules.required]"
                      ></v-text-field>
                    </template>
                    <v-date-picker v-model="lease.start_date" @input="startDatePicker = false"></v-date-picker>
                  </v-menu>
                </v-col>
  
                <v-col cols="12">
                  <v-menu
                    v-model="endDatePicker"
                    :close-on-content-click="false"
                    transition="scale-transition"
                    offset-y
                    min-width="290px"
                  >
                    <template v-slot:activator="{ on, attrs }">
                      <v-text-field
                        v-model="lease.end_date"
                        label="End Date"
                        prepend-icon="mdi-calendar"
                        readonly
                        v-bind="attrs"
                        v-on="on"
                        :rules="[rules.required]"
                      ></v-text-field>
                    </template>
                    <v-date-picker v-model="lease.end_date" @input="endDatePicker = false"></v-date-picker>
                  </v-menu>
                </v-col>
  
                <v-col cols="12">
                  <v-text-field
                    label="Rent Amount (KES)"
                    v-model="lease.rent_amount"
                    :rules="[rules.required, rules.isPositiveNumber]"
                    prepend-icon="mdi-currency-usd"
                  ></v-text-field>
                </v-col>
  
              </v-row>
            </v-container>
          </v-form>
        </v-card-text>
  
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" @click="updateLease" :disabled="!isFormValid">
            Update
          </v-btn>
          <v-btn text @click="dialog = false">Cancel</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </template>
  
  <script>
  export default {
    data() {
      return {
        dialog: false,
        isFormValid: false,
        startDatePicker: false,
        endDatePicker: false,
        lease: {
          term: '',
          start_date: '',
          end_date: '',
          rent_amount: '',
        },
        rules: {
          required: value => !!value || 'This field is required',
          isPositiveNumber: value => (value > 0) || 'Must be a positive number',
        },
      };
    },
    methods: {
      updateLease() {
        if (this.$refs.form.validate()) {
          // Perform update action here (API call, etc.)
          console.log('Lease Updated:', this.lease);
          this.dialog = false;
          this.resetForm();
        }
      },
      resetForm() {
        this.lease = {
          term: '',
          start_date: '',
          end_date: '',
          rent_amount: '',
        };
        this.$refs.form.resetValidation();
      },
    },
  };
  </script>
  