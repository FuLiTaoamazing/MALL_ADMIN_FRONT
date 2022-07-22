<template>
  <div>
    <el-tree
      :data="childrens"
      :props="defaultProps"
      node-key="catId"
      :expand-on-click-node="false"
      show-checkbox
      :default-expanded-keys="expandKey"
      draggable
      :allow-drop="allowDrop"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            增加子节点
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">
            修改
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            删除子节点
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="flag"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button
          v-if="dialogType == 'add'"
          type="primary"
          @click="() => subitCategory()"
          >确 定</el-button
        >
        <el-button
          v-if="dialogType == 'edit'"
          type="primary"
          @click="() => editCategory()"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      title: "",
      category: {
        catId: "",
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: "",
        productUnit: "",
      },
      flag: false,
      childrens: [],
      expandKey: [],
      dialogVisible: false,
      dialogType: "",
      defaultProps: {
        children: "children",
        label: "name",
      },
      maxDeepLevel:-1
    };
  },
  methods: {
    getChirends() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        this.childrens = data.data;
      });
    },
    append(data) {
      this.title = "添加分类";
      this.dialogVisible = true;
      this.dialogType = "add";
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      if (data.children.length != 0) {
        this.category.sort =
          data.children[data.children.length - 1].sort * 1 + 1;
      }
    },
    edit(curNode) {
      this.title = `修改分类`;
      this.dialogVisible = true;
      this.dialogType = "edit";
      //从服务器拿到最新的数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${curNode.catId}`),
        method: "get",
      }).then(({ data }) => {
        let onlineCurCategory = data.category;
        this.category.name = onlineCurCategory.name;
        this.category.catId = onlineCurCategory.catId;
        this.category.icon = onlineCurCategory.icon;
        this.category.productUnit = onlineCurCategory.productUnit;
      });
    },
    remove(node, data) {
      this.$confirm(
        `此操作将永久删除【${data.name}】该节点, 是否继续?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      )
        .then(() => {
          var ids = [data.catId];
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            this.$message({
              type: "success",
              message: "删除成功!",
            });
            this.expandKey.push(node.parent.data.catId);
            this.getChirends();
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    subitCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "保存成功!",
        });
        this.expandKey.push(this.category.parentCid);
        this.getChirends();
        this.dialogVisible = false;
      });
    },
    editCategory() {
      let { catId, name, icon, productUnit } = this.category;
      let dto = { catId: catId, name: name, icon: icon, productUnit };
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(dto, false),
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "保存成功!",
        });
        this.category.catId = "";
        this.category.name = "";
        this.category.icon = "";
        this.category.productUnit = "";
        this.expandKey.push(dto.catId);
        this.getChirends();
        this.dialogVisible = false;
      });
    }, //判斷節點是否能拖拽
    allowDrop(draggingNode, dropNode, type) {
        //1、被拖動的当前节点以及父节点的层数的和不能大于3
        let curNodeLevel=draggingNode.data.catLevel;

    },
    curNoodeDeep(curNode){
      //找到当前节点的最大深度
      if(curNode.children!=null&&curNode.children.length>0){
        for(let i;i<curNode.children.length;i++){
            if(curNode.children[i].catLevel>this.maxDeepLevel){
              this.maxDeepLevel=curNode.children[i].catLevel
            }
            this.curNoodeDeep(curNode.children[i])
        }
      }
    }
  },
  created() {
    this.getChirends();
  },
};
</script>

<style>
</style>