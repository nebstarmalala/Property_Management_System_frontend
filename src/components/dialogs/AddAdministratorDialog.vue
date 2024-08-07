<template>
  <v-row justify="center">
    <v-dialog v-model="showDialog" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="text-h5">Add Administrator</span>
        </v-card-title>
        <v-form
          ref="form"
          method="post"
          action="/"
          lazy-validation
          @submit.prevent="saveUser()"
        >
          <v-card-text>
            <v-container>
              <v-row>
                <v-col cols="12">
                  <v-text-field
                    v-model="form.name"
                    label="Full Name"
                    placeholder="John Doe"
                    :rules="nameRules"
                    :error="form.errors.has('name')"
                    :error-messages="form.errors.get('name')"
                    outlined
                  ></v-text-field>
                </v-col>
                <v-col cols="12">
                  <v-text-field
                    v-model="form.phone_number"
                    label="Phone Number"
                    placeholder="0712345678"
                    :rules="phoneNumberRules"
                    :error="form.errors.has('phone_number')"
                    :error-messages="form.errors.get('phone_number')"
                    outlined
                  ></v-text-field>
                </v-col>
                <v-col cols="12">
                  <v-text-field
                    v-model="form.email"
                    label="Email"
                    placeholder="example@domain.com"
                    :rules="emailRules"
                    :error="form.errors.has('email')"
                    :error-messages="form.errors.get('email')"
                    outlined
                  ></v-text-field>
                </v-col>
              </v-row>
            </v-container>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" text @click="closeDialog()">
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
  },
  data: () => ({
    form: new Form({
      name: '',
      email: '',
      phone_number: '',
    }),
    phoneNumberRules: [
      v => !!v || 'Phone number is required',
      v => /^[0-9]{10}$/.test(v) || 'Phone number must be 10 digits',
    ],
    emailRules: [
      v => !!v || 'Email is required',
      v => /.+@.+\..+/.test(v) || 'E-mail must be valid',
    ],
    nameRules: [
      v => !!v || 'Name is required',
    ],
  }),
  methods: {
    closeDialog() {
      this.$emit('close')
      this.form.reset()
      this.form.clear()
    },
    saveUser() {
      const isFormValid = this.$refs.form.validate()
      if (isFormValid) {
        this.form
          .post('admins')
          .then(() => {
            this.$toast.success('Administrator added successfully')
            this.$emit('close')
            this.$emit('user-added')
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
