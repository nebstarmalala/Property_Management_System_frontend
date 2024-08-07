<template>
  <v-row justify="center">
    <v-dialog v-model="showDialog" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="text-h5">{{ isEdit ? 'Edit Tenant' : 'Add Tenant' }}</span>
        </v-card-title>
        <v-form ref="form" method="post" action="/" lazy-validation @submit.prevent="saveTenant">
          <v-card-text>
            <v-container>
              <v-row>
                <v-col cols="12">
                  <v-text-field
                    v-model="form.name"
                    label="Name"
                    placeholder="Enter tenant name"
                    :rules="nameRules"
                    :error="form.errors.has('name')"
                    :error-messages="form.errors.get('name')"
                    outlined
                  ></v-text-field>
                </v-col>
                <v-col cols="12">
                  <v-text-field
                    v-model="form.email"
                    label="Email"
                    placeholder="Enter tenant email"
                    :rules="emailRules"
                    :error="form.errors.has('email')"
                    :error-messages="form.errors.get('email')"
                    outlined
                    type="email"
                  ></v-text-field>
                </v-col>
                <v-col cols="12">
                  <v-text-field
                    v-model="form.phone_number"
                    label="Phone Number"
                    placeholder="Enter phone number"
                    :rules="phoneNumberRules"
                    :error="form.errors.has('phone_number')"
                    :error-messages="form.errors.get('phone_number')"
                    outlined
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
import validationRules from '@/mixins/validationRules'

export default {
  mixins: [validationRules],
  props: {
    showDialog: {
      type: Boolean,
    },
    tenant: {
      type: Object,
      default: () => ({}),
    },
  },
  data() {
    return {
      form: new Form({
        name: '',
        email: '',
        phone_number: '',
      }),
      nameRules: [v => !!v || 'Name is required'],
      emailRules: [
        v => !!v || 'Email is required',
        v => /.+@.+\..+/.test(v) || 'Email must be valid',
      ],
      phoneNumberRules: [v => !!v || 'Phone number is required'],
    }
  },
  computed: {
    isEdit() {
      return !!this.tenant.id
    },
  },
  watch: {
    tenant: {
      handler(newTenant) {
        if (newTenant && newTenant.id) {
          this.form.name = newTenant.user.name
          this.form.email = newTenant.user.email
          this.form.phone_number = newTenant.user.phone_number
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
    saveTenant() {
      const isFormValid = this.$refs.form.validate()
      if (isFormValid) {
        const request = this.isEdit
          ? this.form.put(`users/${this.tenant.id}`)
          : this.form.post(`users/`)

        request
          .then(() => {
            this.$toast.success(`${this.isEdit ? 'Tenant updated' : 'Tenant added'} successfully`)
            this.$emit('close')
            this.$emit('tenant-added')
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
