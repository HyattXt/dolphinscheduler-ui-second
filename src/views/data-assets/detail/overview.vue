<template>
  <n-card :bordered="false" size="small" >
    <n-alert title="仅显示前10条数据" type="info">
    </n-alert>
    <n-scrollbar x-scrollable >
    <n-table :single-line="false" style="margin-top: 10px">
      <thead>
      <tr>
        <th class="tableWidth" v-for="(value,key) in dataInfo[0]">{{ key.replace(route.query.tableName+'.','') }}</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="item in dataInfo" >
        <td class="tableWidth" v-for="value in item">{{ value }}</td>
      </tr>
      </tbody>
    </n-table>
    </n-scrollbar>
  </n-card>
</template>
<script setup>

import axios from 'axios'
import { onMounted, ref} from "vue";
import {useRoute} from "vue-router";
import {useMessage} from "naive-ui";

const dataInfo =ref([])
const route = useRoute()
const message = useMessage()

onMounted(() => {
  const url = import.meta.env.MODE === 'development'
      ? import.meta.env.VITE_APP_DEV_ASSETS_URL+'/hive/dataByTableName?tableName='+route.query.tableName
      : import.meta.env.VITE_APP_PROD_ASSETS_URL+'/hive/dataByTableName?tableName='+route.query.tableName
  //const url = "http://127.0.0.1:4523/m1/1263204-0-default/hive/dataByTableName?tableName="+route.query.tableName
  axios
      .get(url)
      .then(function (response) {
        console.log(response)
        dataInfo.value = response.data
      })
      .catch(function () {
        message.error('请求数据失败，请咨询管理员')
      })
})
</script>

<style scoped>
.tableWidth{
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
</style>
