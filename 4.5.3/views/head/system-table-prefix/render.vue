<template>
  <div :class="$style.root" :position="props.position">
    <div :class="$style.iconBox">
      <span :class="[props.position === 'export' ? $style.required : undefined]">系统表前缀</span>
      <s-others-icon name="info" tooltipPlacement="top" tooltip=" " :class="$style.iconInfo">
        <template #tooltip>流程2.0、权限、定时任务等模块创建的数据库表，为了避免多个应用的系统表名冲突，增加了前缀自定义表前缀时，需要统一为前缀</template>
      </s-others-icon>
    </div>
    <el-radio-group v-model="systemTablePrefixType" :class="$style.radioGroup">
      <el-radio :value="SystemTablePrefixTypeEnum.default">应用ID（默认）</el-radio>
      <el-radio :value="SystemTablePrefixTypeEnum.custom"
        :class="$style.customRadio"
        @click="handleSelectSystemTablePrefixCustom">
        <div>自定义表前缀</div>
        <el-form v-if="props.position === 'preference' && systemTablePrefixType === SystemTablePrefixTypeEnum.custom" :class="$style.form"
          :model="systemTablePrefix"
        >
          <el-form-item ref="systemTablePrefixFormItem" prop="value" :rules="rules.systemTablePrefix" style="margin-bottom: 0">
            <el-input
              v-model="systemTablePrefix.value"
              placeholder="请输入前缀" maxlength="10"
            />
          </el-form-item>
        </el-form>
      </el-radio>
    </el-radio-group>
    <el-form v-if="props.position === 'export' && systemTablePrefixType === SystemTablePrefixTypeEnum.custom" :class="$style.form"
          :model="systemTablePrefix"
        >
      <el-form-item ref="systemTablePrefixFormItem" prop="value" :rules="rules.systemTablePrefix" :class="$style.formItem">
        <el-input
          v-model="systemTablePrefix.value"
          placeholder="请输入前缀" maxlength="10"
        />
      </el-form-item>
    </el-form>
  </div>
</template>

<script setup>
import { inject } from 'vue';

const props = defineProps({
  position: String,
});

const {
  /**
   * 系统表前缀类型
   * 
   * @type {Vue.ref<string>}
   */
  systemTablePrefixType,
  /**
   * 系统表前缀值
   * 
   * @type {Vue.ref<object>}
   * @property {string} value - 前缀值
   */
  systemTablePrefix,
  /**
   * 表单验证项引用
   * 
   * @type {Vue.ref<HTMLElement>}
   */
  systemTablePrefixFormItem,
  /**
   * 表单验证规则
   * 
   * @type {object}
   */
  rules,
  /**
   * 系统表前缀类型枚举
   * 
   * @type {object}
   */
  SystemTablePrefixTypeEnum,
  /**
   * 选择自定义表前缀事件处理
   * 
   * @function
   */
  handleSelectSystemTablePrefixCustom,
} = inject('$context') || {};
</script>

<style module>
.root {
    display: flex;
    width: 100%;
}
.iconBox {
    display: flex;
    flex-grow: 1;
    height: 32px;
    line-height: 32px;
}
.iconInfo {
  color: var(--group-attr-panel-icon-color);
  font-size: 16px;
  margin-left: 4px;
}
.required::after {
  content: "*";
  color: var(--el-color-danger);
  margin-left: 4px;
}

.root[position="preference"] .radioGroup{
  width: 188px;
}
.root[position="preference"] .customRadio {
  align-items: flex-start;
  height: auto;
}
.root[position="preference"] .form {
  margin-top: 6px;
}

.root[position="export"] {
  flex-wrap: wrap;
  margin-bottom: 18px;
}
.root[position="export"] .iconBox {
  width: 100%;
}
.root[position="export"] .form {
  display: flex;
  align-items: center;
}
.root[position="export"] .formItem {
  margin-bottom: 0;
  margin-left: 12px;
}
.root[position="export"] .form :global(.el-form-item__error) {
  width: 200px;
}
</style>

