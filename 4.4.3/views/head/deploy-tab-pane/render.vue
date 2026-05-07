<template>
  <div v-bind="attrs" :class="$style.root">
    <div :class="$style.container" v-if="deployRecords.length">
      <div :class="$style.header">
        <el-button type="primary" @click="handleDeploy">远程部署</el-button>
      </div>
      <el-table
        :data="deployRecords"
        key="deployRecordsTable"
        :loading="listLoading"
        :class="$style.deployRecordsTable"
        :height="360">
        <el-table-column prop="ideVersion" label="IDE版本" width="100" show-overflow-tooltip>
          <template #default="scope">
            <template v-if="scope.$index === -1" />
            <span v-else>{{ scope.row.ideVersion || '-' }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="branchName" label="所属分支" width="100" show-overflow-tooltip>
          <template #default="scope">
            <template v-if="scope.$index === -1" />
            <span v-else>{{ scope.row.branchName || '-' }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="status" label="部署状态" width="100" show-overflow-tooltip>
          <template #default="{ row: item, $index }">
            <template v-if="$index === -1" />
            <template v-else>
              <div v-if="item.message === '进行中'" :class="$style.loading"></div>
              <s-others-icon v-if="item.message === '部署失败'" name="fail" :class="$style.statusicon"></s-others-icon>
              <s-others-icon v-if="item.message === '部署成功'" name="primary" :class="$style.statusicon"></s-others-icon>
              {{ item.message }}
            </template>
          </template>
        </el-table-column>
        <el-table-column prop="createdBy" label="部署人" width="100" show-overflow-tooltip>
          <template #default="scope">
            <template v-if="scope.$index === -1" />
            <span v-else>{{ scope.row.createdBy || '-' }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="createdTime" label="部署时间" width="200" show-overflow-tooltip>
          <template #default="scope">
            <template v-if="scope.$index === -1" />
            <span v-else>{{ scope.row.createdTime || '-' }}</span>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template #default="scope">
            <template v-if="scope.$index === -1" />
            <div v-else>
              <el-link @click="viewLog(scope.row)" style="margin-right: 10px">日志</el-link>
              <el-link @click="remove(scope.row)" style="margin-right: 10px">删除</el-link>
            </div>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        v-bind="tableProps.pagination"
        :current-page="tableProps.pageNo"
        :total="tableProps.count"
        :page-size="tableProps.pageSize"
        @current-change="changePage"
        class="s-table-pagination"
      ></el-pagination>
    </div>
    <div v-else :class="$style.emptyContent">
      <div :class="$style.emptyInner">
        <p>没有远程部署记录</p>
        <el-button style="width: 100%;" type="primary" @click="handleDeploy">远程部署</el-button>
      </div>
    </div>
  </div>
  <s-upgrade-modal ref="upgradeModal" />
</template>

<script setup>
import { inject, useAttrs } from 'vue';

const attrs = useAttrs();

const {
  /**
   * 远程部署记录列表
   *
   * @type {Vue.ref<Array>}
   */
  deployRecords,
  /**
   * 列表加载中状态
   *
   * @type {Vue.ref<boolean>}
   */
  listLoading,
  /**
   * 分页及表格属性
   *
   * @type {Vue.ref<object>}
   */
  tableProps,
  /**
   * 升级弹窗实例
   *
   * @type {Vue.ref<any>}
   */
  upgradeModal,
  /**
   * 当前时间
   *
   * @type {Vue.ref<number>}
   */
  currentDate,
  /**
   * 是否支持导出源码/远程部署
   *
   * @type {Vue.computed<boolean>}
   */
  exportSourceSupport,
  /**
   * 切换页码
   *
   * @function
   */
  changePage,
  /**
   * 远程部署操作
   *
   * @function
   */
  handleDeploy,
  /**
   * 查看日志
   *
   * @function
   */
  viewLog,
  /**
   * 删除记录
   *
   * @function
   */
  remove,
} = inject('$context');
</script>

<style module>
.root {
  height: 458px;
  overflow: auto;
}

.header {
  padding: 0 0 15px;
  text-align: right;
}

.emptyContent {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
}

.emptyInner {
  display: inline-block;
  transform: translateY(-50%);
}

.emptyInner p {
  text-align: center;
  color: #999;
}

.iconInfo {
  font-size: 16px;
  vertical-align: top;
  color: #666;
}

.statusicon svg {
  cursor: pointer;
  width: 16px;
  height: 16px;
  vertical-align: sub;
}

.s-table-pagination {
  justify-content: flex-end;
  --pagination-item-background-selected: var(--fill-color-active);
  --pagination-item-color-selected: var(--text-color-primary);
  margin-top: 10px;
}

.s-table-pagination .el-pager li.is-active {
  background-color: var(--pagination-item-background-selected);
  color: var(--pagination-item-color-selected);
  border-radius: 4px;
}
.s-table-pagination .el-pager li + li {
  margin-left: 4px;
}

.loading {
  display: inline-block;
  position: relative;
  width: 16px;
  height: 16px;
  vertical-align: sub;
  border: 2px solid #337EFF;
  border-top-color: rgba(0, 0, 0, 0.2);
  border-right-color: #00000033;
  border-bottom-color: rgba(0, 0, 0, 0.2);
  border-radius: 100%;
  animation: circle infinite 0.75s linear;
}

@keyframes circle {
  0% {
    transform: rotate(0);
  }

  100% {
    transform: rotate(360deg);
  }
}
</style>

