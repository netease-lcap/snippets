<template>
  <el-dialog
    v-model="visible"
    title="应用日志"
    :class="$style.modal"
    destroy-on-close
    append-to-body
    :close-on-click-modal="false"
    @close="close"
  >
    <s-log
      :events="events"
      :flowLogs="graph"
      :exportStatus="exportStatus"
      :jobId="jobId"
      :isRemoteDeploy="false"
    />
  </el-dialog>
</template>

<script setup>
import { inject } from 'vue';

const {
  /**
   * 弹窗是否可见
   *
   * @type {Vue.ref<boolean>}
   */
  visible,
  /**
   * 日志事件列表
   *
   * @type {Vue.ref<Array>}
   */
  events,
  /**
   * 任务 ID
   *
   * @type {Vue.ref<string>}
   */
  jobId,
  /**
   * 导出状态
   *
   * @type {Vue.ref<string>}
   */
  exportStatus,
  /**
   * 流程图数据
   *
   * @type {Vue.ref<any>}
   */
  graph,
  /**
   * 关闭弹窗并通知父组件
   *
   * @function
   */
  close,
} = inject('$context');
</script>

<style module>
.modal {
  width: 1000px;
  height: 490px;
}

.modal div[class^='el-dialog__body'] {
  margin: 0;
}
</style>
