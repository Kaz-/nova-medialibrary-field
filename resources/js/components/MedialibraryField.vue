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

  data() {
    return {
      currentFormat: null
    }
  },

  computed: {
    shouldShow() {
      // Track format changes through data property
      this.currentFormat = this.field.resource.format;

      console.log('dependsOn:', this.field.dependsOn);
      console.log('resource:', this.field.resource);

      if (!this.field.dependsOn || Object.keys(this.field.dependsOn).length === 0) {
        return true;
      }

      const [field, value] = Object.entries(this.field.dependsOn)[0];
      console.log('field:', field);
      console.log('value:', value);
      console.log('resource[field]:', this.field.resource[field]);
      console.log('resource[field] === value:', this.field.resource[field] === value);

      return this.field.resource[field] === value;
    },
  },

  mounted() {
    // Listen for changes in the dependent field
    Nova.$on('format-change', (fieldValue) => {
      console.log('Format change event received:', fieldValue);
      if (this.field.resource) {
        this.currentFormat = fieldValue;  // Update data property first
        this.field.resource.format = fieldValue;
        this.$forceUpdate();  // Force a re-render
      }
    });
  },

  beforeDestroy() {
    Nova.$off('format-change');
  },
}
</script>
