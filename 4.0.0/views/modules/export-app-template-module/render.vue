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
  form,
  uploadImgRef,
  mockDataModal,
  visible,
  close,
  model,
  valid,
  changeIcon,
  canEditIcon,
  validateRules,
  exported,
  onStart,
  onEnd,
  checkPermission,
  openAssetconfig,
  taglist,
  onSelectTags,
  lastVersion,
  files,
  onUploadSuccess,
  needPreview,
  showMockModal,
  onSubmit,
  submiting,
  currentRole,
  getMockList,
  setValid,
} = inject('$context');

console.log('form', form);
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
