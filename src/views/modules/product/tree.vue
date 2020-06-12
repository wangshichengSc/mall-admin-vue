<template>
  <div>


    <el-tree
      :props="props"
      :data="data"
      node-key="id"
      show-checkbox
      default-expand-all
      @check-change="handleCheckChange"
      :expand-on-click-node="false">
    <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <=2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >Append</el-button>
          <el-button type="text" size="mini" @click="edit(data)">edit</el-button>
          <el-button
            v-if="node.childNodes.length==0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >Delete</el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog title="增加球员" :visible.sync="dialogFormVisible"
               :before-close="cancel">
      <el-form :model="category">
        <el-form-item label="商品名称" :label-width="formLabelWidth">
          <el-input v-model="category.name" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="商品图标" :label-width="formLabelWidth">
          <el-input v-model="category.icon" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位" :label-width="formLabelWidth">
          <el-input v-model="category.productUnit" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="商品数量" :label-width="formLabelWidth">
          <el-input v-model="category.productCount" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="是否显示" :label-width="formLabelWidth">
          <el-radio-group v-model="category.showStatus">
            <el-radio :label=0>否</el-radio>
            <el-radio :label=1>是</el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancel()">取 消</el-button>
        <el-button type="primary" @click="commit()">确 定</el-button>
      </div>
    </el-dialog>

    <el-dialog
      title="提示"
      :visible.sync="centerDialogVisible"
      width="30%"
      :before-close="cancel"
      center>
      <span>{{showContext}}</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="cancel()">取 消</el-button>
        <el-button type="primary" @click="doRemove()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>

  export default {
    data() {
      return {
        data: [],
        props: {
          label: 'name',
          children: 'children'
        },
        dialogFormVisible: false,
        centerDialogVisible: false,
        formLabelWidth: '120px',
        category: {
          name: '',
          parentCid: 0,
          catLevel: 1,
          showStatus: 1,
          sort: 0,
          icon: '',
          productUnit: null,
          productCount: null
        },
        doType: '',
        showContext: '',
      };
    },
    methods: {
      handleCheckChange(data, checked, indeterminate) {
        console.log(data, checked, indeterminate);
      },
      handleNodeClick(data) {
        console.log(data);
      },
      getTree() {
        this.$http({
          url: this.$http.adornUrl('/product/category/list'),
          method: 'get'
        }).then(({data}) => {
          console.log(data);
          if (data && data.data) {
            this.data = data.data
          }
        })
      },
      append(data) {
        // console.log("append", data);
        this.category.parentCid = data.catId;
        this.category.catLevel = data.catLevel + 1;
        this.dialogFormVisible = true;
        this.doType = 'add';
      },
      edit(data) {
        this.category = data;
        this.dialogFormVisible = true;
        this.doType = 'edit';
      },
      remove(node, data) {
        this.showContext = '确认要把 ' + data.name + ' 删除吗？';
        this.centerDialogVisible = true;
        this.category = data;
      },
      commit() {
        let url = "";
        if (this.doType === 'edit') {
          url = this.$http.adornUrl('/product/category/update');
        } else {
          url = this.$http.adornUrl('/product/category/save');
        }
        this.$http({
          url: url,
          method: 'post',
          data: JSON.stringify(this.category)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dialogFormVisible = false;
            this.getTree();
            this.category = {};
          }
        })
      },
      doRemove() {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: [this.category.catId]
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.centerDialogVisible = false;
            this.getTree();
          }
        })
      },
      cancel() {
        this.centerDialogVisible = false;
        this.dialogFormVisible = false;
        this.category = {};
      }
    },
    created: function () {
      this.getTree();
    },

  };
</script>

<style scoped>
  .custom-tree-node {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 14px;
    padding-right: 8px;
  }
</style>
