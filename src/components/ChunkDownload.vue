<script setup>
import axios from "axios";

const downloadFile = async () => {
  const url = 'api/oss/object/preview/test/WebStorm-2024.1.5.exe';
  // 分段大小  5MB
  const chunkSize = 1024 * 1024 * 5;
  let start = 0;
  let end = chunkSize - 1;
  let chunks = [];

  // 获取文件总大小和文件名
  const response = await axios.head(url);
  let fileSize = response.headers['content-length'];
  console.log(`===============文件大小：${fileSize}`);
  const fileName = getFileName(response.headers.get('Content-Disposition'));
  console.log(`===============文件名：${fileName}`);
  // 计算分段数量
  const numChunks = Math.ceil(fileSize / chunkSize);
  console.log(`===============分段数量：${numChunks}`);
  if (fileSize < chunkSize) {
    end = fileSize - 1;
  }
  // 分段下载
  for (let i = 0; i < numChunks; i++) {
    const range = `bytes=${start}-${end}`;
    console.log(`===============下载：${range}`);
    const config = {
      headers: {
        Range: range
      },
      responseType: 'arraybuffer'
    };
    const response = await axios.get(url, config);
    chunks.push(response.data);
    start = end + 1;
    end = Math.min(end + chunkSize, fileSize - 1);
  }

  // Combine chunks into a single file
  const blob = new Blob(chunks);
  const link = document.createElement('a');
  link.href = window.URL.createObjectURL(blob);
  link.download = fileName;
  link.click();
}

/**
 * 获取文件名
 * @param contentDisposition 请求头contentDisposition
 * @returns {string} 文件名
 */
const getFileName = (contentDisposition) => {
  let fileName = 'downloaded-file'; // 默认文件名
  if (contentDisposition) {
    const matches = /filename[^;=\n]*=((['"]).*?\2|[^;\n]*)/.exec(contentDisposition);
    if (matches != null && matches[1]) {
      // 去掉引号并解码
      fileName = decodeURIComponent(matches[1].replace(/['"]/g, ''));
    }
  }
  return fileName;
}
</script>

<template>
  <div>
    <button @click="downloadFile">分片下载文件</button>
  </div>
</template>
