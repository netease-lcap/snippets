<template>
  <div :class="$style.root">
    <div :class="$style.head">
      <div :class="$style.flexgrid">
        <div :class="$style.gridcolumn" class="extraForm">
          <el-form
            ref="formRef"
            label-width="80px"
            label-position="right"
            hide-required-asterisk
            :model="entityModel"
            :rules="validateRules"
            :class="$style.entityfrom"
            @submit.prevent
            :disabled="isViewEntity"
          >
            <el-form-item
              label="名称"
              prop="name"
              :rules="formNameRules"
            >
              <el-tooltip
                ref="tooltip"
                :disabled="!!entityModel.name"
                :visible="tooltipOpened === 'entity'"
                placement="top-start"
              >
                <template #content> 实体为英文字母、数字和下划线，且首字母大写 </template>
                <el-input
                  ref="entityName"
                  :placeholder="entityPlaceholder"
                  v-model="entityModel.name"
                  @keyup.enter="$refs.entityName.blur()"
                  :class="$style.fixedwidthinput"
                  @focus="tooltipOpened = 'entity'"
                  @blur="tooltipOpened = ''"
                  :disabled="entityList.includes(entityModel.name)"
                ></el-input>
              </el-tooltip>
            </el-form-item>

            <el-form-item label="系统属性">
              <div :class="$style.systemwrap" class="checkBox">
                <el-checkbox
                  @change="setSystemProperty('createdTime', $event)"
                  :class="$style.checkbox"
                  v-model="storageModel.createdTime"
                  :disabled="entityList.includes(entityModel.name) || systemActionState.createdTimeDisabled"
                >
                  存储 createdTime
                </el-checkbox>
                <el-checkbox
                  @change="setSystemProperty('createdBy', $event)"
                  :class="$style.checkbox"
                  v-model="storageModel.createdBy"
                  :disabled="entityList.includes(entityModel.name) ||systemActionState.createdByDisabled"
                >
                  存储 createdBy
                </el-checkbox>
                <el-checkbox
                  @change="setSystemProperty('updatedTime', $event)"
                  :class="$style.checkbox"
                  v-model="storageModel.updatedTime"
                  :disabled="entityList.includes(entityModel.name) ||systemActionState.updatedTimeDisabled"
                >
                  存储 updatedTime
                </el-checkbox>
                <el-checkbox
                  @change="setSystemProperty('updatedBy', $event)"
                  :class="$style.checkbox"
                  v-model="storageModel.updatedBy"
                  :disabled="entityList.includes(entityModel.name) ||systemActionState.updatedByDisabled"
                >
                  存储 updatedBy
                </el-checkbox>
              </div>
            </el-form-item>
          </el-form>
        </div>
        <div :class="$style.gridcolumn">
          <el-form
            ref="form1"
            :model="entityModel"
            label-width="120px"
            label-position="right"
            hide-required-asterisk
            :rules="validateRules"
            :disabled="isViewEntity"
            @submit.prevent
          >
            <el-form-item
              label="数据库表名"
              prop="tableName"
              style="margin-bottom: 16px"
              :rules="nameRules.tableName">
              <el-tooltip
                :disabled="!!entityModel.tableName"
                :visible="tooltipOpened === 'tableName'"
                placement="top-start"
              >
                <template #content> 以字母开头，不能超过60个字符 </template>
                <el-input
                  ref="tableName"
                  placeholder="请输入"
                  maxlength="60"
                  v-model="entityModel.tableName"
                  @keyup.enter="$refs.tableName.blur()"
                  :disabled="canSetTableName"
                  :class="$style.fixedwidthinput"
                  @focus="tooltipOpened = 'tableName'"
                  @blur="tooltipOpened = ''"
                ></el-input>

              </el-tooltip>
              <el-tooltip
                v-if="multiSchema"
                content="若没有填写表模式名称则自动取数据库配置的模式名中的第一个"
                placement="top"
              >
                <s-others-icon name="help_line" class="iconTip"></s-others-icon>
              </el-tooltip>
            </el-form-item>

            <el-form-item :rules="nameRules.description" label="描述" prop="description" class="desTextArea">
              <el-input
                type="textarea"
                v-model="entityModel.description"
                :class="$style.textarea"
                style="height: 115px"
                resize="none"
              ></el-input>
            </el-form-item>
          </el-form>
        </div>
      </div>
    </div>
    <template  v-if="true">
      <div class="tabSelected" :class="$style.body" style="margin-top: 0px ; margin-bottom: 0px;border-bottom:0px ;">
        <el-radio-group v-model="tabSelected" class="s-capsule-tab" flat style="width: 224px">
          <el-radio-button value="property">属性</el-radio-button>
          <el-radio-button v-if="!hideIndexTab" value="index">索引</el-radio-button>
        </el-radio-group>
        <template v-if="tabSelected === 'index'">
          <el-tooltip class="box-item" effect="dark" placement="bottom-end">
            <template #content class="tabSelectIcon">
              <span>查看“</span>
              <el-link @click="dialogVisible = true" type="primary" :line="false"
                >如何创建高性能的索引</el-link
              >
              <span> ”</span>
            </template>
            <s-others-icon name="info" class="info"> </s-others-icon>
          </el-tooltip>
        </template>
      </div>
      <!-- 属性 -->
      <skeleton-render
        name="views/data/entity-property-designer/index"
        v-if="tabSelected === 'property'"
        ref="propertyDesigner"
        :entity="entity"
        @select="$emit('select', $event)"
      />
      <div :class="$style.body" style="margin-top: 0px">
        <!-- 索引 -->
        <skeleton-render
          name="views/data/entity-index-designer/index"
          v-if="tabSelected === 'index'"
          ref="indexDesigner"
          :entity="entity"
          :list="entity.indexes"
        />
        <!-- 弹框 -->
        <el-dialog
          width="564"
          v-model="dialogVisible"
          ref="infoModal"
          title="如何创建高性能索引"
          close-on-press-escape
          align-center
          destroy-on-close
          append-to-body
          :close-on-click-modal="false"
        >
          <ul :class="[$style.info, $style.indexinfo]">
            <li>
              <span :class="$style.tit">什么是索引？</span>
              <div>索引是对数据库表中的一列或多列值进行排序的一种结构，</div>
              <div>使用索引可以快速访问数据库表中的特定信息；</div>
            </li>
            <li>
              <span :class="$style.tit">索引的作用？</span>
              <div>索引相当于图书上的目录，可以根据目录上的页码快速找到所需的内容，</div>
              <div>提升查询速度；</div>
            </li>
            <li>
              <span :class="$style.tit">什么时候需要创建索引？</span>
              <div>频繁作为查询条件的属性应该创建索引；</div>
            </li>
            <li>
              <span :class="$style.tit">什么时候不需要创建索引？</span>
              <div>表记录太少，如一万行以下，不需要创建索引；</div>
            </li>
            <li>
              <span :class="$style.tit">索引有缺点吗？</span>
              <div>索引需要占用物理空间，数据量越大，占用空间越大，所以索引不是越多越好。</div>
            </li>
          </ul>
          <template #footer>
            <span class="dialog-footer">
              <el-button type="primary" @click="dialogVisible = false"> 关闭 </el-button>
            </span>
          </template>
        </el-dialog>
      </div>
    </template>
  </div>
