<template>
  <component
    :is="tag"
    :contenteditable="contenteditable"
    @input="update"
    @blur="update"
    @paste="onPaste"
    @keypress="onKeypress"
    ref="element"
  >
  </component>
</template>

<script setup lang="ts">
import { defineProps, ref, computed, onMounted, watch } from "vue";

function replaceAll(str: string, search: string, replacement: string) {
  return str.split(search).join(replacement);
}

const props = defineProps({
  tag: {
    type: [String],
    default: "div",
  },
  contenteditable: {
    type: [Boolean, String],
    default: true,
  },
  modelValue: {
    type: [String],
    default: "",
  },
  noHtml: {
    type: Boolean,
    default: true,
  },
  noNl: {
    type: Boolean,
    default: false,
  },
});

const emit = defineEmits({
  returned: String,
  "update:modelValue": String,
});

const element = ref<HTMLElement | null>();

function contentGet() {
  return props.noHtml ? element.value!.innerText : element.value!.innerHTML;
}

function contentSet(newcontent: string) {
  if (props.noHtml) {
    element.value!.innerText = newcontent;
  } else {
    element.value!.innerHTML = newcontent;
  }
}

function update(event: any) {
  emit("update:modelValue", contentGet());
}

function onPaste(event: any) {
  event.preventDefault();
  let text = (event.originalEvent || event).clipboardData.getData("text/plain");
  if (props.noNl) {
    text = replaceAll(text, "\r\n", " ");
    text = replaceAll(text, "\n", " ");
    text = replaceAll(text, "\r", " ");
  }
  window.document.execCommand("insertText", false, text);
}
function onKeypress(event: any) {
  if (event.key == "Enter" && props.noNl) {
    event.preventDefault();
    emit("returned", contentGet());
  }
}

onMounted(() => {
  contentSet(props.modelValue ?? "");
});

watch(
  () => props.modelValue,
  (newval, oldval) => {
    if (newval != contentGet()) {
      contentSet(newval ?? "");
    }
  }
);

watch(
  () => props.noHtml,
  (newval, oldval) => {
    contentSet(props.modelValue ?? "");
  }
);

watch(
  () => props.tag,
  (newval, oldval) => {
    contentSet(props.modelValue ?? "");
  },
  { flush: "post" }
);
</script>
