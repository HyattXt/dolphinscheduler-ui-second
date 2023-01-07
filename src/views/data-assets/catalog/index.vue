<template>
  <n-space vertical>
    <n-card size="small">
      <n-space justify="end" style="height: 40px">
        <n-form ref="formRef" :model="pagination">
          <n-grid :cols="10" :x-gap="12">
            <n-form-item-gi
              :span="8"
              :show-label="false"
              path="pagination.sqlLineageName"
            >
              <n-input
                size="small"
                v-model:value="pagination.sqlLineageName"
                placeholder="表名"
              />
            </n-form-item-gi>
            <n-form-item-gi :span="2" :show-label="false">
              <n-button
                size="small"
                type="primary"
                @click="handlePageChange(1)"
              >
                <NIcon :component="SearchOutlined"></NIcon>
              </n-button>
            </n-form-item-gi>
          </n-grid>
        </n-form>
      </n-space>
    </n-card>
    <n-card>
      <n-data-table
        ref="table"
        remote
        :columns="columns"
        :data="data"
        :loading="loading"
        :pagination="pagination"
        :row-key="rowKey"
        @update:page="handlePageChange"
      />
    </n-card>
  </n-space>
</template>

<script>
import {defineComponent, ref, reactive, onMounted, h} from 'vue'
import axios from 'axios'
import {
  ProfileOutlined,
  SearchOutlined
} from '@vicons/antd'
import {NButton, NIcon, NSpace, NTooltip} from "naive-ui";
import {useRouter} from "vue-router";

const columns = ({ play }) => {
    return [
      {
        title: '数据库',
        key: 'dataSourceName'
      },
      {
        title: '数据库类型',
        key: 'dbType'
      },
      {
        title: '表名',
        key: 'sqlLineageName'
      },
      {
        title: '注释',
        key: 'notes'
      },
      {
        title: '操作',
        key: 'actions',
        width: 132,
        render(row) {
          return h(NSpace, null, {
            default: () => [
              h(
                  NTooltip,
                  {},
                  {
                    trigger: () =>
                        h(
                            NButton,
                            {
                              circle: true,
                              type: 'info',
                              size: 'small',
                              class: 'edit',
                              onClick: () => {
                                play(row)
                              }
                            },
                            {
                              icon: () =>
                                  h(NIcon, null, { default: () => h(ProfileOutlined) })
                            }
                        ),
                    default: () => '查看详情'
                  }
              )
            ]
          })
        }
      }
    ]
  }
  const TableData = reactive({
    tableList: [],
    totalNum: 0
  })

  function query(
    page,
    pageSize = 10,
    sqlLineageName = ''
  ) {
    return new Promise((resolve) => {
      const url = import.meta.env.MODE === 'development'
          ? import.meta.env.VITE_APP_DEV_ASSETS_URL+'/getSqlLineageListByParams'
          : import.meta.env.VITE_APP_PROD_ASSETS_URL+'/getSqlLineageListByParams'
      const params = {
        'pageNum': page,
        'pageSize': pageSize,
        'sqlLineageName': sqlLineageName
      }

      axios
        .post(url, params)
        .then(function (response) {
          console.log(response)
          TableData.tableList = response.data.data
          TableData.totalNum = response.data.totalNum
          TableData.tableList.dbType = TableData.tableList.forEach((item) => {
            if (item.dbType === '0') {
              item.dbType = 'mysql'
            }
            if (item.dbType === '5') {
              item.dbType = 'oracle'
            }
          })
          console.log(TableData.tableList)
          console.log(TableData.totalNum)
          const copiedData = TableData.tableList.map((v) => v)
          const total = TableData.totalNum
          const pageCount = Math.ceil(total / pageSize)

          setTimeout(
            () =>
              resolve({
                pageCount,
                data: copiedData,
                total
              }),
            300
          )
        })
        .catch(function (error) {
          console.log(error)
        })
    })
  }

  export default defineComponent({
    setup() {
      const dataRef = ref([])
      const router = useRouter()
      const loadingRef = ref(true)
      const columnsRef = ref(
          columns(
              {
                play(row) {
                  router.push({
                        path: '/data-assets/assets-detail',
                        query: {tableName: row.sqlLineageName, tableComment: row.notes}
                      }
                  )
                }
              }
          )
      )
      const paginationReactive = reactive({
        page: 1,
        pageCount: 1,
        pageSize: 10,
        sqlLineageName: '',
        prefix({ itemCount }) {
          return `共${itemCount}条`
        }
      })

      onMounted(() => {
        query(
          paginationReactive.page,
          paginationReactive.pageSize,
          paginationReactive.sqlLineageName
        ).then((data) => {
          dataRef.value = data.data
          paginationReactive.pageCount = data.pageCount
          paginationReactive.itemCount = data.total
          loadingRef.value = false
        })
      })

      return {
        data: dataRef,
        pagination: paginationReactive,
        loading: loadingRef,
        columns: columnsRef,
        SearchOutlined,
        rowKey(rowData) {
          return rowData.colName
        },
        handlePageChange(currentPage) {
          if (!loadingRef.value) {
            loadingRef.value = true
            query(
              currentPage,
              paginationReactive.pageSize,
              paginationReactive.sqlLineageName
            ).then((data) => {
              dataRef.value = data.data
              paginationReactive.page = currentPage
              paginationReactive.pageCount = data.pageCount
              paginationReactive.itemCount = data.total
              loadingRef.value = false
            })
          }
        }
      }
    }
  })
</script>

<style scoped>
  a {
    text-decoration: none;
  }
</style>