</template>

<script setup>
import { inject } from 'vue';

const props = defineProps({
  entity: Object,
});

const {
  /**
   * 顶部表单节点
   * 包含：名称、系统属性
   * 
   * @type {Vue.ref<HTMLElement>}
   */
  form1,
  /**
   * 顶部表单节点
   * 包含：数据库表名、描述
   * 
   * @type {Vue.ref<HTMLElement>}
   */
  formRef,
  /**
   * 顶部表单名称输入框的占位符
   * 
   * @type {Vue.computed<string>}
   */
  entityPlaceholder,
  /**
   * 聚合 entity 表单属性生成的对象
   * 
   * @type {Vue.reactive<object>}
   * 
   * @property name - 名称
   * @property tableName - 数据库表名
   * @property description - 描述
   * @property origin - 实体来源
   */
  entityModel,
  /**
   * 顶部表单的校验规则
   */
  validateRules,
  /**
   * 数据库视图导入生成的实体
   * entity 的 origin 属性是否是 view
   * 
   * @type {Vue.computed<boolean>}
   */
  isViewEntity,
  /**
   * 顶部表单名称校验规则
   * 
   * @type {Vue.computed}
   */
  formNameRules,
  /**
   * 显示不同的文字提示
   * foucs时显示在顶部表单
   * 
   * @type {Vue.computed<enum>}
   * 
   * @enum '' - 默认，不显示
   * @enum 'entity' - 名称
   * @enum 'tableName' - 数据库表名
   */
  tooltipOpened,
  /**
   * 官方实体名列表
   */
  entityList,
  /**
   * 聚合 entity 系统属性生成的对象
   * 
   * @type {Vue.reactive<object>}
   * 
   * @property createdTime - 默认值: true
   * @property updatedTime - 默认值: true
   * @property createdBy - 默认值: true
   * @property updatedBy - 默认值: true
   */
  storageModel,
  /**
   * 顶部表单 - 系统属性 选项禁用状态
   * 
   * @type {Vue.computed<object>}
   * 
   * @property createdTime
   * @property updatedTime
   * @property createdBy
   * @property updatedBy
   */
  systemActionState,
  /**
   * 设置 entity 的系统属性
   * 
   * @function
   * @param propertyKey - 系统属性名
   * @param isStorage - 是否存储
   */
  setSystemProperty,
  /**
   * 顶部表单数据库表名校验规则
   * 
   * @type {Vue.computed}
   */
  nameRules,
  /**
   * 能否编辑顶部表单数据库表名
   * 
   * @type {Vue.computed<boolean>}
   */
  canSetTableName,
  /**
   * 是否为多模式数据源、
   * 多模式数据源：数据表名需要加上模式名前缀
   * 
   * @type {Vue.computed<boolean>}
   */
  multiSchema,
  /**
   * 展示的功能标签页
   * 
   * @type {Vue.ref<enum>}
   * 
   * @enum 'property' - 默认，属性
   * @enum 'index' - 索引
   */
  tabSelected,
  /**
   * 是否隐藏索引功能标签页
   * 
   * @type {Vue.computed<boolean>}
   */
  hideIndexTab,
  /**
   * 是否展示《如何创建高性能索引》
   * 
   * @type {Vue.ref<boolean>}
   */
  dialogVisible,
} = inject('$context');
</script>

