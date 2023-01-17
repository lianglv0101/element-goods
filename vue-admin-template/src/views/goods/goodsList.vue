<template>
  <div class="app-container">
    <el-form :inline="true" :model="queryMap">
      <el-form-item label="商品名称">
        <el-input v-model="queryMap.name" placeholder="商品名称" size="small" />
      </el-form-item>
      <!--      <el-form-item label="活动区域">-->
      <!--        <el-select v-model="queryForm.region" placeholder="活动区域" size="small">-->
      <!--          <el-option label="区域一" value="shanghai" />-->
      <!--          <el-option label="区域二" value="beijing" />-->
      <!--        </el-select>-->
      <!--      </el-form-item>-->
      <el-form-item>
        <el-button type="primary" size="small" @click="query" icon="el-icon-search">查询</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" size="small" @click="add" icon="el-icon-edit">新增</el-button>
      </el-form-item>
    </el-form>

    <el-table
      v-loading="loading"
      :data="tableData"
      border
      style="width: 100%"
    >
      <el-table-column align="center" label="序号" type="index" width="80" />
      <el-table-column
        prop="name"
        label="商品名称"
        width="200"
      />
      <el-table-column
        prop="details"
        label="商品描述"
        width=""
      />
      <el-table-column
        prop="num"
        label="浏览次数"
        width="180"
      />
      <el-table-column
        prop="price"
        label="商品价格"
        width="180"
      />
      <el-table-column
        prop="fbsj"
        label="发布时间"
        width="250"
      />
      <el-table-column
        align="center"
        fixed="right"
        label="操作"
        width="180"
      >
        <template slot-scope="scope">
          <el-button type="primary" size="small" icon="el-icon-edit" @click="edit(scope.row.id, false)"></el-button>
          <el-button type="success" size="small" icon="el-icon-view" @click="edit(scope.row.id, true)"></el-button>
          <el-button type="danger" size="small" icon="el-icon-delete" @click="del(scope.row.id)"></el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      style="margin-top: 50px;float: right"
      background
      :page-size="queryMap.pageSize"
      layout="total, prev, pager, next, jumper"
      :total="queryMap.total"
      @current-change="handleCurrentChange"
    />

    <add-or-update v-if="dialogFormVisible" ref="addOrUpdate" @refreshDataList="query"></add-or-update>
  </div>
</template>
<script>
import AddOrUpdate from './goodsEdit'
import request from '@/utils/request'
export default {
  data() {
    return {
      fileList: [],
      dialogImageUrl: '',
      dialogVisible: false,
      disabled: false,
      loading: false,
      queryMap: {
        pageSize: 10,
        currentPage: 1,
        total: 0,
        name: '',
        type: '1'
      },
      tableData: [],
      dialogFormVisible: false
    }
  },
  components: { AddOrUpdate },
  created() {
    this.query()
  },
  methods: {
    handleCurrentChange(currentPage) {
      this.queryMap.currentPage = currentPage
      this.query()
    },
    edit(id, disabled) {
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id, disabled)
      })
    },
    del(id) {
      this.$confirm('此操作将永久删除该条记录, 是否继续?').then(() => {
        request({
          url: '/goods/deleteGoods/' + id,
          method: 'delete'
        }).then(res => {
          this.$message.success('删除成功')
          this.query()
        })
      }).catch(() => {
        this.$message.info('已取消删除')
      })
    },
    add() {
      // this.temp.timestamp = new Date(this.form.fbsj)
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(null, false)
      })
    },
    // 查询
    query() {
      this.loading = true
      this.dialogFormVisible = false
      request({
        url: '/goods/page',
        method: 'get',
        params: this.queryMap
      }).then(res => {
        this.tableData = res.data.records
        this.queryMap.total = res.data.total
        this.queryMap.currentPage = res.data.current
        this.loading = false
      })
    }
  }
}
</script>

<style scoped>
.app-container {
  padding: 20px;
}
</style>
