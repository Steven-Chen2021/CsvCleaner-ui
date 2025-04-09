<template>
  <div>
    <!-- API URL 輸入欄位 -->
    <label for="api-url">API URL:</label>
    <input
      id="api-url"
      type="text"
      v-model="apiUrl"
      placeholder="輸入 API URL"/><br />

    <!-- 其他上傳功能 -->
    <input type="file" @change="handleFileUpload" /><br />

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
      <h4>Corrections:</h4>
      <ul>
        <li>Blank Rows Removed: {{ summary.corrections.blankRowsRemoved }}</li>
        <li>Invalid Emails Corrected: {{ summary.corrections.invalidEmailsCorrected }}</li>
        <li>Extra Whitespace Trimmed: {{ summary.corrections.extraWhitespaceTrimmed }}</li>
        <li>Inconsistent Casing Fixed: {{ summary.corrections.inconsistentCasingFixed }}</li>
        <li>Duplicate Entries Removed: {{ summary.corrections.duplicateEntriesRemoved }}</li>
        <li>Date Formats Standardized: {{ summary.corrections.dateFormatsStandardized }}</li>
        <li>Currency Formats Standardized: {{ summary.corrections.currencyFormatsStandardized }}</li>
      </ul>
      <button @click="downloadCleanedCSV">Download Cleaned CSV</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

// 使用 ref 並設定預設值
const apiUrl = ref(import.meta.env.VITE_API_BASE_URL || "http://localhost:5171");
const file = ref(null);
const delimiter = ref(',');
const hasHeader = ref(true);
const loading = ref(false);
const summary = ref(null); // Initialize as null to hide the summary initially

const handleFileUpload = (event) => {
  const uploadedFile = event.target.files[0];
  if (uploadedFile && uploadedFile.type !== 'text/csv') {
    alert("請選擇 CSV 檔案");
    event.target.value = ''; // Reset the file input
    file.value = null;
  } else {
    file.value = uploadedFile;
  }
};

const submitFile = async () => {
  if (!file.value) {
    alert("請選擇檔案");
    return;
  }
  if (!apiUrl.value) {
    alert("請輸入有效的 API URL");
    return;
  }

  loading.value = true;
  const formData = new FormData();
  formData.append('file', file.value);
  formData.append('delimiter', delimiter.value);
  formData.append('hasHeader', hasHeader.value);

  try {
    const response = await axios.post(`${apiUrl.value}/api/csv/clean`, formData, {
      headers: { 'Content-Type': 'multipart/form-data' },
    });
    console.log('Before summary:', summary.value);
    
    summary.value = {
      originalRowCount: response.data.originalRowCount,
      cleanedRowCount: response.data.cleanedRowCount,
      corrections: response.data.corrections,
      downloadUrl: response.data.downloadUrl, // 更新 downloadUrl
    };
    console.log('After summary:',summary.value);
    alert("清理成功");
    
  } catch (err) {
    alert("清理失敗");
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const downloadCleanedCSV = async () => {
    if (!summary.value.downloadUrl) {
        alert('Download URL is not available.');
        return;
    }
    const response = await axios.get(summary.value.downloadUrl, { responseType: 'blob' });
    const blob = new Blob([response.data], { type: 'text/csv' });
    const link = document.createElement('a');
    link.href = URL.createObjectURL(blob);
    link.download = 'cleaned.csv';
    link.click();
};
</script>
