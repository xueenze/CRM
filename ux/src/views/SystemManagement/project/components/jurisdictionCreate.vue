<template>
  <el-dialog :title="diaTitle"
             :visible="show"
             v-loading="loading"
             @close="closeView"
             :modal-append-to-body="true"
             :append-to-body="true"
             :close-on-click-modal="false"
             class="create-dialog"
             width="700px">
    <div class="label-input">
      <label class="label-title">权限名称</label>
      <el-input v-model="title"
                placeholder="请输入权限名称"></el-input>
    </div>
    <div class="label-input">
      <label class="label-title">权限描述</label>
      <el-input v-model="remark"
                type="textarea"
                :rows="2"
                placeholder="请输入权限描述"></el-input>
    </div>
    <label class="label-title">权限配置</label>
    <div class="jurisdiction-content-checkbox">
      <el-tree :data="showTreeData"
               show-checkbox
               node-key="id"
               style="height: 0;"
               ref="tree"
               :indent="0"
               empty-text=""
               default-expand-all
               :expand-on-click-node="false"
               :props="defaultProps">
      </el-tree>
    </div>
    <span slot="footer"
          class="dialog-footer">
      <el-button type="primary"
                 @click="sureClick">确 定</el-button>
      <el-button @click="closeView">取 消</el-button>
    </span>
  </el-dialog>
</template>

<script>
import {
  rulesList,
  roleAdd,
  roleUpdate
} from '@/api/systemManagement/RoleAuthorization'

export default {
  name: 'jurisdiction-create', // 文件导入
  components: {},
  data() {
    return {
      loading: false,
      title: '',
      remark: '',
      showTreeData: [],
      defaultProps: {
        children: 'children',
        label: 'title'
      }
    }
  },

  computed: {
    diaTitle() {
      if (this.action.type == 'save') {
        return '新建'
      }

      return '编辑'
    }
  },

  props: {
    show: {
      type: Boolean,
      default: false
    },

    action: {
      type: Object,
      default: () => {
        return {
          type: 'save'
        }
      }
    }
  },

  watch: {
    show(value) {
      if (value) {
        this.initInfo()
      }
    }
  },

  mounted() {},

  methods: {
    /**
     * 初始化数据
     */
    initInfo() {
      if (this.action.type == 'update') {
        this.title = this.action.data.title
        this.remark = this.action.data.remark
      } else {
        this.title = ''
        this.remark = ''
        if (this.$refs.tree) {
          this.$refs.tree.setCheckedKeys([])
        }
      }

      if (this.showTreeData.length == 0) {
        this.getRulesList()
      } else {
        this.checkTreeByUpdateInfo()
      }
    },

    /**
     * 获取规则
     */
    getRulesList() {
      this.loading = true
      rulesList({
        type: 'tree',
        pid: 5
      })
        .then(res => {
          this.showTreeData = res.data
          this.checkTreeByUpdateInfo()
          this.loading = false
        })
        .catch(() => {
          this.loading = false
        })
    },

    /**
     * 标记数据
     */
    checkTreeByUpdateInfo() {
      if (this.action.type == 'update') {
        this.$nextTick(() => {
          if (this.$refs.tree) {
            this.$refs.tree.setCheckedKeys(
              this.getUserModuleRules(this.action.data.rules.split(','))
            )
          }
        })
      }
    },

    /**
     * 确定
     */
    sureClick() {
      if (!this.title) {
        this.$message.error('请填写权限名称')
      } else {
        this.loading = true
        let rules = this.$refs.tree
          .getCheckedKeys()
          .concat(this.$refs.tree.getHalfCheckedKeys())

        let request = {
          save: roleAdd,
          update: roleUpdate
        }[this.action.type]

        let params = {
          type: 'work',
          title: this.title,
          remark: this.remark,
          rules: rules,
          pid: 5
        }

        if (this.action.type == 'update') {
          params.id = this.action.data.id
        }
        request(params)
          .then(res => {
            this.loading = false
            this.$emit('submite')
            this.closeView()
          })
          .catch(() => {
            this.loading = false
          })
      }
    },

    closeView() {
      this.$emit('update:show', false)
    },

    /**
     * 获得check的实际数据
     */
    getUserModuleRules(array) {
      if (!array) {
        array = []
      }
      var firstTree = this.showTreeData[0]
      var hasRemove = false
      var copyArray = this.copyItem(array)
      for (
        let firstIndex = 0;
        firstIndex < firstTree.children.length;
        firstIndex++
      ) {
        const firstItem = firstTree.children[firstIndex]

        if (!firstItem.hasOwnProperty('children')) {
          if (firstItem.length + 1 != copyArray.length) {
            this.removeItem(copyArray, firstTree.id)
          }
          return copyArray
        }

        for (let index = 0; index < array.length; index++) {
          const element = array[index]
          var temps = []
          for (
            let secondIndex = 0;
            secondIndex < firstItem.children.length;
            secondIndex++
          ) {
            const secondItem = firstItem.children[secondIndex]
            if (secondItem.id == element) {
              temps.push(secondItem)
            }
          }
          if (temps.length != firstItem.children.length) {
            hasRemove = true
            this.removeItem(copyArray, firstItem.id)
          }
        }
      }

      if (hasRemove) {
        this.removeItem(copyArray, firstTree.id)
      }

      var checkedKey = []
      for (let index = 0; index < copyArray.length; index++) {
        const element = copyArray[index]
        if (element) {
          checkedKey.push(parseInt(element))
        }
      }
      return checkedKey
    },
    copyItem(array) {
      var temps = []
      for (let index = 0; index < array.length; index++) {
        temps.push(array[index])
      }
      return temps
    },
    removeItem(array, item) {
      var removeIndex = -1
      for (let index = 0; index < array.length; index++) {
        if (item == array[index]) {
          removeIndex = index
          break
        }
      }
      if (removeIndex > 0) {
        array.splice(removeIndex, 1)
      }
    },
    containItem(array, item) {
      for (let index = 0; index < array.length; index++) {
        if (item == array[index]) {
          return true
          break
        }
      }
      return false
    }
  }
}
</script>

<style scoped lang="scss">
.create-dialog /deep/ .el-dialog__body {
  padding-top: 0 !important;
}

.label-title {
  display: block;
  margin: 20px 0 10px 0;
  color: #666;
}

.label-input {
  position: relative;
}

.jurisdiction-content-checkbox {
  border: 1px solid #e6e6e6;
  border-radius: 2px;
  height: 200px;
  overflow-y: auto;
  padding: 10px 0;
}
.jurisdiction-content-checkbox
  .el-tree
  /deep/
  .el-tree-node
  > .el-tree-node__content {
  margin-bottom: 5px;
  width: 150px;
}
.jurisdiction-content-checkbox /deep/ .el-tree .el-tree-node {
  white-space: inherit;
  margin-bottom: 5px;
}
.jurisdiction-content-checkbox
  /deep/
  .el-tree
  > .el-tree-node
  > .el-tree-node__children
  > .is-expanded
  > .el-tree-node__children
  > .is-expanded {
  display: inline-block;
}
</style>
