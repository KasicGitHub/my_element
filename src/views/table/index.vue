<template>
  <div class="app-container">
<!--    <el-table-->
<!--      v-loading="listLoading"-->
<!--      :data="list"-->
<!--      element-loading-text="Loading"-->
<!--      border-->
<!--      fit-->
<!--      highlight-current-row>-->
<!--      <el-table-column align="center" label="ID" width="95">-->
<!--        <template slot-scope="scope">-->
<!--          {{ scope.$index }}-->
<!--        </template>-->
<!--      </el-table-column>-->
<!--      <el-table-column label="Title">-->
<!--        <template slot-scope="scope">-->
<!--          {{ scope.row.title }}-->
<!--        </template>-->
<!--      </el-table-column>-->
<!--      <el-table-column label="Author" width="110" align="center">-->
<!--        <template slot-scope="scope">-->
<!--          <span>{{ scope.row.author }}</span>-->
<!--        </template>-->
<!--      </el-table-column>-->
<!--      <el-table-column label="Pageviews" width="110" align="center">-->
<!--        <template slot-scope="scope">-->
<!--          {{ scope.row.pageviews }}-->
<!--        </template>-->
<!--      </el-table-column>-->
<!--      <el-table-column class-name="status-col" label="Status" width="110" align="center">-->
<!--        <template slot-scope="scope">-->
<!--          <el-tag :type="scope.row.status | statusFilter">{{ scope.row.status }}</el-tag>-->
<!--        </template>-->
<!--      </el-table-column>-->
<!--      <el-table-column align="center" prop="created_at" label="Display_time" width="200">-->
<!--        <template slot-scope="scope">-->
<!--          <i class="el-icon-time"/>-->
<!--          <span>{{ scope.row.display_time }}</span>-->
<!--        </template>-->
<!--      </el-table-column>-->
<!--    </el-table>-->

    <el-table
      :data="tableData"
      border
      style="width: 100%">
      <el-table-column
        prop="date"
        label="日期"
        width="180">
      </el-table-column>
      <el-table-column
        prop="name"
        label="姓名"
        width="180">
      </el-table-column>
      <el-table-column
        prop="address"
        label="地址">
      </el-table-column>
      <el-table-column prop="hosts" label="服务器">
        <template slot-scope="scope">
          <el-popover
            placement="right"
            title="IP列表"
            width="150"
            trigger="click">
            <p v-for="(ip,index) in scope.row.host_list" :key="index"> {{ ip }}</p>
            <el-button slot="reference" type="text" class="badgeItem">
              <el-badge :value="scope.row.host_list.length" :max="99">
                <el-button size="small">{{ scope.row.host_list[0] }}</el-button>
              </el-badge>
            </el-button>

          </el-popover>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import { getList } from '@/api/table'

export default {
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'gray',
        deleted: 'danger'
      }
      return statusMap[status]
    }
  },
  data() {
    return {
      list: null,
      listLoading: true,
      tableData: [{
        date: '2016-05-02',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1518 弄',
        hosts: '1.1.1.1'
      }, {
        date: '2016-05-04',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1517 弄',
        hosts: '1.1.1.1,2.2.2.2'
      }, {
        date: '2016-05-01',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1519 弄',
        hosts: '1.1.1.1,2.2.2.2,3.3.3.3'
      }, {
        date: '2016-05-03',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1516 弄',
        hosts: '1.1.1.1,2.2.2.2,3.3.3.3,4.4.4.4'
      }]
    }
  },
  created() {
    this.getTableData()
  },
  methods: {
    fetchData() {
      this.listLoading = true
      getList().then(response => {
        this.list = response.data.items
        this.listLoading = false
      })
    },
    getTableData() {
      this.tableData.forEach((item) => {
        if (JSON.stringify(item.hosts) !== '') {
          item['host_list'] = item.hosts.split(',')
        }
      })
    }
  }
}
</script>

<style scoped>

  .badgeItem {
    margin-top: 10px;
    margin-right: 40px;
  }

</style>
