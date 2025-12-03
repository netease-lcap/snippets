<template>
  <div>
    <div v-if="!isViewEntity" class="topIconGroup">
      <!-- 添加实体索引 -->
      <s-others-icon button tooltip="添加属性" name="add" @click="addItem"></s-others-icon>
      <s-others-icon
        name="remove"
        button
        :tooltip="
          !selectedItem ||
          !selectedItem.naslNode ||
          !selectedItem.naslNode.id ||
          selectedItem.loading ||
          selectedItem.naslNode.primaryKey
            ? '请先选择属性'
            : '删除选中属性'
        "
        :disabled="
          !selectedItem ||
          !selectedItem.naslNode ||
          !selectedItem.naslNode.id ||
          selectedItem.loading ||
          selectedItem.naslNode.primaryKey
        "
        @click="removeItem(selectedItem)"
      >
      </s-others-icon>
      <span :class="$style.icondivider">|</span>
      <s-others-icon
        name="position-up"
        button
        :tooltip="
          !selectedItem ||
          !selectedItem.naslNode ||
          !selectedItem.naslNode.id ||
          selectedItem.loading ||
          renderList[1]?.naslNode.name === selectedItem.naslNode.name
            ? '当前属性已置顶'
            : '上移'
        "
        :disabled="
          !selectedItem ||
          !selectedItem.naslNode ||
          !selectedItem.naslNode.id ||
          selectedItem.loading ||
          renderList[1]?.naslNode.name === selectedItem.naslNode.name
        "
        @click="moveUp()"
      >
      </s-others-icon>
      <s-others-icon
        name="position-down"
        button
        :tooltip="
          !selectedItem ||
          !selectedItem.naslNode ||
          !selectedItem.naslNode.id ||
          selectedItem.loading ||
          renderList[renderList.length - 1].naslNode.name === selectedItem.naslNode.name
            ? '当前属性已置底'
            : '下移'
        "
        :disabled="
          !selectedItem ||
          !selectedItem.naslNode ||
          !selectedItem.naslNode.id ||
          selectedItem.loading ||
          renderList[renderList.length - 1].naslNode.name === selectedItem.naslNode.name
        "
        @click="moveDown()"
      >
      </s-others-icon>
    </div>
    <div :class="$style.bodywrap" class="tableCell">
      <el-table
        ref="tableviewRef"
        :data="renderList"
        highlight-current-row
        :style="`min-width:800px;${renderList.length > 8 ? 'height:365px;' : ''}`"
        :class="
          renderList.length > 8
            ? [$style.table, $style.tablescroll, 's-data-table-edit']
            : [$style.table, 's-data-table-edit']
        "
        :value="selectedItem && selectedItem.name"
        value-field="name"
        :header-row-style="{ height: '40px' }"
        :cell-style="{ padding: '0px' }"
        :row-style="{ height: '42px' }"
        :row-class-name="setRowClassName"
        empty-text="暂无数据"
        @row-contextmenu="onContextMenuRow"
        @row-click="onRowClick"
        @click.right.prevent
      >
        <el-table-column width="38px">
          <template #default="{ row: item, $index }">
            <template v-if="$index === -1" />
            <div v-else>
              <s-others-icon
                name="key"
                :class="[$style.iconBase, $style.iconKey]"
                v-if="item.primaryKey"
              ></s-others-icon>
              <s-others-icon
                name="link"
                :class="[$style.iconBase, $style.iconLink]"
                v-if="item.$relationProperty"
              ></s-others-icon>
            </div>
          </template>
        </el-table-column>
        <!-- 名称 -->
        <el-table-column label="名称" style="width: 20%">
          <template #default="{ row: item, $index }">
            <template v-if="$index === -1" />
            <template v-else-if="getDisable(item)">
              <div :class="$style.text">
                <span>{{ item.naslNode.name }}</span>
              </div>
            </template>
            <template v-else>
              <div v-if="item.edit === 'name'" :class="$style.editwrap">
                <!-- value 传入的是被验证的值 -->
                <u-validator
                  :value="item.naslNode.name"
                  :rules="[
                    `maxLength(30) | ^azAZ | ^azAZ09_$ #属性为英文字母、数字和下划线，且首字母小写 | notKeywords | notEntityPropertyKeywords`,
                    {
                      validate: 'excluded',
                      trigger: 'blur',
                      message: '该属性名已经存在',
                      args: [entity.getRenderEntityIndexExistingNames([item.naslNode])]
                    },
                    {
                      validate: 'required',
                      required: true,
                      trigger: 'blur',
                      message: '属性不得为空'
                    }
                  ]"
                  @blur-valid="onBlurName(item, $event.value)"
                  @blur-invalid="errorScrollIntoView"
                  @validate-result="hasInvalid = !$event.valid"
                  v-slot="slotProps"
                >
                  <s-input
                    placeholder="请输入属性名称"
                    :model-value="item.naslNode.name"
                    ref="nameEditor"
                    :class="$style.input"
                    @blur:value="slotProps.blurFn($event)"
                    @keyup.enter="slotProps.blurFn()"
                    @input="slotProps.inputChange($event)"
                  >
                  </s-input>
                </u-validator>
              </div>
              <div
                :class="$style.edit"
                v-else
                @dblclick="onSetItemEdit(item, 'name', true)"
                tabindex="0"
                @keyup="onKeyUp($event, item, 'name')"
                :title="item.naslNode.name"
              >
                <div :class="$style.text">
                  <span :class="$style.textContent">{{ item.naslNode.name }}</span>
                  <s-others-icon
                    name="edit"
                    :class="[$style.iconBase, $style.iconEdit]"
                    @click="onSetItemEdit(item, 'name', true)"
                  >
                  </s-others-icon>
                </div>
              </div>
            </template>
          </template>
        </el-table-column>
        <!-- 属性 -->
        <el-table-column label="属性">
          <template #default="{ row: item, $index }">
            <template v-if="$index === -1" />
            <template v-else-if="getDisable(item)">
              <div :class="$style.text">
                <span>{{ item.naslNode?.propertyNames?.toString() }}</span>
              </div>
            </template>
            <template v-else>
              <div :class="$style.editwrap" v-if="item.edit === 'propertyIds'">
                <el-select
                  :model-value="item.naslNode.propertyNames"
                  ref="propertyIdsEditor"
                  multiple
                  clearable
                  collapse-tags
                  automatic-dropdown
                  @change="setIndexPropertyIds($event, item)"
                  @visible-change="onVisibleChangePropertyIds($event, item)"
                >
                  <el-option
                    v-for="item in propertyList"
                    :key="item.value"
                    :label="item.text"
                    :value="item.value"
                    @click.stop
                  />
                </el-select>
              </div>
              <div
                :class="$style.edit"
                v-else
                :ref="`propertyIds${item.id}`"
                @dblclick="onDblClickPropertyIds(item, $event)"
                tabindex="0"
                @keyup="onKeyUp($event, item, 'propertyIds')"
                :title="item.naslNode?.propertyNames?.toString()"
              >
                <div :class="$style.text">
                  <span :class="$style.textContent">{{
                    item.naslNode?.propertyNames?.toString()
                  }}</span>
                  <s-others-icon
                    name="edit"
                    :class="[$style.iconBase, $style.iconEdit]"
                    @click="onSetItemEdit(item, 'propertyIds', true)"
                  >
                  </s-others-icon>
                </div>
              </div>
            </template>
          </template>
        </el-table-column>
        <!-- 是否唯一 -->
        <el-table-column label="是否唯一" width="80">
          <template #default="{ row: item, $index }">
            <template v-if="$index === -1" />
            <el-checkbox
              v-else
              @change="setUnique(item, $event)"
              :class="$style.checkbox"
              :disabled="getDisable(item)"
              :model-value="item.naslNode.unique"
              :tabindex="getDisable(item) ? -1 : 0"
            >
            </el-checkbox>
          </template>
        </el-table-column>
        <!-- 描述 -->
        <el-table-column label="描述">
          <template #default="{ row: item, $index }">
            <template v-if="$index === -1" />
            <div v-else-if="getDisable(item)" :class="$style.text">
              <span>{{ item.naslNode.description }}</span>
            </div>
            <template v-else>
              <div :class="$style.editwrap" v-if="item.edit === 'description'">
                <u-validator
                  v-slot="slotProps"
                  :value="item.naslNode.description"
                  rules="maxLength(63)"
                  :class="$style.validator"
                  @blur-valid="onBlurDescription(item, $event.value)"
                  @blur-invalid="errorScrollIntoView"
                  @validate-result="hasInvalid = !$event.valid"
                >
                  <s-input
                    placeholder="请输入描述"
                    :model-value="item.naslNode.description"
                    ref="descriptionEditor"
                    :class="$style.input"
                    @blur:value="slotProps.blurFn($event)"
                    @keyup.enter="slotProps.blurFn()"
                    @input="slotProps.inputChange($event)"
                  >
                  </s-input>
                </u-validator>
              </div>
              <div
                :class="$style.edit"
                v-else
                @dblclick="onSetItemEdit(item, 'description', true)"
                tabindex="0"
                @keyup="onKeyUp($event, item, 'description')"
                :title="item.naslNode.description"
              >
                <div :class="$style.text">
                  <span :class="$style.textContent">{{ item.naslNode.description }}</span>
                  <s-others-icon
                    name="edit"
                    :class="[$style.iconBase, $style.iconEdit]"
                    @click="onSetItemEdit(item, 'description', true)"
                  ></s-others-icon>
                </div>
              </div>
            </template>
          </template>
        </el-table-column>
      </el-table>
      <!-- 右键点击 -->
      <el-popover
        :disabled-scroll="true"
        trigger="click"
        ref="menuPopper"
        :popper-options="popperOptions"
        v-if="menuSelectedItem && popperExistMap[getPopperId(menuSelectedItem)]"
        v-model:visible="popperVisibleMap[getPopperId(menuSelectedItem)]"
        :virtual-ref="triggerRef"
        :placement="placement"
        :show-arrow="false"
        @hide="onMenuPopperHide"
      >
        <el-menu :class="$contextmenu.menu" @click="onMenuClick">
          <el-menu-item-group :class="$contextmenu.group" title="索引">
            <el-menu-item :class="$contextmenu.item" @click="onMenuFindUsage"
              >查找引用</el-menu-item
            >

            <el-menu-item
              :class="$contextmenu.item"
              @click="onMenuDelete(removeItem)"
              :disabled="menuSelectedItem && !menuSelectedItem.id && isViewEntity"
              >删除</el-menu-item
            >
          </el-menu-item-group>
        </el-menu>
      </el-popover>
    </div>
    <div :class="[$style.bodywrap1]" v-show="errorMsgs.length">
      <div v-for="errorMsg in errorMsgs" :key="errorMsg" class="errorMsg">
        <i class="infoicon"></i>
        <span>{{ errorMsg }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { inject } from 'vue';

const props = defineProps({
  entity: Object,
});

const {
  /**
   * 名称输入框节点
   *
   * @type {Vue.ref<HTMLElement>}
   */
  nameEditor,
  /**
   * 属性输入框节点
   *
   * @type {Vue.ref<HTMLElement>}
   */
  propertyIdsEditor,
  /**
   * 描述输入框节点
   *
   * @type {Vue.ref<HTMLElement>}
   */
  descriptionEditor,
  /**
   * 数据库视图导入生成的实体
   * entity 的 origin 属性是否是 view
   *
   * @type {Vue.computed<boolean>}
   */
  isViewEntity,
  /**
   * 添加实体索引
   *
   * @function
   */
  addItem,
  /**
   * 表格选中的行元素
   *
   * @type {Vue.computed<object>}
   */
  selectedItem,
  /**
   * 移除实体索引
   *
   * @function
   * @param item - 需要删除的行元素
   */
  removeItem,
  /**
   * 表格的渲染数据
   *
   * @type {Vue.computed<array>}
   */
  renderList,
  /**
   * 上移当前选中的实体索引
   *
   * @function
   */
  moveUp,
  /**
   * 下移当前选中的实体索引
   *
   * @function
   */
  moveDown,
  /**
   * 设置表单列的样式
   *
   * @function
   */
  setRowClassName,
  /**
   * 右键选中表格行元素
   *
   * @function
   */
  onContextMenuRow,
  /**
   * 选中表格行元素
   *
   * @function
   */
  onRowClick,
  /**
   * 名称输入框失焦并保存数据
   *
   * @function
   * @param item - 当前行元素
   * @param name - 需要保存的名称
   */
  onBlurName,
  /**
   * 验证不通过时滚动到错误元素
   */
  errorScrollIntoView,
  /**
   * 存在错误
   *
   * @type {Vue.ref<boolean>}
   */
  hasInvalid,
  /**
   * 设置表格列元素编辑状态
   *
   * @function
   * @param item - 当前行元素
   * @param name - 属性名称
   * @param focus - 是否选中输入框
   */
  onSetItemEdit,
  /**
   * Tab 键跳转切换下一个属性
   *
   * @function
   * @param event - 原生键盘事件
   * @param item - 行元素
   * @param name - 属性名称
   */
  onKeyUp,
  /**
   * 是否禁止编辑行元素
   *
   * @function
   * @param item - 行元素
   */
  getDisable,
  /**
   * 设置propertyNames
   *
   * @function
   * @param value - 需要设置的propertyNames
   * @param item - 行元素
   */
  setIndexPropertyIds,
  /**
   * 下拉框出现/隐藏时触发
   *
   * @function
   * @param value - 需要设置的propertyNames
   * @param item - 行元素
   */
  onVisibleChangePropertyIds,
  /**
   * 双击编辑属性输入框
   *
   * @function
   * @param item - 行元素
   * @param event - 原生事件
   */
  onDblClickPropertyIds,
  /**
   * 编辑是否唯一
   *
   * @function
   * @param item - 行元素
   * @param unique - 是否唯一
   */
  setUnique,
  /**
   * 描述输入框失焦并保存数据
   *
   * @function
   * @param item - 当前行元素
   * @param description - 需要保存的描述
   */
  onBlurDescription,
  /**
   * ElementPlus 的 popper 配置
   */
  popperOptions,
  /**
   * 右键菜单选中项
   *
   * @type {Vue.ref<object>}
   */
  menuSelectedItem,
  /**
   * 项对应的右键菜单是否存在
   */
  popperExistMap,
  /**
   * 项对应的右键菜单是否显示
   */
  popperVisibleMap,
  /**
   * 项对应的右键菜单对应节点唯一标识函数
   */
  getPopperId,
  /**
   * 右键菜单的触发节点
   */
  triggerRef,
  /**
   * 右键菜单的出现位置
   */
  placement,
  /**
   * 右键菜单隐藏时的处理函数
   */
  onMenuPopperHide,
  /**
   * 菜单-查找引用
   *
   * @function
   */
  onMenuFindUsage,
  /**
   * 菜单-删除
   *
   * @function
   */
  onMenuDelete,
  /**
   * 错误信息
   *
   * @type {Vue.computed<array>}
   */
  errorMsgs,
  /**
   * 属性列表
   *
   * @type {Vue.computed<array>}
   */
  propertyList,
} = inject('$context');
</script>

<style module>
div.table [class^='u-input__']:focus,
div.table [class^='u-input__'][focus],
div.table [class^='u-input__']:hover,
div.table [class^='u-select__']:focus,
div.table [class^='u-select__'][focus],
div.table [class^='u-select__'][opened],
div.table [class^='u-select__']:hover {
  border-color: var(--entity-designer-table-input-focus-border-color);
  box-shadow: var(--entity-designer-table-input-focus-box-shadow);
}

.head {
  margin: 16px;
  padding: 16px;
  background: var(--entity-designer-panel-background-color);
  border: 1px solid var(--entity-designer-border-color);
}

div.head [class^='u-input__'] {
  height: 32px;
  line-height: 30px;
}

.body {
  margin: 16px;
  background: var(--entity-designer-panel-background-color);
  border: 1px solid var(--entity-designer-border-color);
  overflow: hidden;
}

.entityListIcon {
  width: 16px;
  height: 16px;
  display: inline-block;
  vertical-align: middle;
}

.edit {
  display: inline-block;
  width: 100%;
  line-height: 38px;
}

.edit:hover {
  cursor: pointer;
}

.edit:focus {
  outline: none;
}

.edit .text > span {
  color: var(--entity-designer-table-color);
}

.edit .text {
  display: inline-block;
  vertical-align: middle;
}

.editwrap {
  margin-left: -9px;
}

.editwrap [class^='s-datatype-select__'] {
  vertical-align: middle;
}

.text {
  width: 100%;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  word-break: break-all;
}

.textContent {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  word-break: break-all;
  max-width: calc(100% - 28px);
  margin-right: 8px;
}
.textContent:empty {
  display: none;
}

.text > span {
  display: inline-block;
  color: var(--entity-designer-table-disabled-color);
  vertical-align: middle;
}

.validator {
  width: 100%;
}

.btnnav {
  border-bottom: 1px solid var(--entity-designer-border-color);
  border-top: 1px solid var(--entity-designer-border-color);
  padding: 5px 24px;
}

div.btnnav[direction='horizontal'][gap='small'] > :not(:last-child) {
  margin-right: 8px;
}

.btnnav1 {
  border-top: none;
  background-color: var(--entity-designer-btnnav-background);
}

.bodywrap {
  padding: 16px 24px 0;
  overflow-x: auto;
  overflow-y: hidden;
  margin-bottom: 24px;
}

div.bodywrap [class^='u-grid-layout_column__'] {
  padding-bottom: 0;
}

.bodywrap1 {
  padding: 16px 24px;
  background-color: var(--entity-designer-body-background);
}

.advancedWrap {
  padding: 10px 24px 25px 24px;
  background-color: var(--entity-designer-body-background);
}

.entityfrom [class^='u-form_item_label__'] {
  width: 36px !important;
}

div.input {
  width: 100%;
  margin-right: 12px;
}

.iconBase {
  display: inline-block;
  width: 20px;
  height: 38px;
  line-height: 38px;
  text-align: center;
  font-size: var(--font-size-large);
  font-size: 14px;
}

.iconKey {
  color: var(--designer-table-edit-icon-key-color);
}

.iconLink {
  color: var(--designer-table-edit-icon-link-color);
}

.iconEdit {
  color: var(--color-base);
  opacity: 0;
}

.iconLink2 {
  display: inline-block;
  width: 16px;
  height: 16px;
  line-height: 16px;
  font-size: var(--font-size-huge);
  color: var(--brand-primary);
  cursor: pointer;
  position: relative;
  top: 2px;
  left: 4px;
}

.edit:hover .iconEdit {
  opacity: 0.6;
  cursor: pointer;
}

.edit:hover .iconEdit:hover {
  opacity: 1;
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

div.reffrom [class^='u-form_item_extra__'] {
  width: 12px;
}

.flexgrid {
  display: flex;
}

.gridcolumn {
  width: auto;
}

.gridcolumn:not(:last-child) {
  margin-right: 24px;
}

div.fixedwidthinput,
div.textarea {
  width: 240px;
  max-width: 240px;
}

.icondivider {
  color: var(--entity-designer-border-color);
}

.advanced {
  padding-top: 10px;
}

.scaleErrorMsg {
  position: absolute;
  color: rgb(242, 73, 87);
}
.typeError {
  border-radius: 4px;
  border: 1px solid var(--input-border-color-error);
}
.checkbox [class='el-checkbox__label'] {
  line-height: 23px;
  font-size: 12px;
  text-overflow: ellipsis;
  text-align: left;
  font-weight: 600;
  color: var(--el-table-header-text-color);
}
.checkbolabel [class='el-checkbox__label'] {
  line-height: 23px !important;
  font-size: 12px !important;
  text-overflow: ellipsis !important;
  text-align: left !important;
  font-weight: 600 !important;
  color: var(--el-table-header-text-color) !important;
}
.checkbox {
  height: 23px;
  line-height: 23px;
}
</style>
<style module="$contextmenu">
.menu {
  margin-top: 0 !important;
}

.menu .group [class^='el-menu-item-group__title'] {
  --el-menu-text-color: var(--el-menu-top-title-text-color);
  height: 32px;
  font-weight: bold;
  border-bottom: 1px solid var(--line-color-regular);
  background: var(--fill-color-thead);
  color: var(--el-menu-text-color);
  cursor: auto;
}
</style>

<style scoped>
.tableCell :deep(.cell) {
  padding: 0;
}

.topIconGroup {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  gap: 8px;
  background-color: var(--entity-designer-icon-group-background-color);
  padding: 5px 24px;
  border-bottom: 1px solid var(--entity-designer-icon-group-border-color);
}

.extraForm :deep(.el-form-item__label) {
  font-size: 12px;
}
.errorMsg {
  color: var(--designer-font-second-color);
  margin: 0 46px;
}

.errorMsg + .errorMsg {
  margin-top: 10px;
}

.infoicon {
  display: inline-block;
  height: 12px;
  width: 12px;
  background: var(--assets-warning-info-svg) no-repeat;
  background-size: 100% 100%;
  vertical-align: -1px;
  margin-right: 5px;
}
</style>
