<template>
  <slide-view v-empty="!canShowDetail"
              xs-empty-icon="nopermission"
              xs-empty-text="暂无权限"
              :listenerIDs="listenerIDs"
              :noListenerIDs="noListenerIDs"
              :noListenerClass="noListenerClass"
              @side-close="hideView"
              :body-style="{padding: 0, height: '100%'}">
    <flexbox v-if="canShowDetail"
             v-loading="loading"
             direction="column"
             align="stretch"
             class="d-container">
      <c-r-m-detail-head crmType="leads"
                         @handle="detailHeadHandle"
                         @close="hideView"
                         :detail="detailData"
                         :headDetails="headDetails"
                         :id="id">
      </c-r-m-detail-head>
      <div class="tabs">
        <el-tabs v-model="tabCurrentName"
                 @tab-click="handleClick">
          <el-tab-pane v-for="(item, index) in tabnames"
                       :key="index"
                       :label="item.label"
                       :name="item.name">
          </el-tab-pane>
        </el-tabs>
      </div>
      <div class="t-loading-content"
           id="follow-log-content">
        <keep-alive>
          <component v-bind:is="tabName"
                     crmType="leads"
                     :id="id"></component>
        </keep-alive>
      </div>
    </flexbox>
    <c-r-m-create-view v-if="isCreate"
                       crm-type="leads"
                       :action="{type: 'update', id: this.id}"
                       @save-success="editSaveSuccess"
                       @hiden-view="isCreate=false"></c-r-m-create-view>
  </slide-view>
</template>

<script>
import { crmLeadsRead } from '@/api/customermanagement/clue'

import SlideView from '@/components/SlideView'
import CRMDetailHead from '../components/CRMDetailHead'
import ClueFollow from './components/ClueFollow' // 跟进记录
import CRMBaseInfo from '../components/CRMBaseInfo' // 线索基本信息
import RelativeFiles from '../components/RelativeFiles' //相关附件
import RelativeHandle from '../components/RelativeHandle' //相关操作

import CRMCreateView from '../components/CRMCreateView' // 新建页面

import moment from 'moment'
import detail from '../mixins/detail'

export default {
  /** 线索管理 的 线索详情 */
  name: 'clue-detail',
  components: {
    SlideView,
    CRMDetailHead,
    ClueFollow,
    CRMBaseInfo,
    RelativeFiles,
    RelativeHandle,
    CRMCreateView
  },
  mixins: [detail],
  props: {
    // 详情信息id
    id: [String, Number],
    // 监听的dom 进行隐藏详情
    listenerIDs: {
      type: Array,
      default: () => {
        return ['crm-main-container']
      }
    },
    // 不监听
    noListenerIDs: {
      type: Array,
      default: () => {
        return []
      }
    },
    noListenerClass: {
      type: Array,
      default: () => {
        return ['el-table__body']
      }
    }
  },
  data() {
    return {
      loading: false, // 展示加载loading
      crmType: 'leads',
      detailData: {}, // read 详情
      headDetails: [
        { title: '线索来源', value: '' },
        { title: '手机', value: '' },
        { title: '负责人', value: '' },
        { title: '更新时间', value: '' }
      ],
      tabnames: [
        { label: '跟进记录', name: 'followlog' },
        { label: '基本信息', name: 'basicinfo' },
        { label: '附件', name: 'file' },
        { label: '操作记录', name: 'operationlog' }
      ],
      tabCurrentName: 'followlog',
      isCreate: false // 编辑操作
    }
  },
  computed: {
    tabName() {
      if (this.tabCurrentName == 'followlog') {
        return 'clue-follow'
      } else if (this.tabCurrentName == 'basicinfo') {
        return 'c-r-m-base-info'
      } else if (this.tabCurrentName == 'file') {
        return 'relative-files'
      } else if (this.tabCurrentName == 'operationlog') {
        return 'relative-handle'
      }
      return ''
    }
  },
  mounted() {},
  methods: {
    getDetial() {
      this.loading = true
      crmLeadsRead({
        id: this.id
      })
        .then(res => {
          this.detailData = res.data

          this.headDetails[0].value = res.data.source
          this.headDetails[1].value = res.data.mobile
          // 负责人
          this.headDetails[2].value = res.data.owner_user_id_info
            ? res.data.owner_user_id_info.realname
            : ''
          this.headDetails[3].value = res.data.update_time
          this.loading = false
        })
        .catch(err => {
          if (err && err.code == 102) {
            this.hasRequestAuth = false
          }
          this.loading = false
        })
    },
    //** 点击关闭按钮隐藏视图 */
    hideView() {
      this.$emit('hide-view')
    },
    //** tab标签点击 */
    handleClick(tab, event) {},
    editSaveSuccess() {
      this.$emit('handle', { type: 'save-success' })
      this.getDetial()
    }
  }
}
</script>

<style lang="scss" scoped>
@import '../styles/crmdetail.scss';
</style>
