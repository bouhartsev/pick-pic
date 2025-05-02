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
    handleImageSelection(event: Event) {
      const target = event.target as HTMLInputElement | null;
      if (!target) return;

      const selectedFile = target.files?.[0];
      if (selectedFile) {
        const imageUrl = URL.createObjectURL(selectedFile);
        this.selectedImage = imageUrl;
        this.$emit('onImageSelected', imageUrl);

        target.value = '';
      }
    },
    loadImageFromUrl(e: Event) {
      e.preventDefault();
      if (this.inputUrl) {
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
      <input class="file-input" id="fileInput" type="file" accept="image/*" @change="handleImageSelection" />
      <label class="file-label" for="fileInput">
        <span class="upload-icon">🖼</span>
        <p>Drag &amp; Drop your file here or click to upload</p>
      </label>
      <form action="" method="get" @submit="loadImageFromUrl" class="message-box">
        <input required class="url-input" v-model="inputUrl" placeholder="URL" aria-label="Image URL to upload" />
        <button type="submit" class="button url-button" @click="loadImageFromUrl" aria-label="Load image from URL">
          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-upload"
            viewBox="0 0 16 16">
            <path
              d="M.5 9.9a.5.5 0 0 1 .5.5v2.5a1 1 0 0 0 1 1h12a1 1 0 0 0 1-1v-2.5a.5.5 0 0 1 1 0v2.5a2 2 0 0 1-2 2H2a2 2 0 0 1-2-2v-2.5a.5.5 0 0 1 .5-.5z" />
            <path
              d="M7.646 1.146a.5.5 0 0 1 .708 0l3 3a.5.5 0 0 1-.708.708L8.5 2.707V11.5a.5.5 0 0 1-1 0V2.707L5.354 4.854a.5.5 0 1 1-.708-.708l3-3z" />
          </svg>
        </button>
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
  text-align: center;
  background-color: inherit;
  transition: background-color 0.3s ease-in-out;
  height: 500px;
  max-height: 80%;
}

.file-upload:hover:not(:has(.message-box:hover)) {
  background-color: var(--color-background-mute);
}

.file-input {
  display: none;
}

.file-label {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  cursor: pointer;

  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.upload-icon {
  font-size: 50px;
  color: #007bff;
  margin-bottom: 10px;
}

.file-upload p {
  margin: 0;
  font-size: 2rem;
  color: #6c757d;
}

.file-upload.dragover {
  background-color: #007bff;
  color: var(--color-text);
}

.message-box {
  position: absolute;
  bottom: 40px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 2;

  width: fit-content;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #2d2d2d;
  /* padding: 0 15px; */
  /* padding-right: 15px; */
  border-radius: 10px;
  border: 1px solid rgb(63, 63, 63);
}

.message-box:focus-within {
  border: 1px solid rgb(110, 110, 110);
}

.url-input {
  width: 200px;
  height: 100%;
  background-color: transparent;
  outline: none;
  border: none;
  padding-left: 10px;
  color: white;
}

.url-button:hover,
.url-input:focus~.url-button,
.url-input:valid~.url-button {
  color: white;
}

.url-button {
  width: fit-content;
  height: 100%;
  background-color: transparent;
  outline: none;
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s;

  padding-inline: 15px;
}

/* .url-button svg {
  height: 18px;
  transition: all 0.3s;
}
.url-button svg path {
  transition: all 0.3s;
} */
/* .url-button:hover svg path {
  fill: #3c3c3c;
  stroke: white;
} */
</style>
