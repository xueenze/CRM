<template>
  <el-dialog title="客户成交状态"
             v-loading="loading"
             :visible="visible"
             @close="handleCancel"
             :append-to-body="true"
             width="400px">
    <div class="handle-box">
      <flexbox class="handle-item"
               align="stretch">
        <div class="handle-item-name"
             style="margin-top: 8px;">成交状态：</div>
        <el-select v-model="status"
                   class="handle-item-content"
                   placeholder="请选择">
          <el-option v-for="item in options"
                     :key="item.value"
                     :label="item.label"
                     :value="item.value">
          </el-option>
        </el-select>
      </flexbox>
    </div>
    <span slot="footer"
          class="dialog-footer">
      <el-button @click.native="handleCancel">取消</el-button>
      <el-button type="primary"
                 @click.native="handleConfirm">保存</el-button>
    </span>
  </el-dialog>
</template>

<script>
import { crmCustomerDealStatusAPI } from '@/api/customermanagement/customer'

export default {
  /** 客户管理  成交状态 操作*/
  name: 'deal-status-handle',
  components: {},
  mixins: [],
  watch: {},
  props: {
    visible: {
      type: Boolean,
      required: true,
      default: false
    },
    crmType: {
      type: String,
      default: ''
    },
    // 勾选数据
    selectionList: {
      type: Array,
      default: () => {
        return []
      }
    }
  },
  data() {
    return {
      loading: true,
      status: 1,
      options: [
        {
          label: '已成交',
          value: 1
        },
        {
          label: '未成交',
          value: 2
        }
      ]
    }
  },
  computed: {},
  mounted() {},
  methods: {
    /**
     * 取消选择
     */
    handleCancel() {
      this.$emit('update:visible', false)
    },

    /**
     * 点击确定
     */
    handleConfirm() {
      var self = this
      var action_ids = this.selectionList.map(function(item, index, array) {
        return item[self.crmType + '_id']
      })
      var params = {
        status: this.status
      }
      params[this.crmType + '_id'] = action_ids
      this.loading = true
      crmCustomerDealStatusAPI(params)
        .then(res => {
          this.$message({
            type: 'success',
            message: res.data
          })
          this.loading = false
          this.$emit('handle', {
            type: 'deal_status'
          })
          this.handleCancel()
        })
        .catch(() => {
          this.loading = false
        })
    }
  }
}
</script>

<style lang="scss" scoped>
.handle-box {
  color: #333;
  font-size: 12px;
}
.handle-item {
  padding-bottom: 15px;
  .handle-item-name {
    flex-shrink: 0;
    width: 90px;
  }
  .handle-item-content {
    flex: 1;
  }
}
.handle-bar {
  position: relative;
  margin-top: 10px;
  height: 30px;
  button {
    float: right;
    margin-right: 10px;
  }
}
</style>
