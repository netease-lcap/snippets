<template>
  <el-dialog
    title="创建凭证"
    v-model="visible"
    align-center
    destroy-on-close
    append-to-body
    width="432px"
    :close-on-click-modal="false"
    :close-on-press-escape="false"
  >
    <el-form
      ref="formRef"
      label-position="top"
      require-asterisk-position="right"
      :model="formModel"
      :rules="rules"
    >
      <el-form-item label="凭证类型" prop="type" required>
        <el-radio-group v-model="formModel.type">
          <el-radio v-for="item in certTypeList" :key="item.value" :value="item.value">{{
            item.text
          }}</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="用户名" prop="username" required>
        <el-input v-model="formModel.username" placeholder="请输入用户名" maxlength="32"></el-input>
      </el-form-item>
      <el-form-item
        v-if="formModel.type === 'SSHUsernameWithPrivateKey'"
        label="私钥"
        prop="privateKey"
        required
      >
        <el-input
          v-model="formModel.privateKey"
          :rows="3"
          type="textarea"
          placeholder="请输入私钥"
        ></el-input>
      </el-form-item>
      <el-form-item v-else label="密码" prop="password" required>
        <el-input
          v-model="formModel.password"
          placeholder="请输入密码"
          show-password
          type="password"
          maxlength="32"
        ></el-input>
      </el-form-item>
      <el-form-item label="描述" prop="description">
        <el-input
          maxlength="16"
          placeholder="请输入描述"
          v-model="formModel.description"
        ></el-input>
      </el-form-item>
    </el-form>
    <el-row justify="end">
      <el-button @click="onClose">取 消</el-button>
      <el-button type="primary" @click="onCreate">创 建</el-button>
    </el-row>
  </el-dialog>
</template>

<script setup>
import { inject } from 'vue';

const {
  /**
   * 弹窗可见状态
   *
   * @type {Vue.ref<boolean>}
   */
  visible,
  /**
   * 表单实例引用
   *
   * @type {Vue.ref<import('element-plus').FormInstance>}
   */
  formRef,
  /**
   * 创建凭证表单数据
   *
   * @type {Vue.reactive<object>}
   * @property {string} type - 凭证类型
   * @property {string} username - 用户名
   * @property {string} privateKey - 私钥
   * @property {string} password - 密码
   * @property {string} description - 描述
   */
  formModel,
  /**
   * 表单校验规则
   *
   * @type {object}
   */
  rules,
  /**
   * 凭证类型列表
   *
   * @type {Array<{ value: string, text: string }>}
   */
  certTypeList,
  /**
   * 关闭弹窗并重置表单
   *
   * @function
   */
  onClose,
  /**
   * 提交创建凭证
   *
   * @function
   */
  onCreate,
} = inject('$context') || {};
</script>

<style module></style>