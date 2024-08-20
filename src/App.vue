<script setup xmlns="http://www.w3.org/1999/html">
import ChunkDownload from "@/components/ChunkDownload.vue";
import Download from "@/components/Download.vue";
</script>

<template>
  <el-main>
    <el-row class="row-layout">
      <el-col :span="24" >
        <ChunkDownload/>
      </el-col>
      <el-col :span="24">
        <Download/>
      </el-col>
    </el-row>
  </el-main>
</template>

<style scoped>
.row-layout {
  .el-col {
    margin-bottom: 20px;
  }
}
</style>