<style module="$icon">
.icon {
  font-size: 14px;
}
.icon:hover {
  color: var(--group-attr-panel-icon-color-hover) !important;
}
</style>

<style module>
.root {
  width: 100%;
  height: 100%;
  overflow: auto;
  background: var(--data-designer-background);
}

.head {
  margin: 16px;
  padding: 16px;
  padding-bottom: 0;
  background: var(--entity-designer-panel-background-color);
  border: 1px solid var(--entity-designer-border-color);
  overflow: auto;
}

div.head [class^='u-input__'] {
  height: 32px;
  line-height: 30px;
}

div.head [class^='u-input__'][color='error'][focus] {
  box-shadow: none;
}

.body {
  margin: 16px;
  background: var(--entity-designer-panel-background-color);
  border: 1px solid var(--entity-designer-border-color);
  overflow: hidden;
}

.root [class^='u-form_item__'][layout='block'] > label {
  margin-top: 7px;
}

.root [class^='u-form__'][gap='small'][layout='block'] [class^='u-form_item__']:not(:last-child) {
  margin-bottom: 16px;
}

.root [class^='u-list-view__'] {
  min-width: auto;
}

.entityListIcon {
  width: 16px;
  height: 16px;
  display: inline-block;
  vertical-align: middle;
}

/* .edit {
  display: inline-block;
  width: 100%;
  line-height: 38px;
} */

