<template>
  <el-dialog :visible.sync="visible"
             width="700px"
             :append-to-body="true"
             :before-close="handleClose">
    <div class="position-relative">
      <el-table :data="tableData"
                border
                style="width: 100%">
        <el-table-column prop="name"
                         label="功能"
                         width="180">
        </el-table-column>
        <el-table-column prop="des"
                         label="功能描述">
        </el-table-column>
      </el-table>
      <div class="switch">
        <el-switch v-model="callSwitch"
                   disabled>
        </el-switch>
        未启用
      </div>
    </div>
    <span slot="footer"
          class="dialog-footer">
      <el-button type="primary"
                 @click="handleClose">确 定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  name: 'call-detail',
  components: {},
  data() {
    return {
      tableData: [
        {
          name: '一键外呼',
          des: '在系统中点击拨号按钮就可以实现一键外呼。'
        },
        {
          name: '呼入弹屏',
          des: '客户来电，自动匹配系统客户，一键查看客户信息。'
        },
        {
          name: '通话录音',
          des: '通话自动录音，支持在线播放和下载。'
        },
        {
          name: '通话记录',
          des: '自动生成通话记录，可以随时查看通话时长、通话时间等信息。'
        }
      ],
      callSwitch: false
    }
  },
  computed: {},
  props: {
    visible: {
      type: Boolean,
      default: false
    }
  },
  mounted() {},
  methods: {
    handleClose() {
      this.$emit('update:visible', false)
    }
  }
}
</script>

<style scoped lang="scss">
.position-relative {
  position: relative;
}

.phone-info {
  margin: 15px 0;
  img {
    width: 16px;
    margin-right: 5px;
  }
}

.switch {
  color: #666666;
  font-size: 13px;
  position: absolute;
  left: 0;
  bottom: -30px;
}
</style>
