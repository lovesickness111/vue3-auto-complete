<template>
  <div class="p-relative">
    <div class="formula-editor" contenteditable ref="editorRef" @input="onInput" @keydown="onKeyDown" v-html="formula"
      dir="ltr">

    </div>
    <div class="suggestions">
      <ul v-if="showSuggestions">
        <li :class="{ 'hightlight': suggestionIndex == index }" v-for="(suggestion, index) in filteredSuggestions"
          :key="index" @mousedown.prevent="onSuggestionClick(suggestion)">
          {{ suggestion }}
        </li>
      </ul>
    </div>
  </div>
</template>
  
<script setup>
import { ref, watch, onMounted, onUnmounted } from 'vue';

const props = defineProps({
  formula: {
    type: String,
    default: '',
  },
  suggestions: {
    type: Array,
    default: () => ['lovesickness', 'cuong', 'SUM', 'Suites  '],
  },
});
const emit = defineEmits(['onUpdateFormula']);

const editorRef = ref(null);
const formula = ref(props.formula);
const showSuggestions = ref(false);
const suggestionIndex = ref(-1);
const filteredSuggestions = ref([]);

const onInput = () => {
  const text = editorRef.value.innerHTML;
  //formula.value = text;
  emit('onUpdateFormula', text);
  const currentWord = getCurrentWord();
  filteredSuggestions.value = currentWord ? props.suggestions.filter((suggestion) => suggestion.toUpperCase().startsWith(currentWord.toUpperCase())) : [];
  showSuggestions.value = filteredSuggestions.value.length > 0;
};

const getCurrentWord = () => {
  const selection = window.getSelection();
  const range = selection.getRangeAt(0);
  const text = range.endContainer.textContent.slice(0, range.startOffset);
  const words = text.split(/\s+/);
  return words[words.length - 1];
};
const onKeyDown = (event) => {
  if (event.key === 'ArrowUp') {
    event.preventDefault();
    suggestionIndex.value =
      suggestionIndex.value <= 0 ? filteredSuggestions.value.length - 1 : suggestionIndex.value - 1;
  } else if (event.key === 'ArrowDown') {
    event.preventDefault();
    suggestionIndex.value =
      suggestionIndex.value >= filteredSuggestions.value.length - 1 ? 0 : suggestionIndex.value + 1;
  } else if (event.key === 'Enter' || event.key === 'Tab') {
    if (suggestionIndex.value !== -1) {
      event.preventDefault();
      onSuggestionClick(filteredSuggestions.value[suggestionIndex.value]);
    }
  } else {
    showSuggestions.value = filteredSuggestions.value.length > 0;
    if (showSuggestions.value) {
      suggestionIndex.value = 0;

    } else {
      suggestionIndex.value = -1;
    }
  }
};

const onSuggestionClick = (suggestion) => {
  const selection = window.getSelection();
  const range = selection.getRangeAt(0);
  const text = editorRef.value.innerHTML;
  const start = range.startOffset - getCurrentWord().length;
  const end = range.startOffset;
  const replacedText = text.slice(0, start) + `<text class='hightlight'>` + suggestion + `</text>` + text.slice(end);
  editorRef.value.innerHTML = replacedText;
  // // formula.value = editorRef.value.innerHTML;

  // emit('onUpdateFormula', editorRef.value.innerHTML);

  // showSuggestions.value = false;

  // // Set the selection to the end of the replaced text
  // // debugger
  // const nodeLength = editorRef.value.childNodes.length;
  // const newRange = new Range();
  // newRange.setStart(editorRef.value,  suggestion.length);
  // newRange.setEnd(editorRef.value, suggestion.length);
  // selection.removeAllRanges();
  // selection.addRange(newRange);

  const editor = editorRef.value;

  // get the range of the text between 
  const textNode = editor.lastElementChild;
  range.setStart(textNode, 1);
  range.setEnd(textNode, 1);

  // set the typing cursor to the end of the range
  range.collapse(false);
  selection.removeAllRanges();
  selection.addRange(range);
  suggestionIndex.value = -1;

};
watch(
  () => props.suggestions,
  () => {
    const prefix = editorRef.value.innerHTML.slice(0, editorRef.value.selectionStart).replace(/.*\b(\w+)$/, '$1').toUpperCase();
    filteredSuggestions.value = prefix ? props.suggestions.filter((suggestion) => suggestion.toUpperCase().startsWith(prefix)) : [];
    showSuggestions.value = filteredSuggestions.value.length > 0;
  }
);

onMounted(() => {
  editorRef.value.focus();
});

onUnmounted(() => {
  editorRef.value.blur();
});
</script>
<style>
.p-relative {
  position: relative;

}

.formula-editor {
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 8px;
  min-height: 40px;
  cursor: text;
}

.suggestions {
  position: absolute;
  bottom: -66px;
}

ul {
  list-style: none;
  padding: 0;
  margin: 4px 0 0 0;
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 4px;
  max-height: 120px;
  overflow-y: auto;
}

li {
  padding: 4px 8px;
  cursor: pointer;
}

.hightlight {
  background-color: aqua;
}

li:hover {
  background-color: #f2f2f2;
}
</style>