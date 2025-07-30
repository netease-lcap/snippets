<template>
  <el-dialog v-model="visible" class="s-edit-mock-data-modal" width="500px" align-center destroy-on-close append-to-body
    :close-on-click-modal="false" @close="close()">
    <template #header>
      <span style="margin-right: 8px">编辑预览数据</span>
    </template>
    <!-- @validate="valid = $event.valid"  -->
    <el-form :class="$style.form" ref="form" label-position="top"
      require-asterisk-position="right">
      <div :class="$style.iconbox">
        <el-form-item label="角色" style="width:100%;margin-left: 10px;" :required="!onlyH5" :rules="nameRules">
          <el-select style="width:100%;" placeholder="请选择预览角色" v-model="model.role"
            @select="onSelectTags($event)">
               <el-option
                v-for="item in rolelist"
                :key="item"
                :label="item"
                :value="item"
              />
            </el-select>
        </el-form-item>
      </div>
      <div :class="$style.card">
        <div :class="$style.cardHeader">
          <span>接口数据</span>
          <s-others-icon @click="setUseSearch(true)" v-show="!useSearch" name="search" :class="$style.searchIcon">
          </s-others-icon>
          <el-input v-show="useSearch" ref="search" prefix="search" clearable :placeholder="placeholder"
            v-model="searchKey" @change="onSearchChange" style="width:204px;height:28px">
            <template #prefix>
              <s-others-icon name="search" :class="$style.searchIcon"></s-others-icon>
            </template>
          </el-input>
        </div>
        <div :class="$style.cardBody"  class="mockTextarea" >
             <div  :class="$style.emptyText" v-if="useSearch &&!mockList.find(item=>item.path.toLowerCase().includes(searchKey.toLowerCase()))">
                      搜索结果为空 ，请尝试再次搜索
                    </div>
          <div v-else :class="$style.apiItem"  v-for="(item,index) in mockList" :key="item">
            <el-form-item :label="`接口${index+1}:(${item.method})${item.path}`"
              v-if="!searchKey ||(searchKey&&item.path.toLowerCase().includes(searchKey.toLowerCase()))">
              <el-input type="textarea" v-model.trim="mockList[index].mockData" style="min-height:98px"
               :placeholder="`请输入接口${index+1}的mock数据`">
              </el-input>
            </el-form-item>
          </div>
        </div>
      </div>

    </el-form>
    <template #footer>
      <el-space justify="end">
        <el-button @click="close()">取 消</el-button>
        <el-button type="primary" @click="submit" :loading="submiting">保 存</el-button>
      </el-space>
    </template>
  </el-dialog>
</template>

<script setup>
import { inject } from 'vue';

const {
  /**
   * 表单节点引用
   * 
   * @type {Vue.ref<HTMLElement>}
   */
  form,
  /**
   * 搜索输入框节点引用
   * 
   * @type {Vue.ref<HTMLElement>}
   */
  search,
  /**
   * 对话框是否可见
   * 
   * @type {Vue.ref<boolean>}
   */
  visible,
  /**
   * 关闭对话框方法
   * 
   * @function
   */
  close,
  /**
   * 是否仅H5模式
   * 
   * @type {boolean}
   */
  onlyH5,
  /**
   * 表单验证规则
   * 
   * @type {Array}
   */
  nameRules,
  /**
   * 表单数据模型
   * 
   * @type {object}
   * @property {string} role - 角色名称
   */
  model,
  /**
   * 选择标签回调方法
   * 
   * @function
   * @param {string} event - 选中的标签值
   */
  onSelectTags,
  /**
   * 角色列表数据
   * 
   * @type {Array<string>}
   */
  rolelist,
  /**
   * 是否使用搜索功能
   * 
   * @type {boolean}
   */
  useSearch,
  /**
   * 搜索框占位文本
   * 
   * @type {string}
   */
  placeholder,
  /**
   * 搜索关键词
   * 
   * @type {string}
   */
  searchKey,
  /**
   * 搜索关键词变化回调
   * 
   * @function
   */
  onSearchChange,
  /**
   * Mock数据列表
   * 
   * @type {Array<object>}
   * @property {string} path - 接口路径
   * @property {string} method - 请求方法
   * @property {string} mockData - Mock数据内容
   */
  mockList,
  /**
   * 提交表单方法
   * 
   * @function
   */
  submit,
  /**
   * 是否正在提交中
   * 
   * @type {boolean}
   */
  submiting,
  /**
   * 设置是否使用搜索功能
   * 
   * @function
   * @param {boolean} value - 是否启用搜索
   */
  setUseSearch,
} = inject('$context');
</script>

<style module>
  .form {
    padding: 8px;
  }

.form  [class*='el-textarea__inner'] {
     min-height: 98px !important;

}
.form [class*='el-form-item'] {
xxx-margin-bottom:16px!important;
}

  .cardHeader {
    padding: 0 16px;
    width: 100%;
    height: 32px;
    background-color: #F2F3F5;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .cardBody {
    padding: 16px;
    border-radius: 0px 0px 4px 4px;
    box-sizing: border-box;
    border-width: 0px 1px 1px 1px;
    border-style: solid;
    border-color: #E7EAEE;
    max-height: 434px;
    min-height: 164px;
    overflow: scroll;
  }


  .tagcategory {
    position: absolute;
    top: -28px;
    right: 0px;
    white-space: nowrap;
  }

  .searchIcon {
    color: #666;
  }

  .iconbox {
    display: flex;
    justify-content: space-between;
    margin-bottom: 20px;
  }
  .emptyText{
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: PingFang SC;
    font-size: 12px;
    font-weight: normal;
    line-height: 12px;
    color: #999999;
    height:100%;
    margin-top: 60px;
}

</style>
