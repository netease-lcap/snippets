<template>
  <el-scrollbar>
    <el-form
      ref="formRef"
      style="padding: 16px 0px 16px 0px;"
      label-position="top"
      hide-required-asterisk
      :model="annotaions"
      @submit.prevent
    >
<!-- :current-component="currentComponent" -->
       <s-form-setter
          :class="$style.formSetter"
          :selected-node="entity"
          :annotaions="annotaions">
       </s-form-setter>
       <s-share-data v-if="!inModule" :node="entity" />
    </el-form>
  </el-scrollbar>
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
  formRef,
  /**
   * 注解列表
   * 
   * @type {Vue.computed<Array<Annotation>>}
   */
  annotaions,
  /**
   * 是否属于模块（依赖库）
   * 
   * @type {Vue.computed<Module>}
   */
  inModule,
} = inject('$context');
</script>

<style lang="scss" module>
.formSetter :global(.el-form-item) :global(.el-form-item__content) {
  left: 0px !important;
}
</style>
