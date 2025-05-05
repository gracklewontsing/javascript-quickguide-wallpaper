<template>
  <div class="code-window">
    <div class="code-tab">
      <span>{{ tabName }}</span>
    </div>
    <pre
      class="code-body"><code v-html="highlighted"></code></pre>
    <transition name="fade-expand">
      <div class="output" v-if="props.output && showOutput" :key="output">
        <div style="font-family: Arial, Helvetica, sans-serif; margin-bottom: 2px">Output:</div>
        <div>{{ output }}</div>
      </div>
    </transition>
  </div>
</template>

<script setup lang="ts">
import { ref, watchEffect, watch} from 'vue'
import Prism from 'prismjs'
import 'prismjs/components/prism-typescript'
import 'prismjs/components/prism-javascript'
import 'prismjs/components/prism-markup'
import 'prismjs/components/prism-css'
import 'prismjs/components/prism-gdscript' // GDScript Easter Egg

const props = defineProps<{
  tabName: string
  code: string
  language?: string
  output?: string
}>()

const highlighted = ref('')
const rawHighlighted = ref('')
const showOutput = ref(false) // Local trigger for output visibility

const isTyping = ref(false);
const isDeleting = ref(false);
const isFinished = ref(true);
const cancelTyping = ref(false);
const cancelDeleting = ref(false);


function extractText(htmlString: string) {
  const tempDiv = document.createElement('div');
  tempDiv.innerHTML = htmlString;
  return tempDiv.textContent || '';
}

async function startTyping(newVal: string) {  
  isTyping.value = true;
  isFinished.value = false;

  const plainNew = extractText(newVal);
  const baseTypeDelay = 150;
  const lengthFactor = plainNew.length < 100 ? 100 : plainNew.length;
  const typeDelay = Math.ceil(baseTypeDelay / Math.sqrt(lengthFactor));
  
  for (let i = 0; i <= plainNew.length; i++) {
    highlighted.value = Prism.highlight(
      plainNew.slice(0, i),
      Prism.languages[props.language || 'javascript'],
      props.language || 'javascript'
    );
    await new Promise(resolve => setTimeout(resolve, typeDelay));

    // If the snippet changes mid-typing, stop and reset
    if (newVal !== rawHighlighted.value) {
      cancelTyping.value = true;
      break;
    }
  }
  cancelDeleting.value = false;
  if (!cancelTyping.value) {
    isFinished.value = true;
    isTyping.value = false;    
  }
}

async function startDeleting() {  
  isDeleting.value = true;
  
  let plainCurrent = extractText(highlighted.value);

  const baseDeleteDelay = 5;
  const lengthFactor = plainCurrent.length > 100 ? 1: plainCurrent.length;  
  const deleteDelay = Math.max(1, baseDeleteDelay - (baseDeleteDelay / Math.sqrt(lengthFactor)));

  for (let i = plainCurrent.length; i >= 0; i--) {
    if (cancelDeleting.value) {
      plainCurrent = ''
      isDeleting.value = false;
      highlighted.value = '';
      isFinished.value = true;      
      cancelDeleting.value = false;
      break
    }
    highlighted.value = Prism.highlight(
      plainCurrent.slice(0, i),
      Prism.languages[props.language || 'javascript'],
      props.language || 'javascript'
    );
    await new Promise(resolve => setTimeout(resolve, deleteDelay));
  }

  isDeleting.value = false;
  isFinished.value = true;
  showOutput.value = false;
}

watch(rawHighlighted, async (newVal) => {
  if (isFinished.value) {
    await startTyping(newVal);
    showOutput.value = true;
  } else if (isTyping.value && !isDeleting.value) {
    cancelTyping.value = true;
    showOutput.value = false;
    await startDeleting();
    cancelDeleting.value = false;
    // Recursively call typing again if rawHighlighted has content
    if (rawHighlighted.value !== '') {
      await startTyping(rawHighlighted.value);
      showOutput.value = true;
    }
  } else if (isDeleting.value) {
    cancelDeleting.value = true;
    showOutput.value = false;
    await startTyping(newVal);
    showOutput.value = true;
  }
});


watchEffect(() => {
  const lang = props.language || 'javascript'
  const grammar = Prism.languages[lang]
  if (grammar) {
    rawHighlighted.value = Prism.highlight(props.code.trimStart(), grammar, lang)
  } else {
    rawHighlighted.value = props.code
  }
})

</script>


<style scoped>
select {
  pointer-events: auto;
}

.code-window {
  display: flex;
  flex-direction: column;
  height: 100%;
  border: 1px solid var(--background-color-lighter);
  background-color: var(--background-color-darker);
  color: var(--color-description);
  font-family: 'Fira Code', 'Courier New', monospace;
  border-radius: 8px;
  overflow: hidden;
  width: 100%;
  box-shadow: 0 0 10px #00000040;
}

.code-tab {
  background: var(--background-color-lighter);
  padding: 0.5rem 1rem;
  font-weight: bold;
  border-bottom: 1px solid var(--code-window-border);
}

.code-body {
  padding: 1rem;
  overflow-x: auto;
  overflow-y: auto;
  /* Enables vertical scrolling */
  flex-grow: 1;
  /* Takes up remaining space */
}

.output {
  background: var(--background-color-lighter);
  color: var(--color-description);
  padding: 1rem;
  border-top: 1px solid var(--code-window-border);
  overflow-y: scroll;
  height: 60px;
  font-size: 12px;
  /* Set a fixed height for the output */
}

.output h3 {
  margin-top: 0;
}

.fade-expand-enter-active {
  transition: opacity 0.5s ease, max-height 0.5s ease, margin 0.5s ease, padding 0.5s ease;
}

.fade-expand-leave-active {
  transition: opacity 0.3s ease, max-height 0.3s ease, margin 0.3s ease, padding 0.3s ease;
}

.fade-expand-enter-from,
.fade-expand-leave-to {
  opacity: 0;
  max-height: 0;
  margin-top: 0;
  padding-top: 0;
  padding-bottom: 0;
}

.fade-expand-enter-to,
.fade-expand-leave-from {
  opacity: 1;
  max-height: 100px;
  /* Adjust as needed based on your output height */
}
</style>
