<template>
  <div :class="[$style.advanced]" :direction="direction">
    <!-- 高级设置 -->
    <div :class="$style.showAdvanced" @click="triggerShowAdvanced">
      <div :class="$style.advancedLabel">
        高级设置
        <s-others-icon
          :class="$style.icon"
          :show-advanced="property.__showAdvanced || null"
          name="trigger"
        >
        </s-others-icon>
      </div>
      <hr />
    </div>
    <div v-if="property.__showAdvanced" :class="$style.wrap">
      <!-- 存储类型 -->
      <el-form-item label="存储类型" :class="[$style.relationDelRule, $style.dataBaseTypesItem]">
        <el-select
          style="width:240px"
          :class="$style.dataBaseTypesSelect"
          :style="{
            marginBottom: typeNameError ? '20px' : '0'
          }"
          v-model="selectedItemDataBaseInfo.selectedDataType"
          @change="onSelectDataType"
          :color="typeNameError ? 'error' : ''"
          :disabled="disabled"
        >
          <el-option
            v-for="item in selectedItemDataBaseInfo.dataBaseTypes"
            :key="item.value"
            :label="item.displayName"
            :value="item.type"
          >
            <el-tooltip :disabled="item.type !== 'default' || disabled" placement="right" trigger="hover">
              <div>{{ item.displayName }}</div>
              <template #content> 自动选择存储类型和相关设置 </template>
            </el-tooltip>
          </el-option>
        </el-select>

        <div v-if="typeNameError" :class="$style.errorMsg">
          <s-others-icon name="solid-hint"></s-others-icon> {{ typeNameError }}
        </div>
      </el-form-item>
      <el-form-item
        v-for="(option, index) in selectedItemDataBaseInfo.options"
        :key="index"
      >
        <template #label>
          <span>{{ option.info.comment }}</span>
          <el-tooltip :disabled="!option.info.desc || disabled">
            <s-others-icon
              name="info"
              v-if="option.info.desc"
              style="margin-left: 5px"
            ></s-others-icon>
            <template #content>
              <div style="width: 300px">
                {{ option.info.desc }}
              </div>
            </template>
          </el-tooltip>
        </template>

        <el-input-number
          :color="getArgError(option) ? 'error' : ''"
          :disabled="option.isDisabled || option.isDefault || disabled"
          :placeholder="option.placeholder"
          v-model="option.value"
          controls-position="right"
          @change="setDatabaseTypeAnnotationArguments(option, $event)"
          @blur="onBlurInputNumber(option)"
        >
        </el-input-number>

        <div v-if="getArgError(option)" :class="$style.errorMsg">
          <s-others-icon name="solid-hint"></s-others-icon>
          {{ getArgError(option) }}
        </div>
      </el-form-item>
    </div>
  </div>
</template>

<script setup>
import { inject } from 'vue';

const props = defineProps({
  property: Object,
  disabled: Boolean,
  direction: { type: String, default: 'horizontal' },
});

const {
  /**
   * 展开或者收起高级设置
   * 
   * @function
   */
  triggerShowAdvanced,
  /**
   * 存储类型错误信息
   * 
   * @type {Vue.computed<string>}
   */
  typeNameError,
  /**
   * 选中项的数据库类型信息
   * 
   * @type {Vue.reactive<object>}
   * @property selectedDataType - 存储类型
   * @property options - 当前选择存储类型的规则项（精度、显示宽度、存储长度等）
   * @property dataBaseTypes - 可选择的存储类型
   */
  selectedItemDataBaseInfo,
  /**
   * 设置数据库存储类型
   * 
   * @function
   * @param datatype - 修改后的存储类型
   */
  onSelectDataType,
  /**
   * 获取规则项上的错误
   * 
   * @function
   * @param option - 规则项
   * @return string
   */
  getArgError,
  /**
   * 设置数据库类型的参数
   * 
   * @function
   * @param option - 规则项
   * @param event - 值
   */
  setDatabaseTypeAnnotationArguments,
  /**
   * 数字输入框失焦事件
   * 
   * @function
   * @param option - 规则项
   */
  onBlurInputNumber,
} = inject('$context');
</script>
<style module>
.advanced {
  max-width: 743px;
  width: 100%;
}

.showAdvanced {
  position: relative;
  font-size: 12px;
  line-height: 12px;
}

.advanced[direction='horizontal'] .showAdvanced {
  padding-bottom: 26px;
}

.advanced[direction='vertical'] .showAdvanced {
  padding: 20px 0 20px 0;
}

.advanced[direction='horizontal'] [class^='u-form_item_extra__'] {
  width: 12px;
  left: -5px;
}

.advancedLabel {
  display: inline-block;
  vertical-align: middle;
  box-sizing: border-box;
  height: 12px;
  line-height: 12px;
  color: var(--form-item-label-color);
  position: relative;
}

.showAdvanced:hover .advancedLabel {
  cursor: pointer;
  color: var(--primary-color);
}

.advanced .icon {
  transform: rotate(0);
  font-size: 12px;
  height: 12px;
  line-height: 12px;
  position: absolute;
  right: 5px;
}

.advanced .icon[show-advanced] {
  transform: rotate(180deg);
}

.showAdvanced hr {
  border: 1px dashed #e0e0e0;
  display: inline-block;
  vertical-align: middle;
}

.advanced[direction='horizontal'] .advancedLabel {
  width: 106px;
  padding-right: 20px;
  text-align: right;
}

.advanced[direction='horizontal'] [class^='u-form_item_label__'] {
  width: 94px;
  text-align: right;
  padding-right: 8px;
}

.advanced[direction='vertical'] .advancedLabel {
  padding-right: 20px;
}

.advanced[direction='vertical'] [class^='u-form_item_field__'] {
  width: 100%;
  max-width: 100%;
}

.advanced[direction='vertical'] [class^='u-form_item_label__'] {
  text-align: left;
  line-height: 28px;
  width: auto;
  padding-right: 16px;
}

.advanced[direction='horizontal'] .wrap {
  display: flex;
  box-orient: horizontal;
  box-direction: normal;
  flex-direction: row;
  flex-wrap: wrap;
  max-width: 928px;
  align-items: flex-start;
}

.advanced[direction='horizontal'] hr {
  width: calc(100% - 108px);
}

.advanced[direction='vertical'] hr {
  width: calc(100% - 74px);
}

.dataBaseTypesSelect {
  width: 240px;
}
.dataBaseTypesSelect [class^='m-popper__'][class*='u-select_popper__'] [class^='u-select_item__'] {
  padding: 0 0 0 16px;
}

.advanced[direction='horizontal'] .dataBaseTypesItem {
  margin-right: 48px;
}

.advanced[direction='vertical']
  [class^='u-form_item__'][label-size$='small']
  > [class^='u-form_item_field__'] {
  width: 100%;
  max-width: 100%;
}

.advanced[direction='vertical'] [class^='u-form_item_extra__'] {
}

.dataBaseTypesItem [class^='u-form_item_label__'],
.dataBaseTypesItem [class^='u-form_item_field__'] {
  vertical-align: middle !important;
}

.errorMsg {
  position: absolute;
  color: rgb(242, 73, 87);
  top: 28px;
  line-height: 20px;
}

.advanced [class^='u-form_item_extra__'] {
  display: inline-block !important;
}
</style>
