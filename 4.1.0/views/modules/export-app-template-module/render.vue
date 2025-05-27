<template>
  <el-dialog
    v-model="visible"
    class="s-export-app-template-modal"
    width="500px"
    align-center
    destroy-on-close
    append-to-body
    :close-on-click-modal="false"
    @close="close()"
  >
        <template #header>
            <span style="margin-right: 8px">发布为应用模板</span>
        </template>
        <el-form ref="form" :model="model" @validate="setValid($event.valid)"  label-position="top" require-asterisk-position="right" :class="$style.form">
              <div :class="$style.iconbox">
                    <s-upload-img ref="uploadImgRef" @changeIcon="changeIcon" :can-edit="canEditIcon" :prop-icon="model.icon"> </s-upload-img>
                    <el-form-item label="模板名称"
                        style="width:100%;margin-left: 10px;" required :rules="validateRules.name"  prop="name" for="none">
                        <el-input
                            v-model="model.name"
                            placeholder="请输入模板名称"
                            :disabled="exported"
                            @compositionstart="onStart"
                            @compositionend="onEnd"
                        ></el-input>
                    </el-form-item>
                </div>

            <el-form-item label="模板分类(资产中心展示)" :class="$style.longlabel" style="position: relative;" layout="block" prop="tags" for="none">
                <div :class="$style.tagcategory">
                    <el-tooltip v-if="!checkPermission" :content="'暂无操作权限'" trigger="hover" placement="top" :hover-delay="500">
                        <el-link :disabled="!checkPermission" @click="openAssetconfig">分类管理
                            <s-others-icon name="check-detail" :class="$style.toDetail"></s-others-icon>
                        </el-link>
                    </el-tooltip>
                    <el-link v-if="checkPermission" @click="openAssetconfig">分类管理
                        <s-others-icon name="check-detail" :class="$style.toDetail"></s-others-icon>
                    </el-link>
                </div>
                    <el-tree-select
                    class="treetag"
                    placeholder="请选择模板分类"
                    v-model="model.tags"
                    style="width: 100%"
                    :data="taglist"
                    multiple
                    :render-after-expand="false"
                    show-checkbox
                    node-key="id"
                    :default-expanded-keys="['全部分类']"
                    @change="onSelectTags"
                    :props="{
                      children: 'child',
                      label: 'tag',
                      disabled:'disable'
                    }"
                  />
                </el-form-item>
            <el-form-item label="发布版本" required :rules="validateRules.checkVersion" layout="block" prop="version">
                <span :class="$style.lastVersionTitle"
                >(上次导出的版本：{{ lastVersion ? lastVersion : '暂无' }} )</span>
                <s-version v-model:value="model.version" min-value="1.0.0"></s-version>
            </el-form-item>

            <el-form-item label="版本描述"  :rules="validateRules.description" prop="description" for="none">
                <el-input type="textarea" v-model="model.description" placeholder="请输入"></el-input>
            </el-form-item>


            <el-form-item label="模板配图" layout="block" for="none">
              <s-uploader
                v-model:file-list="files"
                multiple
                list-type="picture"
                drag :on-success="onUploadSuccess"
                action="/api/v1/userIcon/upload"
                accept="image/png, image/jpg, image/jpeg, image/gif, image/bmp"
                :limit="10"
                :class="$style.upload"
              >
                <div :class="$style.uploadArea">
                  <s-others-icon name="upload" :class="$style.uploadIcon"></s-others-icon>
                  点击/拖拽/粘贴文件到这里
                </div>
              </s-uploader>
            </el-form-item>
            <el-form-item label="下载选择：" :class="$style.formlabelinline" for="none">
                <el-radio-group v-model="needPreview">
                    <el-radio :value="true">下载模板+资产中心预览效果</el-radio>
                    <el-radio :value="false">仅下载模板</el-radio>
                </el-radio-group>
            </el-form-item>
        </el-form>
        <template #footer>
            <el-space style="display:flex;justify-content:space-between" justify="end">
                <el-space>
                    <el-button @click="showMockModal()" v-if="needPreview">编辑预览数据</el-button>
                </el-space>

          <el-space justify="end">
            <el-button @click="close()">取 消</el-button>
            <el-button type="primary" @click="onSubmit" :loading="submiting">导 出</el-button>
          </el-space>
          </el-space>
        </template>
        <skeleton-render
            name="views/modules/edit-mock-data-modal/index"
            @currentRole="currentRole"
            @getMockList="getMockList"
            ref="mockDataModal"
        />
  </el-dialog>
