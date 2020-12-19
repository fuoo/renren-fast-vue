<template>
  <div>
    <el-tree :data="menus"
             :props="defaultProps"
             :expand-on-click-node="false"
             show-checkbox
             node-key="catId">
      <span class="custom-tree-node"
            slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level <= 2"
                     type="text"
                     size="mini"
                     @click="() => append(data)">
            Append
          </el-button>
          <el-button v-if="node.childNodes.length == 0"
                     type="text"
                     size="mini"
                     @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <div>
      <el-button type="text"
                 @click="dialogVisible = true">点击打开 Dialog</el-button>
      <el-dialog title="提示"
                 :visible.sync="dialogVisible"
                 width="30%">

        <el-form :model="category">
          <el-form-item label="活动名称">
            <el-input v-model="category.name"
                      autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer"
             class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary"
                     @click="addCategory()">确 定</el-button>
        </div>
      </el-dialog>
    </div>
  </div>
</template>
<script>
export default {
  data () {
    return {
      category: {},
      dialogVisible: false,
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  created () {
  },
  computed: {
  },
  methods: {
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({ data }) => {
        console.log('成功获取菜单数据', data.parentMenuList)
        this.menus = data.parentMenuList
      })
    },
    // 添加菜单
    append (data) {
      console.log('append', data)
      this.dialogVisible = true
      this.category.parentCid = data.catId
    },

    addCategory () {
      this.dialogVisible = false
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        console.log(data)
        // const newChild = { id: id++, label: 'testtest', children: [] }
        // if (!data.children) {
        //   this.$set(data, 'children', [])
        // }
        // data.children.push(newChild)

        this.$message({
          message: '添加成功!',
          type: 'success',
          duration: 1500
        })
      })
    },
    // 移除菜单
    remove (node, data) {
      this.$confirm(`是否删除【${data.name}】菜单？`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        var ids = [data.catId]
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          if (data && data.code === 0) {
            const parent = node.parent
            const children = parent.data.children || parent.data
            const index = children.findIndex(d => d.id === data.id)
            children.splice(index, 1)
            this.$message({
              message: '删除成功!',
              type: 'success',
              duration: 1500
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    }
  },
  created () {
    this.getDataList()
  }
}
</script>
<style lang="scss" scoped>
</style>