.systemwrap {
  display: flex;
  flex-direction: column;

  padding: 12px;
  background: var(--entity-designer-panel-background);
  width: 240px;
  border-radius: 4px;
  border: 1px solid var(--entity-designer-border-color);
}

.systemwrap[class^='u-linear-layout__'][direction='vertical'] > :not(:last-child) {
  margin-bottom: 3px;
}

.entityfrom [class^='u-form_item_label__'] {
  width: 72px !important;
}

.entityfrom [class^='u-form_item_field__'] {
  max-width: calc(100% - 72px) !important;
}

.entityfrom [class^='el-form-item__error'] {
  white-space: nowrap;
}

div.input {
  width: 100%;
}

.icon {
  width: 14px;
  height: 14px;
  display: inline-block;
  vertical-align: middle;
  background-repeat: no-repeat;
}

div.normalfrom [class^='u-form_item_label__'] {
  width: 60px !important;
}

div.normalfrom > div {
  margin-bottom: 16px;
}

div.normalfrom [class^='u-input__'],
div.normalfrom [class^='u-select__'],
div.normalfrom [class^='u-form_item_field__'],
div.normalfrom [class^='u-form_item_wrap__'] {
  width: 240px;
  max-width: 240px;
}

div.normalfrom [class^='u-input__'] {
  height: 32px;
  line-height: 30px;
}

div.reffrom [class^='u-form_item_label__'] {
  width: 94px !important;
}

div.reffrom [class^='u-input__'],
div.reffrom [class^='u-tree-select__'],
div.reffrom [class^='u-select__'],
div.reffrom [class^='u-form_item_field__'],
div.reffrom [class^='u-form_item_wrap__'] {
  width: 240px;
  max-width: 240px;
}

div.reffrom [class^='u-input__'] {
  height: 32px;
  line-height: 30px;
}

div.reffrom .text {
  opacity: 0.3;
}

div.reffrom .relationDelRule [class^='u-form_item_label__'] {
  padding-right: 0;
}

.flexgrid {
  display: flex;
  width: max-content;
}

.gridcolumn {
  width: auto;
  flex-shrink: 0;
}

.gridcolumn:not(:last-child) {
  margin-right: 24px;
}

div.fixedwidthinput,
div.textarea {
  width: 240px;
  max-width: 240px;
}

.tabs [class^='u-tabs_head__'] {
  background: var(--entity-designer-btnnav-background);
  padding: 16px 24px;
}

.tabs > [class^='u-tabs_head__'] [class^='u-tabs_item__'] {
  padding: 0 31px;
}

.info {
  list-style-type: decimal !important;
  line-height: 24px;
  margin: 0 32px -30px 32px;
  color: var(--entity-designer-index-info-color);
}

.indexinfo {
  margin: 0 48px 32px;
}
.indexinfo li {
  margin-bottom: 16px;
}
.indexinfo .tit {
  font-weight: 500;
}

.metadata-head {
  height: 100vh;
}
.metadata-line {
  margin: 16px auto;
  border: 1px dashed #DADEE8;
}
</style>

<style scoped>
.desTextArea :deep(.el-textarea__inner) {
  height: 114px;
}
.info {
  font-size: 18px;
  color: var(--group-attr-panel-icon-color);
  line-height: 0;
}
.tabSelected {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 24px;
  background-color: var(--entity-designer-icon-group-background-color);
  position:sticky;
  z-index: 9;
  top:0px;
}
.tabSelectIcon {
  font-size: 14px;
  vertical-align: bottom;
}
.extraForm :deep(.el-form-item__label) {
  font-size: 12px;
}
.extraForm :deep(.el-form-item) {
  margin-bottom: 16px;
}
.checkBox :deep(.el-checkbox) {
  height: 22px;
}
.checkBox :deep(.el-checkbox__input.is-checked + .el-checkbox__label) {
  font-weight: normal;
}

.iconTip {
  font-size: 12px;
  margin-left: 8px;
  vertical-align: 1px;
}
</style>
