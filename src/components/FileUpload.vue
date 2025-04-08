<template>
  <div>
    <!-- API URL 輸入欄位 -->
    <label for="api-url">API URL:</label>
    <input
      id="api-url"
      type="text"
      v-model="apiUrl"
      placeholder="輸入 API URL"
    />

    <!-- 其他上傳功能 -->
    <input type="file" @change="handleFileUpload" />
    <button @click="uploadFile">上傳檔案</button>

    <div>
      <label>Delimiter:
        <select v-model="delimiter">
          <option value=",">Comma (,)</option>
          <option value=";">Semicolon (;)</option>
          <option value="\t">Tab (\t)</option>
        </select>
      </label>
      <label>
        <input type="checkbox" v-model="hasHeader" /> Has Header
      </label>
    </div>
    <button @click="submitFile">Clean CSV</button>
    <div v-if="loading">Cleaning...</div>
    <div v-if="summary">
      <h3>Summary</h3>
      <p>Original Rows: {{ summary.originalRowCount }}</p>
      <p>Cleaned Rows: {{ summary.cleanedRowCount }}</p>
      <p>Headers: {{ summary.standardizedHeaders.join(', ') }}</p>
      <button @click="downloadCleanedCSV">Download Cleaned CSV</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

const apiUrl = ref(import.meta.env.VITE_API_BASE_URL || ""); // 預設為 .env 中的值
const file = ref(null);
const delimiter = ref(',');
const hasHeader = ref(true);
const loading = ref(false);
const summary = ref(null);
const downloadUrl = ref('');

const handleFileUpload = (event) => {
  file.value = event.target.files[0];
};

const submitFile = async () => {
  if (!file.value) return;
  loading.value = true;
  const formData = new FormData();
  formData.append('file', file.value);
  formData.append('delimiter', delimiter.value);
  formData.append('hasHeader', hasHeader.value);

  try {
    const response = await axios.post(`${apiUrl.value}/api/csv/clean`, formData, {
      headers: { 'Content-Type': 'multipart/form-data' },
    });
    summary.value = response.data;
    downloadUrl.value = response.data.downloadUrl;
  } catch (err) {
    alert('Failed to clean CSV.');
  } finally {
    loading.value = false;
  }
};

const downloadCleanedCSV = async () => {
  const response = await axios.get(downloadUrl.value, { responseType: 'blob' });
  const blob = new Blob([response.data], { type: 'text/csv' });
  const link = document.createElement('a');
  link.href = URL.createObjectURL(blob);
  link.download = 'cleaned.csv';
  link.click();
};

const uploadFile = async () => {
  if (!apiUrl.value) {
    alert("請輸入有效的 API URL");
    return;
  }
  if (!file.value) {
    alert("請選擇檔案");
    return;
  }

  const formData = new FormData();
  formData.append("file", file.value);

  try {
    const response = await axios.post(`${apiUrl.value}/api/csv/clean`, formData);
    alert("檔案清理成功");
    console.log(response.data);
  } catch (error) {
    alert("檔案清理失敗");
    console.error(error);
  }
};
</script>