</template>

<script setup>
import { inject } from 'vue';

const {
  /**
   * 表单节点
   * 用于表单验证和提交操作
   * 
   * @type {Vue.ref<HTMLElement>}
   */
  formRef,

  /**
   * 树形视图节点
   * 用于树形数据的展示和操作
   * 
   * @type {Vue.ref<HTMLElement>}
   */
  treeViewRef,

  /**
   * 对话框是否可见
   * 控制导出模板对话框的显示状态
   * 
   * @type {Vue.ref<boolean>}
   */
  visible,

  /**
   * 是否处于加载状态
   * 控制数据加载时的loading状态
   * 
   * @type {Vue.ref<boolean>}
   */
  loading,

  /**
   * 是否为空数据状态
   * 标识当前是否有可导出的数据
   * 
   * @type {Vue.ref<boolean>}
   */
  empty,

  /**
   * 是否处于提交状态
   * 控制表单提交时的loading状态
   * 
   * @type {Vue.ref<boolean>}
   */
  submiting,

  /**
   * 是否编辑旧模板
   * 标识当前是否为编辑已有模板
   * 
   * @type {Vue.ref<boolean>}
   */
  isEditOldTemp,

  /**
   * 是否可以编辑图标
   * 控制模板图标的编辑权限
   * 
   * @type {Vue.ref<boolean>}
   */
  canEditIcon,

  /**
   * 是否自动勾选依赖项
   * 控制是否自动选择相关依赖
   * 
   * @type {Vue.ref<boolean>}
   */
  autoCheckdepends,

  /**
   * 是否为空选中状态
   * 标识是否未选择任何导出项
   * 
   * @type {Vue.ref<boolean>}
   */
  isEmptyChecked,

  /**
   * 是否有预选节点
   * 标识是否存在预先选中的节点
   * 
   * @type {Vue.ref<boolean>}
   */
  hasPreCheckedNodes,

  /**
   * 是否有权限
   * 控制用户的操作权限
   * 
   * @type {Vue.computed<boolean>}
   */
  checkPermission,

  /**
   * 表单数据对象
   * 存储模板的基本信息
   * 
   * @type {Vue.reactive<object>}
   * @property {string} icon - 模板图标
   * @property {array} picture - 模板配图列表
   * @property {array} tags - 模板分类标签
   * @property {string} symbol - 模板标识
   * @property {string} name - 模板名称
   * @property {string} version - 模板版本
   * @property {string} description - 模板描述
   */
  model,

  /**
   * 表单验证规则
   * 定义表单字段的验证规则
   * 
   * @type {object}
   */
  validateRules,

  /**
   * 标签列表数据
   * 存储可选的模板分类标签
   * 
   * @type {Vue.ref<array>}
   */
  taglist,

  /**
   * 历史记录列表
   * 存储模板的历史版本记录
   * 
   * @type {Vue.ref<array>}
   */
  historyList,

  /**
   * 历史编辑列表
   * 存储可编辑的历史版本列表
   * 
   * @type {Vue.ref<array>}
   */
  historyEditList,

  /**
   * 最后一项数据
   * 存储当前选中的历史记录
   * 
   * @type {Vue.reactive<object>}
   * @property {string} name - 模板名称
   * @property {string} symbol - 模板标识
   * @property {string} value - 模板值
   * @property {string} text - 显示文本
   */
  lastItem,

  /**
   * 最后版本号
   * 存储模板的最新版本号
   * 
   * @type {Vue.ref<string>}
   */
  lastVersion,

  /**
   * 树形数据
   * 存储可导出的能力树形结构
   * 
   * @type {Vue.ref<array>}
   */
  treeData,

  /**
   * 树形属性配置
   * 定义树形组件的属性配置
   * 
   * @type {object}
   */
  treeProps,

  /**
   * 默认展开的节点键值
   * 控制树形组件默认展开的节点
   * 
   * @type {Vue.ref<array>}
   */
  defaultExpandedKeys,

  /**
   * 选中的节点值
   * 存储当前选中的节点
   * 
   * @type {Vue.ref<array>}
   */
  checkedValues,

  /**
   * 关闭对话框
   * 关闭导出模板对话框
   * 
   * @function
   */
  close,

  /**
   * 改变图标
   * 更新模板图标
   * 
   * @function
   * @param {string} item - 新的图标值
   */
  changeIcon,

  /**
   * 编辑为新模板
   * 将当前模板编辑为新模板
   * 
   * @function
   * @param {Event} e - 事件对象
   */
  canEditToNewTemp,

  /**
   * 跳转到资产中心
   * 跳转到资产中心页面
   * 
   * @function
   */
  goAssetCenterSeg,

  /**
   * 打开资产配置
   * 打开资产配置对话框
   * 
   * @function
   */
  openAssetconfig,

  /**
   * 选择模板
   * 选择历史模板版本
   * 
   * @function
   * @param {string} symbol - 模板标识
   */
  onSelectSegmentTemp,

  /**
   * 选择标签
   * 更新模板分类标签
   * 
   * @function
   * @param {array} item - 选中的标签列表
   */
  onSelectTags,

  /**
   * 切换历史记录
   * 切换到指定的历史版本
   * 
   * @function
   * @param {object} item - 历史记录项
   */
  toggleHistory,

  /**
   * 提交表单
   * 提交模板表单数据
   * 
   * @function
   */
  onSubmit,

  /**
   * 检查变更
   * 处理树形节点选中状态变更
   * 
   * @function
   * @param {object} node - 节点对象
   * @param {boolean} checked - 是否选中
   */
  onCheckChange,

  /**
   * 设置目录展开图标
   * 更新目录展开状态的图标
   * 
   * @function
   * @param {object} node - 节点对象
   */
  setDirectoryExpandIcon,

  /**
   * 设置目录折叠图标
   * 更新目录折叠状态的图标
   * 
   * @function
   * @param {object} node - 节点对象
   */
  setDirectoryCollapseIcon,

  /**
   * 获取节点引用
   * 获取节点的依赖引用关系
   * 
   * @function
   * @param {object} node - 节点对象
   * @returns {array} 引用列表
   */
  getNodeReferences,

  /**
   * 获取图标
   * 获取节点对应的图标
   * 
   * @function
   * @param {object} node - 节点对象
   * @returns {string} 图标名称
   */
  getIcon,

  /**
   * 获取文本名称
   * 获取节点的显示文本
   * 
   * @function
   * @param {object} node - 节点对象
   * @returns {string} 文本名称
   */
  getTextName,

  /**
   * 获取选中状态
   * 获取节点的选中状态文本
   * 
   * @function
   * @param {object} node - 节点对象
   * @returns {string} 选中状态
   */
  getCheckedState,

  /**
   * 全选操作
   * 处理节点的全选/取消全选
   * 
   * @function
   * @param {Event} e - 事件对象
   * @param {object} node - 节点对象
   */
  onCheckAll,

  /**
   * 判断是否为图片
   * 判断图标是否为图片类型
   * 
   * @function
   * @param {string} icon - 图标值
   * @param {string} connectorKind - 连接器类型
   * @returns {boolean} 是否为图片
   */
  isImage,

  /**
   * 获取连接器设置
   * 获取连接器的配置信息
   * 
   * @function
   * @param {object} node - 节点对象
   * @returns {object} 连接器设置
   */
  getConnectorSetting,

  /**
   * 获取连接器包装图标
   * 获取连接器的包装图标
   * 
   * @function
   * @param {object} node - 节点对象
   * @returns {string} 图标
   */
  getConnectorWrapperIcon,
} = inject('$context');
</script>

<style module>
.form {
    padding: 8px;
}


.tagcategory {
    position: absolute;
    top: -28px;
    right: 0px;
    white-space: nowrap;
}

.iconbox {
    display: flex;
    justify-content: space-between;
/*    margin-bottom:20px;*/
}
.upload {
  width: 100%;
}
.upload [class^='el-upload-dragger'] {
  background-color: var(--uploader-draggable-background);
  padding: 19px;
}
.uploadArea {
  color: var(--uploader-draggable-color);
  pointer-events: none;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}

.uploadIcon {
  font-size: 24px;
}
.lastVersionTitle {
  position: absolute;
  top: -35px;
  left: 70px;
  color: #666;
}
.formlabelinline {
    display: flex !important;
    margin-bottom: 0;
}
.formlabelinline > [class^='el-form-item__label'] {
    display: flex !important;
    align-items: center;
    margin-bottom: 0 !important;
}
</style>
