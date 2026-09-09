<template>
  <!-- 因为老数据没有规则，默认展示平台生成然后不允许修改 -->
  <el-select style="width:240px;"
    v-model="generationRule"
    :disabled="disabledRules"
    @change="onSelect"
    placeholder="请选择"
  >
    <el-option
      v-for="item in generationRules"
      :key="item.value"
      :label="item.text"
      :value="item.value"
    />
  </el-select>
</template>

<script setup>
import { inject } from 'vue';

const props = defineProps({
  selectedItem: Object,
  entity: Object,
  disabled: Boolean,
});

const {
  /**
   * 主键生成规则
   * 
   * @type {Vue.ref<string>}
   */
  generationRule,
  /**
   * 禁用主键生成规则
   * 
   * @type {Vue.computed<boolean>}
   */
  disabledRules,
  /**
   * 设置主键生成规则
   * 
   * @function
   * @param event - 修改后的设置主键生成规则
   */
  onSelect,
  /**
   * 可选的主键生成规则
   * 
   * @type {Vue.computed<array>}
   */
  generationRules,
} = inject('$context');
</script>
