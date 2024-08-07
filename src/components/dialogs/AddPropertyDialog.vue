<template>
    <v-row justify="center">
      <v-dialog v-model="showDialog" persistent max-width="600px">
        <v-card>
          <v-card-title>
            <span class="text-h5">{{ isEdit ? 'Edit Property' : 'Add Property' }}</span>
          </v-card-title>
          <v-form
            ref="form"
            method="post"
            action="/"
            lazy-validation
            @submit.prevent="saveProperty"
          >
            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12">
                    <v-text-field
                      v-model="form.name"
                      label="Property Name"
                      placeholder="Enter property name"
                      :rules="nameRules"
                      :error="form.errors.has('name')"
                      :error-messages="form.errors.get('name')"
                      outlined
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12">
                    <v-text-field
                      v-model="form.location"
                      label="location"
                      placeholder="Enter property location"
                      :rules="addressRules"
                      :error="form.errors.has('location')"
                      :error-messages="form.errors.get('location')"
                      outlined
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12">
                    <v-textarea
                      v-model="form.description"
                      label="Description"
                      placeholder="Enter property description"
                      :rules="descriptionRules"
                      :error="form.errors.has('description')"
                      :error-messages="form.errors.get('description')"
                      outlined
                      rows="4"
                      auto-grow
                    ></v-textarea>
                  </v-col>
                  <v-col cols="12">
                    <v-autocomplete
                      v-model="form.owner_id"
                      label="User"
                      placeholder="Search user by name"
                      :items="users"
                      item-text="name"
                      item-value="id"
                      :rules="[v => !!v || 'User is required']"
                      :error="form.errors.has('owner_id')"
                      :error-messages="form.errors.get('owner_id')"
                      :loading="usersLoading"
                      required
                      outlined
                      hide-no-data
                      :search-input.sync="searchUsers"
                      @input="fetchUsers"
                      @update:search-input="fetchUsers"
                    ></v-autocomplete>
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
  import axios from 'axios'
  import _ from 'lodash'
  import validationRules from '@/mixins/validationRules'
  
  export default {
    mixins: [validationRules],
    props: {
      showDialog: {
        type: Boolean,
      },
      property: {
        type: Object,
        default: () => ({})
      }
    },
    data() {
      return {
        form: new Form({
          name: '',
          location: '',
          description: '',
          owner_id: ''
        }),
        users: [],
        usersLoading: false,
        searchUsers: '',
        nameRules: [
          v => !!v || 'Name is required',
        ],
        addressRules: [
          v => !!v || 'Address is required',
        ],
        descriptionRules: [
          v => !!v || 'Description is required',
        ]
      }
    },
    computed: {
      isEdit() {
        return !!this.property.id
      }
    },
    watch: {
      property: {
        handler(newProperty) {
          if (newProperty && newProperty.id) {
            this.form.name = newProperty.name
            this.form.location = newProperty.location
            this.form.description = newProperty.description
            this.form.owner_id = newProperty.owner_id
            this.fetchUsers(newProperty.owner_id)
          }
        },
        immediate: true
      }
    },
    methods: {
      closeDialog() {
        this.$emit('close')
        this.form.reset()
      },
      saveProperty() {
        const isFormValid = this.$refs.form.validate()
        if (isFormValid) {
          const request = this.isEdit 
            ? this.form.put(`properties/${this.property.id}`)
            : this.form.post('properties')
          
          request
            .then(() => {
              this.$toast.success(`${this.isEdit ? 'Property updated' : 'Property added'} successfully`)
              this.$emit('close')
              this.$emit('property-added')
              this.form.reset()
            })
            .catch(error => {
              this.$toast.error(error.response.data.message)
            })
        }
      },
      fetchUsers: _.debounce(function (search) {
        if (!search) {
          this.users = [] 
          return
        }
  
        this.usersLoading = true
        axios
          .get('/users', {
            params: {
              search,
              searchColumn: "['name', 'email', 'phone_number']"
            }
          })
          .then(response => {
            this.users = response.data.data
          })
          .catch(error => {
            console.error(error)
            this.$toast.error(error.response.data.message)
          })
          .finally(() => {
            this.usersLoading = false
          })
      }, 300)
    }
  }
  </script>
  