<template>
  <el-form-item
    v-for="item in gitCustomConfigs"
    :label="item.title"
    :key="item.code"
    :prop="item.code"
  >
    <el-input
      v-if="item.type === 'multi_line_text'"
      type="textarea"
      v-model="gitForm[item.code]"
      style="width: 200px"
      :autosize="{ minRows: 2, maxRows: 4 }"
      show-word-limit
    ></el-input>
    <el-input
      v-else-if="item.type === 'single_line_text'"
      v-model="gitForm[item.code]"
      autosize
      style="width: 200px"
      show-word-limit
    ></el-input>
    <el-select
      v-else-if="['single_choice', 'multi_choice'].includes(item.type)"
      style="width: 200px"
      :multiple="item.type === 'multi_choice'"
      v-model="gitForm[item.code]"
    >
      <el-option
        v-for="option in item.options"
        :key="option.code"
        :value="option.code"
        :label="option.name"
      >
      </el-option>
    </el-select>
  </el-form-item>
</template>

<script setup>
import { inject } from 'vue';

const {
  /**
   * Git自定义配置列表
   * 
   * @type {Vue.ref<array>}
   */
  gitCustomConfigs,
  /**
   * Git表单数据
   * 
   * @type {Vue.ref<object>}
   */
  gitForm,
} = inject('$context') || {};
</script>

<style module></style>

