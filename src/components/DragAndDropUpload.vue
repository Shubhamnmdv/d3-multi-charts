<template>
    <div class="upload-container">
      <h3>Upload</h3>
      <p class="desc">Add your documents here, and you can upload up to 5 files max</p>
  
      <div
        class="drop-area"
        @dragover.prevent
        @drop.prevent="handleDrop"
      >
        <div class="drop-inner">
          <div class="icon"></div>
          <p>Drag your file(s) to start uploading</p>
          <span class="or">OR</span>
          <div class="actions">
            <input
              type="file"
              ref="fileInput"
              multiple
              accept=".pdf"
              hidden
              @change="handleFileSelect"
            />
            <button class="btn" @click="$refs.fileInput.click()">Browse files</button>
            <button class="btn" @click="openLinkInput">Enter website link</button>
          </div>
        </div>
      </div>
  
      <div v-if="websiteLinkVisible" class="website-link">
        <input type="text" v-model="websiteLink" placeholder="Enter website URL" />
      </div>
  
      <p class="note">Only support .pdf</p>
  
      <input
        class="email-input"
        type="email"
        v-model="email"
        placeholder="Enter your email"
        @blur="validateEmail"
      />
      <span v-if="emailError" class="error">{{ emailError }}</span>
  
      <button class="generate-btn" @click="submitForm">Generate Schedule</button>
  
      <ul class="file-list">
        <li v-for="(file, index) in uploadedFiles" :key="index">
          {{ file.name }}
          <button @click="removeFile(index)">✕</button>
        </li>
      </ul>
  
      <div v-if="fileError" class="error">{{ fileError }}</div>
    </div>
  </template>
  
  <script>
  export default {
    name: "DragAndDropUpload",
    data() {
      return {
        uploadedFiles: [],
        websiteLinkVisible: false,
        websiteLink: "",
        email: "",
        emailError: "",
        fileError: "",
      };
    },
    methods: {
      handleDrop(event) {
        const files = Array.from(event.dataTransfer.files);
        this.processFiles(files);
      },
      handleFileSelect(event) {
        const files = Array.from(event.target.files);
        this.processFiles(files);
      },
      processFiles(files) {
        const validFiles = files.filter(file => file.type === "application/pdf");
        if (validFiles.length + this.uploadedFiles.length > 5) {
          this.fileError = "You can upload a maximum of 5 PDF files.";
          return;
        }
        if (validFiles.length !== files.length) {
          this.fileError = "Only .pdf files are allowed.";
        } else {
          this.fileError = "";
        }
        this.uploadedFiles.push(...validFiles);
      },
      removeFile(index) {
        this.uploadedFiles.splice(index, 1);
      },
      openLinkInput() {
        this.websiteLinkVisible = true;
      },
      validateEmail() {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        this.emailError = emailRegex.test(this.email) ? "" : "Please enter a valid email address.";
      },
      submitForm() {
        this.validateEmail();
        if (this.uploadedFiles.length === 0) {
          this.fileError = "Please upload at least one PDF file.";
          return;
        }
        if (this.uploadedFiles.length > 5) {
          this.fileError = "You can upload a maximum of 5 files.";
          return;
        }
        if (this.emailError || !this.email) {
          this.emailError = "A valid email is required.";
          return;
        }
  
        // Handle submission logic here
        alert("Submitted Successfully!");
      },
    },
  };
  </script>
  
  <style scoped>
  .upload-container {
    max-width: 500px;
    margin: auto;
    padding: 24px;
    border: 1px solid #ddd;
    border-radius: 12px;
    font-family: Arial, sans-serif;
  }
  h3 {
    margin-bottom: 8px;
  }
  .desc {
    font-size: 14px;
    color: #555;
  }
  .drop-area {
    border: 2px dashed #007bff;
    border-radius: 10px;
    padding: 24px;
    text-align: center;
    margin-top: 16px;
  }
  .drop-inner .icon {
    width: 48px;
    height: 48px;
    margin: auto;
    background-image: url('https://cdn-icons-png.flaticon.com/512/561/561127.png');
    background-size: contain;
    background-repeat: no-repeat;
  }
  .or {
    margin: 8px 0;
    display: block;
    font-weight: bold;
  }
  .actions {
    display: flex;
    justify-content: center;
    gap: 10px;
    margin-top: 8px;
  }
  .btn {
    padding: 6px 12px;
    border: 1px solid #007bff;
    background: #fff;
    color: #007bff;
    cursor: pointer;
    border-radius: 4px;
  }
  .btn:hover {
    background-color: #007bff;
    color: #fff;
  }
  .website-link input {
    width: 100%;
    margin: 10px 0;
    padding: 8px;
    border-radius: 4px;
    border: 1px solid #aaa;
  }
  .note {
    font-size: 12px;
    color: #777;
    margin-top: 10px;
  }
  .email-input {
    width: 100%;
    padding: 8px;
    margin-top: 8px;
    border-radius: 4px;
    border: 1px solid #aaa;
  }
  .generate-btn {
    margin-top: 16px;
    width: 100%;
    padding: 10px;
    background-color: #007bff;
    border: none;
    color: white;
    font-weight: bold;
    border-radius: 4px;
    cursor: pointer;
  }
  .generate-btn:hover {
    background-color: #0056b3;
  }
  .file-list {
    margin-top: 10px;
    list-style: none;
    padding: 0;
  }
  .file-list li {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 6px;
    border: 1px solid #ccc;
    border-radius: 4px;
    margin-top: 4px;
    font-size: 14px;
  }
  .file-list button {
    background: transparent;
    border: none;
    font-size: 16px;
    color: red;
    cursor: pointer;
  }
  .error {
    color: red;
    font-size: 12px;
    margin-top: 4px;
  }
  </style>
  