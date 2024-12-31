<template>
  <Provider v-if="shouldShow" :field="field" :resource-name="resourceName" :resource-id="resourceId">
    <div>
      <MediaList dusk="nova-media-list" />
      <MediaUploading v-if="addFiles && !field.readonly" class="mt-2" dusk="nova-media-uploading" />
    </div>
  </Provider>
</template>

<script>
import { Provider } from './Medialibrary/Context'
import MediaList from './Medialibrary/MediaList'
import MediaUploading from './Medialibrary/MediaUploading'

export default {
  components: {
    Provider,
    MediaList,
    MediaUploading,
  },

  props: {
    resourceName: {
      type: String,
      required: true,
    },
    resourceId: {
      required: true,
    },
    field: {
      type: Object,
      required: true,
    },
    addFiles: {
      type: Boolean,
      default: false,
    },
  },

  computed: {
    shouldShow() {
      // If dependsOn is not set, always show the field
      if (!this.field.dependsOn || Object.keys(this.field.dependsOn).length === 0) {
        return true;
      }

      // Get the dependent field and its expected value
      const [field, value] = Object.entries(this.field.dependsOn)[0];

      // Check if the dependent field's value matches the expected value
      return this.$parent.resource[field] === value;
    },
  },

  mounted() {
    // Listen for changes in the dependent field
    Nova.$on('field-change', (fieldAttribute, fieldValue) => {
      if (this.field.dependsOn && fieldAttribute === Object.keys(this.field.dependsOn)[0]) {
        // Force re-render when the dependent field changes
        this.$forceUpdate();
      }
    });
  },

  beforeDestroy() {
    // Clean up the event listener
    Nova.$off('field-change');
  },
}
</script>
