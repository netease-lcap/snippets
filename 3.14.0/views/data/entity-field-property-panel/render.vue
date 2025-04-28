<template>
  <div :class="$style.root">
    <el-form ref="formRef" label-position="top" class="s-data-attr-panel-form" :model="model" @submit.prevent>
      <!-- <s-form-item label="名称" description="实体中的属性标识">
        <s-input-rename-panel
          :node="property"
          placeholder="请输入属性名称"
          :disabled="
            readonly ||
            property.loading ||
            property.generationRule !== 'manual' ||
            property.parentNode.loading ||
            property.isDataTypeSetting ||
            property.editable === false ||
            isModule
          "
        ></s-input-rename-panel>
      </s-form-item> -->

      <s-form-item
        label="标题"
        description="在类型列表中显示的文本"
        prop="label"
        :rules="validateRules.label"
      >
        <el-input
          placeholder="请输入属性标题"
          v-model="model.label"
          :disabled="
            readonly ||
            property.loading ||
            property.parentNode.loading ||
            property.isDataTypeSetting ||
            isModule ||
            isInViewEntity
          "
          @keyup.enter="$event.target.blur()"
        >
        </el-input>
      </s-form-item>

      <!-- <s-form-item label="描述" prop="description" :rules="validateRules.description">
        <el-input
          placeholder="请输入属性描述"
          v-model="model.description"
          :disabled="
            readonly ||
            property.loading ||
            property.parentNode.loading ||
            property.isDataTypeSetting ||
            isModule
          "
        >
        </el-input>
      </s-form-item> -->

      <s-form-item
        label="数据类型"
        description="该属性的类型，可以选择基础类型，也可以选择用户创建的类型，如枚举、实体、数据结构"
      >
        <s-datatype-select
          type="attributePanel"
          :unionable="false"
          :no-union-subtype="true"
          :type-annotation="propertyTypeAnnotation"
          :data-type-list="property.lastVersion ? dataTypeList : undefined"
          :last-version="property.lastVersion"
          :show-system-types="false"
          :show-generic-types="true"
          :show-entity="true"
          :show-structures="true"
          :emptyable="false"
          :hasAnonymousStructure="true"
          :disabled="
            readonly ||
            property.loading ||
            property.generationRule !== 'manual' ||
            property.parentNode.loading ||
            property.editable === false ||
            property.isDataTypeSetting ||
            isAppDeploying ||
            isModule || property.primaryKey ||
            isPublishedComplexType(property) ||
            isInViewEntity
          "
          @change="onChangeDatatype"
        >
        </s-datatype-select>
        <u-tooltip v-if="isAppDeploying">应用发布中，暂不能修改</u-tooltip>
      </s-form-item>

      <s-form-item
        label="必填"
        label-position="left"
      >
        <el-switch
          @change="setRequired()"
          v-model="model.required"
          size="small"
          :disabled="
            readonly ||
            property.loading ||
            property.parentNode.loading ||
            property.isDataTypeSetting ||
            property.editable === false ||
            property.generationRule !== 'manual' ||
            property.primaryKey ||
            isModule ||
            isComplexType(property) ||
            isInViewEntity
          "
        ></el-switch>
      </s-form-item>

      <s-form-item
        v-if="property.parentNode.concept === 'Entity' && isDataTypeEntity(property)"
        label="关系"
        description="实体间的关系"
      >
        <el-select
          v-model="model.relationship"
          clearable
          :disabled="
            readonly ||
            property.loading ||
            property.parentNode.loading ||
            property.isDataTypeSetting ||
            property.editable === false ||
            isModule ||
            isInViewEntity
          "
          @change="property.setRelationship($event)"
        >
          <el-option value="OneToOne">一对一</el-option>
          <el-option value="ManyToOne">多对一</el-option>
        </el-select>
      </s-form-item>
      <s-form-item
        v-if="typeName === 'Decimal'"
        label="小数位数"
        description="为空表示小数位数为0"
      >
        <el-input-number
          v-model="model.scale"
          placeholder="请输入小数位数"
          controls-position="right"
          :color="scaleErrorMsg ? 'error' : ''"
          :min="scaleOption.min"
          :max="scaleOption.max"
          :disabled="decimalDisabled"
          :value-on-clear="0"
          @keyup.enter="$event.target.blur()"
          @change="setScale()"
        ></el-input-number>

        <div v-if="scaleErrorMsg" :class="$style.scaleErrorMsg">
          <s-others-icon name="solid-hint"></s-others-icon>
          {{ scaleErrorMsg }}
        </div>
      </s-form-item>

      <s-form-item
        v-if="canSetDefaultValue(property)"
        label="默认值"
        description="该属性的初始值。如果未填写，则使用该属性数据类型的默认值"
        prop="defaultValue"
      >
      <s-attr-input
        title="默认值绑定"
        node-concept="defaultValue"
        :name="property.name"
        :parent-node="property"
        :node="property.defaultValue"
        :class="{ [$style.typeError]: IsError }"
        :disabled="isInViewEntity"
      ></s-attr-input>
      </s-form-item>

      <!-- 如果是主键的话 -->
      <s-form-item v-if="property.primaryKey" label="主键生成规则">
        <skeleton-render
          name="views/data/components/entity-primarykey-rules/index"
          :selected-item="property"
          :entity="property.parentNode"
          :disabled="isModule || isInViewEntity"
        />
      </s-form-item>

      <s-form-item
        label="在页面生成显示"
        description="用于控制实体拖拽生成的页面的显示。如不勾选“表格列”，则该属性对应的表格列不显示"
      >
        <div class="checkboxGroup">
          <el-checkbox
            v-model="model.display.inTable"
            :disabled="
              readonly ||
              property.loading ||
              property.parentNode.loading ||
              property.isDataTypeSetting ||
              isModule
            "
            @change="onSetDisplay()"
          >
            表格列
          </el-checkbox>
          <el-checkbox
            v-model="model.display.inFilter"
            :disabled="
              readonly ||
              property.loading ||
              property.parentNode.loading ||
              property.isDataTypeSetting ||
              isModule ||
              isComplexType(property)
            "
            @change="onSetDisplay()"
          >
            筛选项
          </el-checkbox>
          <el-checkbox
            v-model="model.display.inForm"
            :disabled="
              readonly ||
              property.loading ||
              property.parentNode.loading ||
              property.isDataTypeSetting ||
              isModule ||
              isComplexType(property)
            "
            @change="onSetDisplay()"
          >
            表单项目
          </el-checkbox>
          <el-checkbox
            v-model="model.display.inDetail"
            :disabled="
              readonly ||
              property.loading ||
              property.parentNode.loading ||
              property.isDataTypeSetting ||
              isModule
            "
            @change="onSetDisplay()"
          >
            详情项目
          </el-checkbox>
        </div>
      </s-form-item>

      <skeleton-render
        name="views/data/components/entity-field-rules/index"
        pos="field"
        :property="property"
        :key="'rules_' + property.name"
        :disabled="
          readonly ||
          property.loading ||
          property.parentNode.loading ||
          property.isDataTypeSetting ||
          property.editable === false ||
          isModule ||
          isInViewEntity
        "
        :is-view-entity="isInViewEntity"
        @save="property.setRules(property.rules)"
      />

      <template v-if="canSetReference">
        <s-form-item label="关联属性" description="外键关联">
          <skeleton-render
            name="views/data/entity-reference/index"
            :entity="property.entity"
            :property="property"
            :value="property.relationEntity + '.' + property.relationProperty"
            :disabled="
              property.loading ||
              property.parentNode.loading ||
              property.isDataTypeSetting ||
              isModule ||
              isInViewEntity
            "
            :data-type-list="property.lastVersion ? dataTypeList : undefined"
            @save="onSaveReference"
            @clear="property.clearReference()"
            :key="property.name"
          />
        </s-form-item>
        <s-form-item label="关联属性实体记录删除规则" v-if="property.relationProperty">
          <el-select
            v-model="model.deleteRule"
            @change="property.setDeleteRule($event)"
            :disabled="isModule || isInViewEntity"
            placeholder="请选择"
          >
            <el-option label="不允许删除" value="protect"></el-option>
            <el-option label="允许删除且同时删除本实体记录" value="cascade"></el-option>
          </el-select>
        </s-form-item>
      </template>

      <el-form-item label="数据库列名">
        <u-validator
          style="width: 100%"
          :rules="entityPropertyColumnNameRules"
          :value="model.columnName"
          @blur-dirty-valid="property.setColumnName($event.value)"
          v-slot="slotProps"
        >
          <el-input
            :placeholder="entityPropertyColumnNamePlaceholder"
            v-model="model.columnName"
            :disabled="isTableOrExcelOrigin || isInViewEntity"
            @change="slotProps.blurChange"
            @keyup.enter.stop="$event.target.blur()"
          ></el-input>
        </u-validator>
      </el-form-item>

      <skeleton-render
        name="views/data/components/entity-property-databasetype/index"
        v-if="showDatabasetype"
        :property="property"
        :direction="'vertical'"
        :disabled="isInViewEntity"
      />
    </el-form>
  </div>
