<template>
  <div>
    <el-form-item :class="$style.backupFormItem" label="备份编号" required label-position="left">
      <span :class="$style.lastVersionTitle">
        (上次保存的备份：{{ lastVersion ? lastVersion : '暂无' }} )
      </span>
    </el-form-item>
    <el-row :class="$style.version" :gutter="20">
      <el-col :span="8">
        <div>主版本号</div>
        <el-input-number v-model="majorVersion" @change="handleInput" :min="0" controls-position="right" />
      </el-col>
      <el-col :span="8">
        <div>次版本号</div>
        <el-input-number v-model="minorVersion" @change="handleInput" :min="0" controls-position="right" />
      </el-col>
      <el-col :span="8">
        <div>修订号</div>
        <el-input-number v-model="revision" @change="handleInput" :min="0" controls-position="right" />
      </el-col>
    </el-row>
  </div>
</template>

<script setup>
import { inject } from 'vue';

const props = defineProps({
  lastVersion: String,
});

const {
  /**
   * 主版本号
   *
   * @type {Vue.ref<string>}
   */
  majorVersion,
  /**
   * 次版本号
   *
   * @type {Vue.ref<string>}
   */
  minorVersion,
  /**
   * 修订号
   *
   * @type {Vue.ref<string>}
   */
  revision,
  /**
   * 处理版本号输入变更
   *
   * @function
   */
  handleInput,
} = inject('$context') || {};
</script>

<style module>
.backupFormItem {
  margin-top: 10px;
  margin-bottom: 0px !important;
}
.backupFormItem [class^='el-form-item__label'] {
  margin-bottom: 0px !important;
}
.version {
  display: flex;
}
.version [class^='el-input'] {
  max-width: 112px;
}
.infoTitle {
  margin: 20px 0 10px;
}
.lastVersionTitle {
  position: absolute;
  top: -27px;
  left: 70px;
  color: #666;
}
</style>


