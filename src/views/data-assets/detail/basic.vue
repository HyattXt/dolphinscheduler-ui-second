<template>
  <n-card title="字段信息" size="large" :bordered="false" >
    <n-table :single-line="false" size="small">
      <thead>
      <tr>
        <th>字段名</th>
        <th>字段类型</th>
        <th>字段注释</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="item in fieldInfo">
        <td>{{ item.colName }}</td>
        <td>{{ item.colDataType }}</td>
        <td>{{ item.colComment }}</td>
      </tr>
      </tbody>
    </n-table>
  </n-card>
</template>
<script setup>

import axios from 'axios'
import { onMounted, ref} from "vue";
import {useRoute} from "vue-router";
import {useMessage} from "naive-ui";

const fieldInfo =ref([])
const route = useRoute()
const message = useMessage()

onMounted(() => {
  //const url = "http://127.0.0.1:4523/m1/1263204-0-default/hive/tableInfo?tableName="+route.query.tableName
  const url = import.meta.env.MODE === 'development'
      ? import.meta.env.VITE_APP_DEV_ASSETS_URL+'/hive/tableInfo?tableName='+route.query.tableName
      : import.meta.env.VITE_APP_PROD_ASSETS_URL+'/hive/tableInfo?tableName='+route.query.tableName
  axios
      .get(url)
      .then(function (response) {
        console.log(response)
        fieldInfo.value = response.data
        console.log(fieldInfo.value)
      })
      .catch(function () {
        message.error('请求数据失败，请咨询管理员')
      })
})
</script>

<style scoped>

</style>
