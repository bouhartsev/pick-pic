<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'UploadImg',
  props: {
    state: String,
  },
  data() {
    return {
      selectedImage: '',
      inputUrl: '',
    };
  },
  methods: {
    handleImageSelection(event) {
      const target = event.target;
      const selectedFile = target.files?.[0];

      if (selectedFile) {
        const imageUrl = URL.createObjectURL(selectedFile);
        this.selectedImage = imageUrl;
        this.$emit('onImageSelected', imageUrl);

        target.value = '';
      }
    },
    loadImageFromUrl(e) {
      e.preventDefault();
      if (this.inputUrl) {
        console.log(this.inputUrl);
        this.$emit('onImageSelected', this.inputUrl);
        this.selectedImage = this.inputUrl;
        this.inputUrl = '';
      }
    },
  },
  beforeUnmount() {
    if (this.selectedImage) {
      URL.revokeObjectURL(this.selectedImage);
    }
  },
});
</script>

<template>
  <div class="file-upload-container">
    <div class="file-upload">
      <input
        class="file-input"
        id="fileInput"
        type="file"
        accept="image/*"
        @change="handleImageSelection"
      />
      <label class="file-label" for="fileInput">
        <span class="upload-icon">🖼</span>
        <p>Drag &amp; Drop your file here or click to upload</p>
      </label>
      <form action="" method="get" @submit="loadImageFromUrl">
        <input class="url-input" v-model="inputUrl" placeholder="URL" />
        <button type="submit" class="button" @click="loadImageFromUrl">Load</button>
      </form>
    </div>
  </div>
</template>

<style scoped>
/* .file-upload-container {
  width: 50%;
  max-width: 500px;
  margin: ;
} */

.file-upload {
  position: relative;
  border: 2px dashed currentColor;
  border-radius: 10px;
  padding: 40px;
  text-align: center;
  background-color: inherit;
  transition: background-color 0.3s ease-in-out;
}

.file-upload:hover {
  background-color: #e2e6ea;
}

.file-input {
  display: none;
}

.file-label {
  display: flex;
  flex-direction: column;
  align-items: center;
  cursor: pointer;
}

.upload-icon {
  font-size: 50px;
  color: #007bff;
  margin-bottom: 10px;
}

.file-upload p {
  margin: 0;
  font-size: 16px;
  color: #6c757d;
}

.file-upload.dragover {
  background-color: #007bff;
  color: var(--color-text);
}

.url-input {
  margin: 1em;
  /* padding: 10px;
  border: 2px solid #007bff;
  border-radius: 5px; */
}
</style>