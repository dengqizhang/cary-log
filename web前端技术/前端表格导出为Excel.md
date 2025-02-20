# Vue3表格导出

本文记录一下如何在Vue3下使用TS语法，elementPlusUI库的表格，实现表格导出功能。


## 下载依赖

`npm install element-plus --save`

`npm install xlsx --save`




## 编写导出函数

```
<script setup lang="ts">
import { ref, onMounted } from "vue";
import * as XLSX from "xlsx";
const tableData = ref([
  {
    id: 1,
    name: "张三",
    age: 12,
  },
  {
    id: 2,
    name: "李四",
    age: 13,
  },
]);
/**
 * 导出函数
 */
const exportChange= () => {
  console.log(tableData);

  if (tableData.value.length === 0) {
    console.log("暂无数据");
    return;
  }
  // 1. 定义中文字段映射关系
  const headerMap = {
    name: "姓名",
    age: "年龄",
  };

  // 2. 构建带中文表头的数据
  const ChineseData = [
    Object.values(headerMap), // 中文表头
    ...tableData.value.map((item) => [item.name, item.age]),
  ];

  // 3. 创建带中文表头的工作表
  const worksheet = XLSX.utils.aoa_to_sheet(ChineseData);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, worksheet, "人员");
  XLSX.writeFile(wb, "人员表.xlsx");
};
</script>

<template>
  <button @click="exportChange()">测试</button>
  <el-table :data="tableData" style="width: 100%">
    <el-table-column prop="name" label="姓名" width="180" />
    <el-table-column prop="age" label="年龄" width="180" />
  </el-table>
</template>

<style scoped>
</style>

```


## 了解XLSX

XLSX的使用文档在SheetJS的官网

`https://docs.sheetjs.com/docs/solutions/output`


## 版本信息

- node：18.16.0

- vue3