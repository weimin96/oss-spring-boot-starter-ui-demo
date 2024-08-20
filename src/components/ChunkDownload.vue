<script setup>
import axios from "axios";
import {ref} from "vue";

const percentage = ref(0)

const downloadFile = async () => {
  const url = 'api/oss/object/preview/demo.zip';
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
    console.log(`===============开始下载：${range}`);
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
    percentage.value = Math.ceil((i + 1) * 100 / numChunks);
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
    <el-button @click="downloadFile">分片下载文件</el-button>
    <div v-if="percentage !== 0" class="demo-progress">
      <el-progress :percentage="percentage"
                   :text-inside="true"
                   :stroke-width="24"
                   striped-flow
                   status="success"/>
    </div>
  </div>
</template>

<style scoped>
.demo-progress .el-progress--line {
  margin-top: 15px;
  margin-bottom: 15px;
  max-width: 600px;
}
</style>
