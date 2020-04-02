<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable>
            <el-button slot="append" icon="el-icon-search" @click="getOrdersList"></el-button>
          </el-input>
        </el-col>
      </el-row>

      <el-table :data="ordersList" stripe border>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="order_number" label="订单编号"></el-table-column>
        <el-table-column prop="order_price" label="订单价格" width="95px"></el-table-column>
        <el-table-column prop="pay_status" label="是否付款" width="95px">
          <template v-slot="{ row }">
            <el-tag type="success" v-if="row.pay_status === '1'">已付款</el-tag>
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send" label="是否发货" width="70px"></el-table-column>
        <el-table-column prop="create_time" label="下单时间" width="140px">
          <template v-slot="{ row }">
            {{row.create_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template v-slot="{ row }">
            <el-tooltip content="修改地址信息" placement="top" :enterable="false" effect="dark">
              <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditAddressDialog(row.order_id)"></el-button>
            </el-tooltip>
            <el-button type="success" icon="el-icon-location" size="mini" @click="showProgressDialog()"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum"
      :page-sizes="[5, 10, 15, 20]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper"
      :total="total" background>
    </el-pagination>
    </el-card>

    <!-- 修改地址信息的对话框 -->
    <el-dialog
      title="修改地址信息"
      :visible.sync="editAddressDialogVisible"
      width="50%" @close="editAddressDialogClosed">
      <!-- 内容主体区域 -->
      <el-form :model="addressForm" :rules="addressFormRules" ref="addressFormRef" label-width="100px">
        <el-form-item label="省市区/县" prop="address1">
          <el-cascader :options="cityData" v-model="addressForm.address1"></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editAddressDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editAddressDialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 显示物流进度的对话框 -->
    <el-dialog
      title="物流进度"
      :visible.sync="progressDialogVisible"
      width="50%" @close="progressDialogClosed">
      <!-- 内容主体区域 -->
      <el-timeline>
        <el-timeline-item
          v-for="(item, index) in progressInfo"
          :key="index"
          :timestamp="item.time">
          {{item.context}}
        </el-timeline-item>
      </el-timeline>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="progressDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="progressDialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import cityData from './citydata.js'

export default {
  data () {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 5
      },
      ordersList: [],
      total: 0,
      editAddressDialogVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [
          { required: true, message: '请选择省市区县', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请填写详细地址', trigger: 'blur' }
        ]
      },
      cityData,
      progressDialogVisible: false,
      progressInfo: []
    }
  },
  created () {
    this.getOrdersList()
  },
  methods: {
    async getOrdersList () {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单列表数据失败')
      }
      this.ordersList = res.data.goods
      this.total = res.data.total
      // console.log(res)
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrdersList()
    },
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrdersList()
    },
    async showEditAddressDialog (id) {
      const { data: res } = await this.$http.get(`orders/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单详情失败')
      }
      // console.log(res)
      this.editAddressDialogVisible = true
    },
    editAddressDialogClosed () {
      this.$refs.addressFormRef.resetFields()
    },
    async showProgressDialog () {
      const { data: res } = await this.$http.get('/kuaidi/804909574412544580')
      if (res.meta.status !== 200) {
        return this.$message.error('获取物流进度失败')
      }
      // console.log(res)
      this.progressInfo = res.data
      this.progressDialogVisible = true
    },
    progressDialogClosed () {
    }
  }
}
</script>

<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>
