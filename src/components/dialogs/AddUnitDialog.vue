<template>
  <v-row justify="center">
    <v-dialog v-model="showDialog" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="text-h5">{{ isEdit ? 'Edit Unit' : 'Add Unit' }}</span>
        </v-card-title>
        <v-form ref="form" method="post" action="/" lazy-validation @submit.prevent="saveUnit">
          <v-card-text>
            <v-container>
              <v-row>
                <v-col cols="12">
                  <v-text-field
                    v-model="form.unit_number"
                    label="Unit Number"
                    placeholder="Enter unit number"
                    :rules="unitNumberRules"
                    :error="form.errors.has('unit_number')"
                    :error-messages="form.errors.get('unit_number')"
                    outlined
                  ></v-text-field>
                </v-col>
                <v-col cols="12">
                  <v-text-field
                    v-model="form.size"
                    label="Size"
                    placeholder="Enter unit size"
                    :rules="sizeRules"
                    :error="form.errors.has('size')"
                    :error-messages="form.errors.get('size')"
                    outlined
                  ></v-text-field>
                </v-col>
                <v-col cols="12">
                  <v-text-field
                    v-model="form.bedrooms"
                    label="Bedrooms"
                    placeholder="Enter number of bedrooms"
                    :rules="bedroomsRules"
                    :error="form.errors.has('bedrooms')"
                    :error-messages="form.errors.get('bedrooms')"
                    outlined
                    type="number"
                  ></v-text-field>
                </v-col>
                <v-col cols="12">
                  <v-text-field
                    v-model="form.rent"
                    label="Rent"
                    placeholder="Enter rent amount"
                    :rules="rentRules"
                    :error="form.errors.has('rent')"
                    :error-messages="form.errors.get('rent')"
                    outlined
                    type="number"
                  ></v-text-field>
                </v-col>
                <v-col cols="12">
                  <v-select
                    v-model="form.status"
                    label="Status"
                    :items="statusOptions"
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
import validationRules from '@/mixins/validationRules'

export default {
  mixins: [validationRules],
  props: {
    showDialog: {
      type: Boolean,
    },
    unit: {
      type: Object,
      default: () => ({}),
    },
    propertyId: {
      type: [Number, String],
      required: true,
    },
  },
  data() {
    return {
      form: new Form({
        unit_number: '',
        size: '',
        bedrooms: '',
        rent: '',
        status: '',
        property_id: this.propertyId,
      }),
      unitNumberRules: [v => !!v || 'Unit number is required'],
      sizeRules: [v => !!v || 'Size is required'],
      bedroomsRules: [
        v => !!v || 'Number of bedrooms is required',
        v => /^\d+$/.test(v) || 'Bedrooms must be a number',
      ],
      rentRules: [v => !!v || 'Rent is required', v => /^\d+(\.\d{1,2})?$/.test(v) || 'Rent must be a valid number'],
      statusOptions: [
        { text: 'Available', value: 1 },
        { text: 'Leased', value: 2 },
        { text: 'Reserved', value: 3 },
      ],
    }
  },
  computed: {
    isEdit() {
      return !!this.unit.id
    },
  },
  watch: {
    unit: {
      handler(newUnit) {
        if (newUnit && newUnit.id) {
          this.form.unit_number = newUnit.unit_number
          this.form.size = newUnit.size
          this.form.bedrooms = newUnit.bedrooms
          this.form.rent = newUnit.rent
          this.form.status = newUnit.status
        }
      },
      immediate: true,
    },
    propertyId: {
      handler(newPropertyId) {
        this.form.property_id = newPropertyId
      },
      immediate: true,
    },
  },
  methods: {
    closeDialog() {
      this.$emit('close')
      this.form.reset()
    },
    saveUnit() {
      const isFormValid = this.$refs.form.validate()
      if (isFormValid) {
        const request = this.isEdit
          ? this.form.put(`units/${this.unit.id}`)
          : this.form.post(`units/`)

        request
          .then(() => {
            this.$toast.success(`${this.isEdit ? 'Unit updated' : 'Unit added'} successfully`)
            this.$emit('close')
            this.$emit('unit-added')
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
