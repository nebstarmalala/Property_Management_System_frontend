<template>
    <v-row justify="center">
      <v-dialog v-model="showDialog" persistent max-width="600px">
        <v-card>
          <v-card-title>
            <span class="text-h5">{{ isEdit ? 'Edit Repair' : 'Add Repair' }}</span>
          </v-card-title>
          <v-form ref="form" v-model="formValid" lazy-validation @submit.prevent="saveRepair">
            <v-card-text>
              <v-container>
                <v-row>
                  <!-- Category -->
                  <v-col cols="12">
                    <v-text-field
                      v-model="form.category"
                      label="Category"
                      :rules="[v => !!v || 'Category is required']"
                      :error="form.errors.has('category')"
                      :error-messages="form.errors.get('category')"
                      outlined
                    ></v-text-field>
                  </v-col>
                  <!-- Description -->
                  <v-col cols="12">
                    <v-textarea
                      v-model="form.description"
                      label="Description"
                      :rules="[v => !!v || 'Description is required']"
                      :error="form.errors.has('description')"
                      :error-messages="form.errors.get('description')"
                      outlined
                    ></v-textarea>
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
                  <!-- Priority -->
                  <v-col cols="12">
                    <v-select
                      v-model="form.priority"
                      :items="priorityOptions"
                      label="Priority"
                      item-text="text"
                      item-value="value"
                      :error="form.errors.has('priority')"
                      :error-messages="form.errors.get('priority')"
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
      repair: {
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
          lease_id: '',
          category: '',
          description: '',
          status: '',
          priority: '',
        }),
        formValid: false,
        statusOptions: [
          { text: 'Pending', value: 1 },
          { text: 'In Progress', value: 2 },
          { text: 'Completed', value: 3 },
        ],
        priorityOptions: [
          { text: 'Low', value: 1 },
          { text: 'Medium', value: 2 },
          { text: 'High', value: 3 },
        ],
      }
    },
    computed: {
      isEdit() {
        return !!this.repair.id
      },
    },
    watch: {
      repair: {
        handler(newRepair) {
          if (newRepair && newRepair.id) {
            this.form.fill(newRepair)
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
      saveRepair() {
        if (this.$refs.form.validate()) {
          this.form.busy = true; // Set loading state
          const request = this.isEdit ? this.form.put(`repairs/${this.repair.id}`) : this.form.post('repairs');
  
          request
            .then(() => {
              this.$toast.success(`${this.isEdit ? 'Repair updated' : 'Repair added'} successfully`);
              this.$emit('close');
              this.$emit('repair-added');
              this.form.reset();
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
  }
  </script>
  
  <style scoped>
  /* Add any specific styles here */
  </style>
  