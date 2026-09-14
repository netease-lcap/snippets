<template>
  <div>
    <el-checkbox v-if="showChangeRepoGit" v-model="model.changeRepoGit">自定义git凭证</el-checkbox>
    <template v-if="model.changeRepoGit">
      <el-form-item
        :class="$style.item"
        prop="credentialId"
        :rules="[
          {
            required: true,
            message: 'git凭证必选',
            trigger: ['blur', 'change'],
          },
        ]"
      >
        <el-select ref="selectRef" v-model="model.credentialId">
          <el-option
            v-for="certItem in certList"
            :key="certItem.value"
            :value="certItem.value"
            :label="certItem.label"
          >
            <div :class="$style.optionItem">
              <span>{{ certItem.label }}</span>
              <div :class="$style.optionActions" @click.stop>
                <el-link
                  :class="$style.actionBtn"
                  type="primary"
                  :underline="false"
                  @click="onEdit(certItem)"
                >编辑</el-link>
                <el-link
                  :class="$style.actionBtn"
                  type="danger"
                  :underline="false"
                  @click="onDeleteClick(certItem)"
                >删除</el-link>
              </div>
            </div>
          </el-option>
        </el-select>
        <s-others-icon name="update" @click="getCert()" :class="$style.icon"></s-others-icon>
      </el-form-item>
      <el-link v-if="checkPermission" type="primary" :underline="false" @click="onCreate">
        <s-others-icon name="add3"></s-others-icon>创建凭证
      </el-link>
      <skeleton-render
        name="views/head/export-cert-add/index"
        v-model="showCreate"
        @done="getCert"
      />
      <skeleton-render
        name="views/head/export-cert-edit/index"
        v-model="showEdit"
        :cert-data="editCertData"
        @done="onEditDone"
      />
    </template>
  </div>
</template>

<script setup>
import { inject } from 'vue';

const {
  /**
   * 下拉选择框节点
   * 
   * @type {Vue.ref<HTMLElement>}
   */
  selectRef,
  /**
   * 是否显示创建凭证对话框
   * 
   * @type {Vue.ref<boolean>}
   */
  showCreate,
  /**
   * 是否显示编辑凭证对话框
   * 
   * @type {Vue.ref<boolean>}
   */
  showEdit,
  /**
   * 编辑的凭证数据
   * 
   * @type {Vue.ref<object>}
   */
  editCertData,
  /**
   * 是否有创建凭证权限
   * 
   * @type {Vue.ref<boolean>}
   */
  checkPermission,
  /**
   * 凭证列表
   * 
   * @type {Vue.ref<array>}
   */
  certList,
  /**
   * 是否显示自定义git凭证选项
   * 
   * @type {Vue.ref<boolean>}
   */
  showChangeRepoGit,
  /**
   * 表单数据模型
   * 
   * @type {Vue.ref<object>}
   * @property {boolean} changeRepoGit - 是否自定义git凭证
   * @property {string} credentialId - 凭证ID
   */
  model,
  /**
   * 创建凭证
   * 
   * @function
   */
  onCreate,
  /**
   * 编辑凭证
   * 
   * @function
   * @param {object} certItem - 凭证项
   */
  onEdit,
  /**
   * 删除凭证点击事件
   * 
   * @function
   * @param {object} certItem - 凭证项
   */
  onDeleteClick,
  /**
   * 获取凭证列表
   * 
   * @function
   * @param {string|number} defaultId - 默认凭证ID
   */
  getCert,
  /**
   * 编辑完成回调
   * 
   * @function
   */
  onEditDone,
} = inject('$context') || {};
</script>

<style module>
.item {
  display: flex;
  margin: 5px 0;
}

.item :global(.el-form-item__content) {
  flex-wrap: nowrap;
}

.icon {
  cursor: pointer;
  margin-left: 4px;
  font-size: 14px;
  display: inline-flex;
  align-items: center;
}

.optionItem {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
}

.optionItem:hover .optionActions {
  display: flex;
}

.optionActions {
  display: none;
  gap: 8px;
  align-items: center;
}

.actionBtn {
  padding: 0 4px;
  font-size: 12px;
}

/* 删除按钮保持 danger 颜色 */
.actionBtn:global(.el-link--danger),
.actionBtn:global(.el-link--danger:hover),
.actionBtn:global(.el-link--danger:focus),
.actionBtn:global(.el-link--danger:active),
.actionBtn:global(.el-link--danger.is-disabled) {
  color: var(--el-color-danger) !important;
}

.actionBtn:global(.el-link--danger .el-link__inner:hover) {
  color: var(--el-color-danger) !important;
}
</style>

