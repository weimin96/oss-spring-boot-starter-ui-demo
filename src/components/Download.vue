<script setup>

const downloadFile = async () => {
  const url = 'api/oss/object/preview/test/WebStorm-2024.1.5.exe';
  const link = document.createElement('a');
  link.href = url;
  link.download = 'WebStorm-2024.1.5.exe';
  link.click();
}
</script>

<template>
  <div>
    <el-button @click="downloadFile">下载文件</el-button>
  </div>
</template>