</template>

<script setup>
import { inject } from 'vue';

const props = defineProps({
  property: Object,
});

const {
  /**
   * 表单节点
   * 
   * @type {Vue.ref<HTMLElement>}
   */
  formRef,
  /**
   * 聚合 property 数据生成的对象
   * 
   * @type {Vue.reactive<object>}
   */
  model,
  /**
   * 校验规则
   */
  validateRules,
  /**
   * 数据类型
   * 
   * @type {Vue.ref<object>}
   */
  propertyTypeAnnotation,
  /**
   * 修改数据类型
   * 
   * @function
   * @param event - event.value 对应修改后的数据类型
   */
  onChangeDatatype,
  /**
   * 是否应用正在发布
   * 
   * @type {Vue.computed<boolean>}
   */
  isAppDeploying,
  /**
   * 修改必填
   * 
   * @function
   */
  setRequired,
  /**
   * 暂无释义
   * 
   * @function
   * @param property
   * @return boolean
   */
  isDataTypeEntity,
  /**
   * 实体属性数据类型
   * 
   * @type {Vue.computed<string>}
   */
  typeName,
  /**
   * 小数位数的校验错误信息
   * 
   * @type {Vue.computed<string>}
   */
  scaleErrorMsg,
  /**
   * 小数位数的范围
   * 
   * @type {Vue.computed<object>}
   * @property min - 最小值
   * @property max - 最大值
   */
  scaleOption,
  /**
   * 是否禁用小数位数
   * 
   * @type {Vue.computed<boolean>}
   */
  decimalDisabled,
  /**
   * 修改小数位数
   * 
   * @function
   */
  setScale,
  /**
   * 能否设置默认值
   * 
   * @function
   * @param property - 实体属性
   * @return boolean
   */
  canSetDefaultValue,
  /**
   * 是否视图实体
   * 
   * @type {Vue.computed<boolean>}
   */
  isInViewEntity,
  /**
   * 设置在页面生成显示
   * 
   * @function
   */
  onSetDisplay,
  /**
   * 能否设置关联属性
   * 
   * @type {Vue.computed<boolean>}
   */
  canSetReference,
  /**
   * 设置关联属性
   * 
   * @function
   * @param property - 实体属性
   */
  onSaveReference,
  /**
   * 是否是模块（依赖库）
   * 
   * @type {Vue.computed<boolean>}
   */
  isModule,
  /**
   * 是否来源于数据库表/excel
   * 
   * @type {Vue.computed<boolean>}
   */
  isTableOrExcelOrigin,
  /**
   * 是否显示数据库高级设置
   * 
   * 1.没有设置数据源
   * 2.实体是excel导入或者数据源反向导入
   * 3.权限实体（贤宇说一直是以LCAP开头来判断的）
   * 这3种情况不显示数据库设置
   * 
   * @type {Vue.computed<boolean>}
   */
  showDatabasetype,
  /**
   * 是否实体已经发布过并且是复合类型
   * 
   * 1. 实体已经发布过，在app下的depEntities可以找到该实体
   * 2. 实体是复合类型：'Map', 'List'，或者typeKind是reference并且namespace是'entities', 'structures'
   * 
   * @function
   * @param property - 实体属性
   * @return boolean
   */
  isPublishedComplexType,
  /**
   * 复合类型
   * 
   * @function
   * @param property - 实体属性
   * @return boolean
   */
  isComplexType,
  /**
   * 数据库列名校验规则
   * 
   * @type {Vue.computed<object>}
   */
  entityPropertyColumnNameRules,
  /**
   * 数据库列名输入框占位符
   * 
   * @type {Vue.computed<string>}
   */
  entityPropertyColumnNamePlaceholder,
} = inject('$context');
</script>

<style module>
.scaleErrorMsg {
  position: absolute;
  color: rgb(242, 73, 87);
}

.root {
  height: 100%;
  overflow-y: auto;
}
.typeError {
  border-radius: 4px;
  border: 1px solid var(--input-border-color-error);
}
</style>

<style scoped>
.formItem :deep(.el-form-item) {
  margin-bottom: 0px;
}

.checkboxGroup {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  width: 100%;
}
.checkboxGroup .el-checkbox {
  width: 100%;
}
</style>
