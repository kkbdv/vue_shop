<template>
<div>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span='8'>
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="goodsSearch"
              @clear="getGoodsList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addGoods">添加商品</el-button>
        </el-col>
      </el-row>
    </el-card>
    <!-- table表格区域 -->
     <el-table
      :data="goodsTableData"
      style="width: 100%" border stripe>
      <el-table-column
        type="index"
        >
      </el-table-column>
      <el-table-column
        label="商品名称"
        prop="goods_name"
        >
      </el-table-column>
      <el-table-column
        label="商品价格(元)"
        prop="goods_price"
        width="95px"
        >
      </el-table-column>
      <el-table-column
        label="商品重量"
        prop="goods_weight"
        width="70px"
        >
      </el-table-column>
      <el-table-column
        label="创建时间"
        prop="add_time"
        width="140px"
        >
        <template slot-scope="scope">
          {{scope.row.add_time|dateFormat}}
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        width="200px"
        >
        <template  slot-scope="scope">
         <el-button size="mini" type="primary" icon="el-icon-edit" >编辑</el-button>
             <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeById(scope.row.goods_id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
     <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[5, 10, 20, 30]"
      :page-size="100"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total" background>
    </el-pagination>
</div>
</template>
<script>
export default {
  data () {
    return {
      inputSearch: '',
      // 查询参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      // 表格对象
      goodsTableData: [],
      // 总条数
      total: 0
    }
  },
  created () {
    this.getGoodsList()
  },
  methods: {
    async getGoodsList () {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error("获取数据失败")
      console.log(res.data)
      this.goodsTableData = res.data.goods
      this.total = res.data.total
    },
    // 分页大小改变
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    // 页码改变
    handleCurrentChange (newNum) {
      this.queryInfo.pagenum = newNum
      this.getGoodsList()
    },
    goodsSearch () {
      this.getGoodsList()
    },
    async removeById (id) {
      const resConfirm = await this.$confirm('是否删除商品?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (resConfirm !== 'confirm') {
        return this.$message.info("取消删除")
      }
      const { data: res } = await this.$http.delete(`goods/${id}`)
      if (res.meta.status !== 200) this.$message.error('删除失败')
      this.$message.success("删除成功")
      this.getGoodsList()
    },
    // 导航去Add页面
    addGoods () {
      this.$router.push('/goods/add')
    }
  }

}
</script>
<style lang="less" scoped>
</style>
