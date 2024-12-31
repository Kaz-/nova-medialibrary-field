<template>
  <DefaultField
    v-if="shouldShow"
    :field="field"
    :errors="errors"
    :full-width-content="true"
    :show-help-text="showHelpText"
  >
    <template #field>
      <MedialibraryField
        :add-files="true"
        :field="field"
        :resource-name="resourceName"
        :resource-id="resourceId"
      />
    </template>
  </DefaultField>
</template>

<script>
import MedialibraryField from './MedialibraryField'
import { FormField, HandlesValidationErrors } from 'laravel-nova'

export default {
  components: {
    MedialibraryField,
  },

  mixins: [FormField, HandlesValidationErrors],

  props: ['resourceName', 'resourceId', 'field'],

  computed: {
    shouldShow() {
      const format = this.field.resource.format;



      if (!this.field.dependsOn || Object.keys(this.field.dependsOn).length === 0) {
        return true;
      }

      const [field, value] = Object.entries(this.field.dependsOn)[0];


      return this.field.resource[field] === value;
    },
  },

  data() {
    return {
      currentFormat: null
    }
  },

  methods: {
    setInitialValue() {
      this.value = this.field.value || ''
    },

    fill(formData) {
      formData.append(this.field.attribute, this.value || '')
    },

    handleChange(value) {
      this.value = value
    },
  },

  mounted() {
    Nova.$on('format-change', (fieldValue) => {
      if (this.field.resource) {
        this.currentFormat = fieldValue;
        this.field.resource.format = fieldValue;
        this.$forceUpdate();
      }
    });
  },

  beforeDestroy() {
    Nova.$off('format-change');
  },
}
</script>
