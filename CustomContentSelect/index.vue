<script lang="ts" setup>
import { onMounted, ref, reactive, watch, nextTick, toRaw, computed, useSlots } from "vue";
import SelectDropdown from "./components/SelectDropdown.vue";
import CustomContentSelectOptions from "./components/CustomContentSelectOptions.vue";
import { Option } from "./type";
type StringNumber = string | number;
interface Props {
  modelValue?:  StringNumber | Array<StringNumber>
  disabled?: boolean
  width?: string
  filterable?: boolean
  clearable?: boolean
  multiple?: boolean
  options?: Option[]
  placeholder?: string
  filterMethod?: (value: string, done: (options?: Option[]) => void) => void
}

interface Emits {
  (e: "update:modelValue" | "change", value: StringNumber | Array<StringNumber>): void
}

interface State {
  selectValue: Option[]
  filterValue: string
  showDropdown: boolean
  dropdownWidth: string
}

const emits = defineEmits<Emits>();
const props = withDefaults(defineProps<Props>(), {
  disabled: false,
  clearable: false,
  filterable: false,
  multiple: false,
  placeholder: "",
  width: "auto",
  options: () => [],
  filterMethod: (v, d) => d()
});
const inputRef = ref<HTMLInputElement>();
const inputValueRef = ref<HTMLDivElement>();
const filterInputRef = ref<HTMLInputElement>();
const selectTriggerRef = ref<HTMLDivElement>();
const selectDropdownRef = ref<InstanceType<typeof SelectDropdown>>();
const containerHeight = ref<string>("auto");
const filterDone = ref<boolean>(false);
const filterList = ref<Option[]>([]);
const slots = useSlots();
const state = reactive<State>({
  selectValue: [],
  filterValue: "",
  showDropdown: false,
  dropdownWidth: "0px"
});
const optionMap = computed<Record<string, Option>>(() => {
  return props.options.reduce((map: Record<string, Option>, option) => {
    map[option.value] = option;
    return map;
  }, {});
});
const inputWrapperPlaceholder = computed<string>(() => {
  if(!props.filterable && state.selectValue.length === 0) {
    return props.placeholder;
  }
  return "";
});

const inputFilterPlaceholder = computed<string>(() => {
  if(props.filterable) {
    if(state.selectValue.length === 0) {
      return props.placeholder;
    }
  }
  return "";
});

const calcHeight = () => {
  if(inputValueRef.value) {
    containerHeight.value = `${inputValueRef.value?.offsetHeight - 1}px`;
    selectDropdownRef.value?.updatePopper();
  }
}

const onInputFocus = () => {
  state.showDropdown = true;
}

const onInputBlur = () => {
  state.showDropdown = false;
  filterDone.value = false;
  filterList.value = [];
  state.filterValue = "";
}

const triggerFilter = (function() {
  let timerID: any = -1;
  return () => {
    clearTimeout(timerID);
    timerID = setTimeout(() => {
      if(state.filterValue) {
        props.filterMethod(state.filterValue, (options?: Option[]) => {
          filterList.value = options || [];
          filterDone.value = true;
        });
      }
    }, 1000);
  }
})();

const onFilter = () => {
  filterDone.value = false;
  triggerFilter();
}

const onClickChangeChecked = (option: Option) => {
  let arr: any[] = JSON.parse(JSON.stringify(toRaw(state.selectValue)));
  if(props.multiple) {
    const index = arr.findIndex(item => item.value === option.value);
    if(index !== -1) {
      arr.splice(index, 1);
    } else {
      arr.push(option);
    }
    emits("update:modelValue", arr.map(option => option.value));
  } else {
    arr = [option];
    emits("update:modelValue", arr[0].value);
  }
}

const onClickTriggerFocus = () => {
  if(selectTriggerRef.value) {
    state.dropdownWidth = `${selectTriggerRef.value?.offsetWidth}px`;
  }
  if(filterInputRef.value) {
    filterInputRef.value?.focus();
  } else {
    inputRef.value?.focus();
  }
}

const onCloseTag = (index: number) => {
  const arr: any[] = JSON.parse(JSON.stringify(toRaw(state.selectValue)));
  arr.splice(index, 1);
  emits("update:modelValue", arr.map(option => option.value));
}

const reloadSelectValue = () => {
  if(props.multiple) {
    if(Array.isArray(props.modelValue)) {
      state.selectValue = props.modelValue.map(key => {
        const option = toRaw(optionMap.value)[key];
        if(option) {
          return option;
        } else {
          return { label: key, value: key }
        }
      }) as Option[];
    } else {
      state.selectValue = [];
    }
  } else {
    if(props.modelValue) {
      const option = toRaw(optionMap.value)[props.modelValue as string];
      if(option) {
        state.selectValue = [option];
      } else {
        state.selectValue = [{ label: props.modelValue as string, value: props.modelValue }];
      }
    } else {
      state.selectValue = [];
    }
  }
}

const onClickClearValue = () => {
  emits("update:modelValue", props.multiple ? [] : "");
}

watch(() => state.selectValue, () => {
  nextTick(() => {
    calcHeight();
  })
}, { deep: true });

watch(() => props.modelValue, () => {
  if(props.multiple) {
    const newValues = state.selectValue.map(item => item.value);
    const values = (props.modelValue as StringNumber[]) || []
    if((newValues.length !== values.length) || !values.every(v => newValues.includes(v))) {
      reloadSelectValue();
      emits("change", values);
    }
  } else {
    const value = state.selectValue[0]?.value || "";
    if(props.modelValue !== value) {
      reloadSelectValue();
      emits("change", state.selectValue[0]?.value);
    }
  }
}, { deep: true });

