<script lang="ts" setup>
import {ref, computed, onMounted} from "vue";
import { ElTooltip } from "element-plus";
interface Props {
  modelValue?: boolean
  width?: number | string
  multiple?: boolean
}

interface Emits {
  (e: "update:modelValue", value: boolean): void
}

const props = withDefaults(defineProps<Props>(), {
  modelValue: false,
  width: "auto"
});
const emits = defineEmits<Emits>();
const tooltipRef = ref<InstanceType<typeof ElTooltip>>();
const visible = computed<boolean>({
  get() {
    return props.modelValue;
  },
  set(value) {
    emits("update:modelValue", value);
  }
})

const onMousedown = (event: MouseEvent) => {
  if(props.multiple) {
    event.preventDefault();
  }
}

defineExpose({
  updatePopper() {
    tooltipRef.value?.updatePopper();
  }
})
</script>
<template>
  <el-tooltip
    :visible="visible"
    :popper-style="{padding: 0, 'max-width': 'none'}"
    placement="bottom-start"
    effect="light"
    ref="tooltipRef"
  >
    <slot></slot>
    <template #content>
      <div class="select-dropdown" :style="{width: props.width}" @mousedown="onMousedown">
        <el-scrollbar max-height="274">
          <slot name="content"></slot>
        </el-scrollbar>
      </div>
    </template>
  </el-tooltip>
</template>
<style lang="scss" scoped>
.select-dropdown {
  padding: 11px 0;
  box-sizing: border-box;
  :deep(.custom-content-select-option) {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 20px;
    width: 100%;
    overflow: hidden;
    cursor: pointer;
    color: #606266;
    font-size: 14px;
    line-height: 32px;
    &:hover {
      background-color: #f5f7fa;
    }
    &.selected {
      color: #035cff;
      font-weight: bold;
      .right {
        display: flex;
        align-items: center;
      }
    }
    .left {
      flex: 1;
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
    }
    .right {
      flex: none;
      display: none;
      margin-left: 5px;
    }
  }
  :deep(.custom-content-select-empty) {
    padding: 0 20px;
    color: #606266;
    text-align: center;
  }
}
</style>
