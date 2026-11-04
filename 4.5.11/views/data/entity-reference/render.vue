<template>
  <div :class="$style.root">
    <s-others-icon
      v-if="value && !disabled"
      name="save-error"
      :class="$style.clearable"
      :disabled="disabled || !entityNode.children.length || null"
      @click="onClear"
    ></s-others-icon>
    <u-tree-select
      filterable
      value-field="keyword"
      :more-children-fields="['properties']"
      :value="value"
      :data-source="[entityNode]"
      :filter-fields="filterFields"
      :class="[$style.tree, value && $style.hideIcon, error && $style.typeError]"
      :disabled="disabled || !entityNode.children.length"
      :teleported="true"
      @select="onInputSelect($event)"
      @before-select="onBeforeSelect($event)"
    >

      <template #text="{ node, selected, expanded }">
        <div :class="$style.nodeText" v-if="node.concept === 'Entity'">
          <s-concept-icon
            :name="node.concept"
            :select="selected"
            :class="$style.conceptIcon"
          ></s-concept-icon>
          <span>{{ node.name }}</span>
        </div>
        <div :class="$style.nodeText" v-else-if="node.concept === 'EntityProperty'">
          <s-concept-icon
            :name="node.concept"
            :select="selected"
            :class="$style.conceptIcon"
          ></s-concept-icon>
          <span>{{ node.name }}</span>
        </div>
        <div :class="$style.nodeText" v-else-if="node.concept === 'Directory'">
          <s-concept-icon
            :name="expanded ? 'expanded-custom-directory' : 'collapsed-custom-directory'"
            :class="$style.conceptIcon"
          ></s-concept-icon>
          <span>{{ node.name }}</span>
        </div>
        <div :class="$style.nodeText" v-else-if="!node.concept">
          <s-concept-icon
            name="category"
            :class="$style.conceptIcon"
          ></s-concept-icon>
          <span>{{ node.name || node.text }}</span>
        </div>
      </template>

      <template #selected="{ node, selected }">
        <div>
          {{ selected }}
          <s-concept-icon
            :name="node.concept"
            :select="selected"
            :class="$style.conceptIcon"
          ></s-concept-icon>
          <span :class="$style.text">{{ node.keyword }}</span>
        </div>
      </template>
    </u-tree-select>
    <u-tooltip v-if="!entityNode.children.length" reference="prev">没有可以关联的属性</u-tooltip>
  </div>
</template>

<script setup>
import { inject } from 'vue';

const props = defineProps({
  value: String,
  entity: Object,
  property: Object,
  disabled: { type: Boolean, default: false },
  error: { type: Boolean, default: false },
});

const {
  /**
   * 树形结构的实体节点
   * 
   * @type {Vue.reactive<object>}
   * @property text - 文件夹名
   * @property children - 实体列表
   */
  entityNode,
  /**
   * 清空关联属性
   * 
   * @function
   */
  onClear,
  /**
   * 过滤字段
   * 
   * @type {Vue.ref<array>}
   */
  filterFields,
  /**
   * 选中的回调函数
   * 
   * @function
   * @param event - 事件对象
   */
  onInputSelect,
  /**
   * 选择前的回调函数
   * 
   * @function
   * @param event - 事件对象
   */
  onBeforeSelect,
} = inject('$context');
</script>

<style module>
.root {
  position: relative;
  flex: 1;
  width: 100%;
  box-sizing: border-box;
}

.clearable {
  position: absolute;
  right: 5px;
  top: 0;
  font-size: 16px;
  color: var(--designer-icon-cancel-color);
  z-index: 1;
  opacity: 0;
}

.clearable:not([disabled]):hover {
  cursor: pointer;
  color: var(--designer-icon-cancel-color-hover);
}

.clearable:hover + .tree {
  border-color: var(--input-border-color-focus);
}

.root:hover .clearable:not([disabled]) {
  opacity: 1;
}

.text {
  overflow: hidden;
  text-overflow: ellipsis;
}

div.tree[disabled],
div.tree[readonly] {
  background: var(--input-background-disabled);
  color: var(--color-light);
  border: var(--input-border-width) solid var(--input-border-color);
}

div.tree:focus,
div.tree[focus] {
  box-shadow: var(--input-box-shadow-focus);
  border-color: var(--input-border-color-focus);
}

div.tree:hover {
  border-color: var(--input-border-color-focus);
}

div.tree[disabled]:hover {
  border-color: var(--input-border-color);
}

div.hideIcon:after {
  content: '';
}

.conceptIcon {
  margin-right: 3px;
  vertical-align: bottom;
}

.typeError {
  border: 1px solid var(--input-border-color-error);
}
</style>