onMounted(() => {
  calcHeight();
  if(selectTriggerRef.value) {
    state.dropdownWidth = `${selectTriggerRef.value?.offsetWidth}px`;
  }
  reloadSelectValue();
});
</script>
<template>
<div class="custom-content-select" :class="{'is-focus': state.showDropdown, disabled: props.disabled}"  :style="{width: props.width}">
  <SelectDropdown ref="selectDropdownRef" v-model="state.showDropdown" :width="state.dropdownWidth" :multiple="props.multiple">
    <div class="select-trigger" :class="{clearable: props.clearable}" ref="selectTriggerRef" @click="onClickTriggerFocus">
      <div class="input-value" ref="inputValueRef">
        <div v-if="props.multiple" class="tags">
          <el-tag type="info" v-for="(option, index) in state.selectValue" :closable="!props.disabled" :key="index" @close="onCloseTag(index)">
            <slot name="content" :option="option">
              {{ option.label }}
            </slot>
          </el-tag>
        </div>
        <div v-else-if="state.selectValue[0] && ((props.filterable && !state.showDropdown) || !props.filterable)" class="text">
          <slot name="content" :option="state.selectValue[0]">
            <span>{{ state.selectValue[0].label }}</span>
          </slot>
        </div>
        <div v-if="props.filterable" class="input-wrapper">
          <input
            type="text"
            :disabled="props.disabled"
            @input="onFilter"
            @blur="onInputBlur"
            ref="filterInputRef"
            @focus="onInputFocus"
            class="input-filter input"
            v-model="state.filterValue"
            :placeholder="inputFilterPlaceholder"
          >
        </div>
      </div>
      <div class="input-box">
        <input
          readonly
          :disabled="props.disabled"
          type="text"
          class="input"
          ref="inputRef"
          @blur="onInputBlur"
          @focus="onInputFocus"
          :placeholder="inputWrapperPlaceholder"
          :style="{height: (props.filterable || (props.multiple && state.selectValue.length)) ? containerHeight : 'auto'}"
        >
        <div class="icon">
          <div class="icon-wrapper icon-arrow">
            <el-icon color="#a8abb2">
              <ArrowDown />
            </el-icon>
          </div>
          <div class="icon-wrapper icon-close">
            <el-icon color="#a8abb2" @click.stop="onClickClearValue">
              <CircleClose />
            </el-icon>
          </div>
        </div>
      </div>
    </div>
    <template #content>
      <template v-if="state.filterValue !== ''">
        <CustomContentSelectOptions
          key="options_1"
          v-if="filterDone"
          :options="filterList"
          :multiple="props.multiple"
          :select-value="state.selectValue"
          :update-method="onClickChangeChecked"
        >
          <template v-if="slots.content" #content="props">
            <slot name="content" v-bind="props"></slot>
          </template>
        </CustomContentSelectOptions>
        <div v-else class="custom-content-select-empty">加载中</div>
      </template>
      <CustomContentSelectOptions
        v-else
        key="options_2"
        :options="props.options"
        :multiple="props.multiple"
        :select-value="state.selectValue"
        :update-method="onClickChangeChecked"
      >
        <template v-if="slots.content" #content="props">
          <slot name="content" v-bind="props"></slot>
        </template>
      </CustomContentSelectOptions>
    </template>
  </SelectDropdown>
</div>
</template>
<style lang="scss" scoped>
.custom-content-select {
  position: relative;
  display: inline-block;
  vertical-align: middle;
  font-size: 14px;
  color: #606266;
  border-radius: 4px;
  box-shadow: 0 0 0 1px #dcdfe6 inset;
  cursor: pointer;
  background-color: #FFF;
  line-height: 30px;
  &.is-focus {
    box-shadow: 0 0 0 1px #035cff inset;
    .input-box .icon{
      .icon-wrapper.icon-arrow {
        transform: rotateZ(180deg);
      }
    }
  }
  &.disabled {
    background-color: #f5f7fa;
    box-shadow: 0 0 0 1px var(--el-disabled-border-color) inset;
    cursor: not-allowed;
    .input-box .input, .input-wrapper .input {
      cursor: not-allowed;
    }
  }
  .input {
    margin: 0;
    padding: 0;
    min-height: 30px;
    outline: none;
    border: none;
    background-color: transparent;
    &::placeholder {
      color: #a8abb2;
    }
  }
  .input-value {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    position: absolute;
    left: 0;
    top: 0;
    padding-top: 1px;
    padding-right: 30px;
    width: 100%;
    overflow: hidden;
    .input-wrapper {
      padding-left: 11px;
      flex: 1;
      .input {
        vertical-align: top;
        width: 100%;
      }
    }
    .tags {
      max-width: 100%;
      flex: none;
      padding-bottom: 3px;
      margin-left: 3px;
      margin-top: -1px;
      line-height: 1em;
      :deep(.el-tag) {
        max-width: 100%;
        margin-top: 4px;
        margin-left: 4px;
        .el-tag__content {
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap;
        }
      }
    }
    .text {
      flex: none;
      max-width: calc(100% - 20px);
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
      padding-left: 11px;
    }
  }
  .input-box {
    display: flex;
    align-items: center;
    padding: 1px 11px;
    .input {
      width: 100%;
      cursor: pointer;
    }
    .icon {
      flex: none;
      height: 100%;
      margin-left: 5px;
      .icon-wrapper {
        display: flex;
        align-items: center;
        &.icon-arrow {
          transform: rotateZ(0);
          transition: transform .15s linear;
        }
        &.icon-close {
          display: none;
        }
      }
    }
  }
  .select-trigger.clearable:hover {
    .input-box .icon .icon-wrapper {
      &.icon-arrow {
        display: none;
      }
      &.icon-close {
        display: flex;
      }
    }
  }
}
</style>
