<script lang="ts" setup>
import { useAttrs } from "vue";
import { Option } from "../type";

interface Props {
  multiple: boolean
  options: Option[]
  selectValue: Option[]
  updateMethod: (option: Option) => void
}

interface Emits {
  (e: "update", value: Option): void
}

const emits = defineEmits<Emits>();
const props = defineProps<Props>();

const isCheckedOption = (option: Option) => {
  return props.selectValue.some(item => item.value === option.value);
}

const onClick = (option: Option) => {
  props.updateMethod(option);
}
</script>
<template>
<template v-if="props.options.length > 0">
  <div
    :key="index"
    @click="onClick(option)"
    class="custom-content-select-option"
    v-for="(option, index) in props.options"
    :class="{selected: isCheckedOption(option)}"
  >
    <div class="left">
      <slot name="content" :option="option">
        {{ option.label }}
      </slot>
    </div>
    <div v-if="props.multiple" class="right">
      <el-icon><Check /></el-icon>
    </div>
  </div>
</template>
<div v-else class="custom-content-select-empty">暂无数据</div>
</template>
