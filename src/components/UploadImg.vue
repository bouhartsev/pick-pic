<template>
  <div class="file-upload-container">
    <div class="file-upload">
      <input class="file-input" id="fileInput" type="file" accept="image/*" />
      <label class="file-label" for="fileInput">
        <i class="upload-icon">📁🖼</i>
        <p>Drag &amp; Drop your file here or click to upload</p>
      </label>
    </div>
  </div>
</template>

<style scoped>
  .file-upload-container {
    width: 50%;
    max-width: 500px;
  }

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
    color: white;
  }
</style>