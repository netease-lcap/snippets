<template>
  <div
    v-if="!uneditable && (formItems.length || typeName === 'String')"
    :class="$style.root"
    :pos="pos"
  >
    <template v-if="formItems.length">
      <s-form-item
        v-for="item in formItems"
        :key="item.key"
        :label="item.label"
        :class="$style.formItem"
        :color="getRuleSeverityColor(item)"
      >
        <el-input-number
          :placeholder="`请输入${item.label}`"
          v-model="model[item.key]"
          :disabled="disabled"
          controls-position="right"
          :precision="['Integer', 'Long'].includes(typeName) || property?.typeAnnotation?.isComplexType()?0:undefined"
          :min="item.min || -Infinity"
          :max="item.max || Infinity"
          @change="handleChange(item.key)"
          @blur="handleNumberInputBlur(item.key)"
        >
        </el-input-number>
        <div v-if="getRuleError(item) || item.errorMsg" :class="$style.errorMsg">
          <s-others-icon name="solid-hint"></s-others-icon>
          {{ getRuleError(item) || item.errorMsg }}
        </div>
      </s-form-item>
    </template>
    <template v-else-if="typeName === 'String'">
      <s-form-item
        label="最小长度"
        :class="$style.formItem"
        :color="
          getRuleSeverityColor({
            key: 'minLength',
            errorMsg: maxErrorMsg
          })
        "
      >
        <el-input-number
          placeholder="请输入最小长度（默认为 0）"
          :min="0"
          :max="model.maxLength || MAX_LEN"
          v-model="model.minLength"
          :disabled="disabled"
          controls-position="right"
          @change="handleChange('minLength')"
          @blur="handleNumberInputBlur('minLength')"
        >
        </el-input-number>
        <div v-if="getRuleError({ key: 'minLength' })" :class="$style.errorMsg">
          <s-others-icon name="solid-hint"></s-others-icon>{{ getRuleError({ key: 'minLength' }) }}
        </div>
      </s-form-item>
      <!-- 最大长度 -->
      <s-form-item
        label="最大长度"
        :class="$style.formItem"
        style="margin-bottom: 16px"
        :color="
          getRuleSeverityColor({
            key: 'maxLength',
            errorMsg: maxErrorMsg
          })
        "
      >
        <el-select
          style="width:240px"
          v-model="maxLengthValue"
          :color="
            maxLengthValue &&
            getRuleSeverityColor({
              key: 'maxLength',
              errorMsg: maxErrorMsg
            })
          "
          :disabled="isViewEntity"
          :class="[maxLengthValue === '' ? $style.smallSelect : '']"
          @change="onMaxLengthChange"
        >
          <el-option
            v-for="item in maxLengthOptions"
            :key="item.value"
            :label="item.text"
            :value="item.value"
          />
        </el-select>
        <el-input-number
          v-if="maxLengthValue === ''"
          ref="numInputRef"
          v-model="model.maxLength"
          placeholder="请输入最大长度"
          controls-position="right"
          :class="$style.numInput"
          :min="0"
          :max="MAX_LEN"
          :disabled="disabled"
          @blur="handleMaxlengthNumberInputBlur('maxLength')"
          @click="onMaxlengthNumberInputClick()"
        >
        </el-input-number>
        <div v-if="getRuleError({ key: 'maxLength' }) || maxErrorMsg" :class="$style.errorMsg">
          <s-others-icon name="solid-hint"></s-others-icon>
          {{ getRuleError({ key: 'maxLength' }) || maxErrorMsg }}
        </div>
      </s-form-item>
    </template>
  </div>
</template>

<script setup lang="ts">
import { inject } from 'vue';

const props = withDefaults(
  defineProps<{
    pos?: string;
    property: any;
    disabled: boolean;
    isViewEntity: boolean;
  }>(),
  {
    disabled: false,
    isViewEntity: false,
  }
);

const {
  /**
   * 最大长度输入框的 ref
   *
   * @type {Vue.ref<HTMLInputElement>}
   */
   numInputRef,
  /**
   * 是否不可编辑
   *
   * @type {Vue.computed<boolean>}
   */
  uneditable,
  /**
   * 表单的编辑项
   *
   * @type {Vue.computed<array>}
   */
  formItems,
  /**
   * 实体属性数据类型
   *
   * @type {Vue.computed<string>}
   */
  typeName,
  /**
   * 报错等级
   *
   * @function
   * @param item - 规则项 和 错误信息
   * @return 'error' | 'warning' | ''
   */
  getRuleSeverityColor,
  /**
   * 聚合数据生成对象
   *
   * @type {Vue.reactive<object>}
   * @property min - 最小值
   * @property max - 最大值
   * @property minLength - 最小长度
   * @property maxLength - 最大长度
   */
  model,
  /**
   * 保存 model 的属性值到节点上
   *
   * @function
   * @param key - 属性名
   */
  handleChange,
  /**
   * 数字输入框清空时不会触发change事件，需要增加blur处理
   *
   * @function
   * @param key - 属性名
   */
  handleNumberInputBlur,
  /**
   * 获取目标规则的错误
   *
   * @function
   * @param item - 目标规则
   * @return 目标规则上的错误
   */
  getRuleError,
  /**
   * 最大长度
   *
   * @type {Vue.ref<string>}
   */
  maxLengthValue,
  /**
   * 设置最大长度
   *
   * @function
   * @param event - 修改后的最大长度
   */
  onMaxLengthChange,
  /**
   * 预设最大长度选项
   *
   * @type {Vue.ref<array>}
   */
  maxLengthOptions,
  /**
   * 处理最大长度数字输入框的失焦事件
   *
   * @function
   * @param key - 规则字段
   */
  handleMaxlengthNumberInputBlur,
  /**
   * 最大长度
   *
   * @type {Vue.ref<number>}
   */
  MAX_LEN,
  /**
   * 最大长度数字输入框的点击事件
   *
   * @function
   */
  onMaxlengthNumberInputClick,
  /**
   * 最大长度输入框的错误信息
   *
   * @type {Vue.ref<string>}
   */
  maxErrorMsg,
} = inject('$context') as any;
</script>

<style module>
.errorMsg {
  position: absolute;
  color: var(--error-color);
  white-space: nowrap;
  line-height: normal;
  top: 100%;
}

.smallSelect {
  width: 114px !important;
}
.smallSelect [class^="el-input"] {
  --el-input-width: 114px !important;
}

.root[pos='field'] .smallSelect {
  width: 84px !important;
}

.numInput {
  width: 116px !important;
  margin-left: 8px;
}
.formItem {
  position: relative;
}

.formItem[color='error'] [class^='el-input__wrapper'] {
  border-color: var(--error-color);
}

.formItem[color='warning'] [class^='el-input__wrapper'] {
  border-color: var(--warning-color);
}

.formItem[color='warning'] .errorMsg {
  color: var(--warning-color);
}
</style>
