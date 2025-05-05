<template>
  <div class="header">
    <div class="button-group">
      <!-- Add the "Go to" dropdown here -->
      <label for="go-to-snippet">Go to:</label>
      <Dropdown id="go-to-snippet" :selected="selectedSnippet" :snippets="snippets" @selectSnippet="goToSnippet" />
    </div>
    <Transition name="fade" mode="out-in">
      <h1 :key="data.title">{{ data.title }}</h1>
    </Transition>
    <div class="button-group-right">
      <button @click="goToPreviousSnippet">Previous</button>
      <button @click="goToNextSnippet">Next</button>
      <button @click="goToRandomSnippet">Random</button>
    </div>
  </div>

  <div class="side-by-side">
    <CodeWindow @resetCode="onChangeSnippet" :tabName="example + fileType" :code="code || ''" :output="data.output">
    </CodeWindow>
    <CodeDescription @resetData="onChangeSnippet" :description="description"></CodeDescription>
  </div>

  <p style="text-align: center;">
    Learn more about {{ language }} in the
    <button @click="copyToClipboard(languageDocumentationUrl)"
      style="background: none; border: none; color: #007bff; cursor: pointer; position: relative; padding:0; margin:1px 0 0 0;">
      {{ language }} Documentation URL
      <Transition name="fade" mode="out-in">
        <div v-if="showToast" class="tooltip">
          Copied to clipboard!
        </div>
      </Transition>
    </button>.
  </p>
  <div class="texts">
    <p class="read-the-docs">Click on the logos to swap to a specific language.</p>
    <button class="reload-button" @click="reloadPage" title="reload the app"
      style="background-color: transparent !important;">
      <svg xmlns="http://www.w3.org/2000/svg" height="20" width="20" viewBox="0 0 20 20" fill="currentColor">
        <path
          d="M12 4V1L8 5l4 4V6c3.31 0 6 2.69 6 6 0 1.31-.42 2.52-1.13 3.5l1.46 1.46C19.11 15.08 20 13.13 20 11c0-4.42-3.58-8-8-8zm-6.87 2.13L3.66 7.66C2.89 8.92 2.5 10.41 2.5 12c0 4.42 3.58 8 8 8v3l4-4-4-4v3c-3.31 0-6-2.69-6-6 0-1.31.42-2.52 1.13-3.5z" />
      </svg>
    </button>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import CodeWindow from './CodeWindow.vue'
import CodeDescription from './CodeDescription.vue';
import Dropdown from './Dropdown.vue';

const props = defineProps<{
  language?: string
  languageDocumentationUrl?: string
  data: {
    title: string
    fileName?: string
    fileType?: string
    code?: string
    description?: string
    output?: string
  },
  theme?: string
  snippets: Array<{ title: string }>
}>()

const emit = defineEmits<{
  (event: 'changeSnippet', action: string | number): void
}>()

const language = computed(() => props.language || 'JavaScript')

const languageDocumentationUrl = computed(() => {
  return props.languageDocumentationUrl || `https://developer.mozilla.org/en-US/docs/Web/JavaScript`
})

const example = computed(() => {
  return props.data.fileName || 'HelloWorld'
})

const fileType = computed(() => {
  return props.data.fileType || '.js'
})

const code = computed(() => {
  return props.data.code
})

const description = computed(() => {
  return props.data.description || `# Code Example\nThis is a simple JavaScript snippet showing how to log a greeting to the console.\n\`\`\`js\nconst greeting = 'Hello, world!';\nconsole.log(greeting);\n\`\`\``
})

const showToast = ref(false)

const selectedSnippet = ref('')

const copyToClipboard = (text: string) => {
  navigator.clipboard.writeText(text)
    .then(() => {
      showToast.value = true;
      setTimeout(() => {
        showToast.value = false;
      }, 2000);
    })
    .catch(err => {
      console.error('Failed to copy: ', err);
    });
}

const goToSnippet = (title: string) => {
  // Find the snippet object based on the selected title
  const snippet = props.snippets.find(snippet => snippet.title === title);
  // If the snippet is found, get its index and emit it
  if (snippet) {
    const snippetIndex = props.snippets.indexOf(snippet); // Get the index of the snippet
    emit('changeSnippet', snippetIndex);  // Emit the index instead of the snippet object
  } else {
    console.error("Snippet not found");
  }
};

const onChangeSnippet = (index: number) => {
  const snippet = props.snippets[index];
  // Update the selectedSnippet to the title of the new snippet
  selectedSnippet.value = snippet.title;

}

const goToPreviousSnippet = () => {
  emit('changeSnippet', 'previous')
}

const goToNextSnippet = () => {
  emit('changeSnippet', 'next')
}

const goToRandomSnippet = () => {
  emit('changeSnippet', 'random')
}
const reloadPage = () => {
  window.location.reload();
};

// Watch for changes to the `selectedSnippet` to keep the dropdown in sync
watch(() => props.data.title, (newTitle) => {
  selectedSnippet.value = newTitle;
});
</script>

<style scoped>
.read-the-docs {
  color: var(--read-the-docs-color);
  text-align: center;
}

.side-by-side {
  display: flex;
  justify-content: center;
  align-items: stretch;
  gap: 2rem;
  height: 55vh;
}

CodeWindow,
CodeDescription {
  flex: 1;
  height: 100%;
  display: flex;
  flex-direction: column;
}

CodeWindow,
CodeDescription {
  width: 100% !important;
  max-height: 60vh;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.header h1 {
  font-size: 2.5rem !important;
  color: var(--code-selector-header-color);
  margin: 1rem;
}

.button-group {
  display: flex;
  gap: 10px;
  align-items: center;
}

button {
  outline: none;
}

select {
  z-index: 1000;
}

.button-group-right {
  display: flex;
  gap: 10px;
  align-items: center;
}

button {
  padding: 5px 15px;
  background-color: var(button-background-color);
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 5px;
  height: 30px;
}

button:hover {
  background-color: var(button-background-color-hover);
}

select {
  padding: 5px;
  border-radius: 3px;
  margin-left: 10px;
}

.reload-button {
  display: flex;
  justify-self: center !important;
  align-self: center !important;
  color: var(--reload-button-color);
  font-size: 8px;
  border: none !important;
  border-radius: 50%;
  cursor: pointer;
  z-index: 200;
  /* higher than hidden-trigger */
}

.reload-button:hover {  
  color: var(--reload-button-color-hover);
}

.toast {
  position: fixed;
  bottom: 20px;
  right: 20px;
  background-color: var(--code-window-border);
  color: white;
  padding: 10px 20px;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  z-index: 1000;
}

.tooltip {
  position: absolute;
  top: -30px;
  left: 50%;
  transform: translateX(-50%);
  background-color: var(--code-window-border);
  color: white;
  padding: 5px 10px;
  border-radius: 5px;
  font-size: 12px;
  white-space: nowrap;
  z-index: 1000;
}

.texts {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: -20px;
}
</style>
