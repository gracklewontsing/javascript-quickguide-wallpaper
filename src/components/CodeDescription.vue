<script setup lang="ts">
import { ref, watchEffect } from 'vue'
// Use the correct import for 'marked'
import { marked } from 'marked'
const fadeClass = ref('')
// Props for the markdown content
const props = defineProps<{
    description: string
}>()

// Reactive ref to hold the formatted description
const highlighted = ref('')

// Function to manually parse the Markdown
const parseMarkdown = async (markdown: string): Promise<string> => {
    // Parse the Markdown into HTML using marked
    const htmlContent = marked(markdown) // Use `marked` directly now

    // Optionally, we could add additional transformations here for code formatting
    return htmlContent
}

// Watch for changes to description and parse the markdown
watchEffect(async () => {
    highlighted.value = await parseMarkdown(props.description)
})
watchEffect(async () => {
    fadeClass.value = ''  // Reset
    highlighted.value = await parseMarkdown(props.description)
    // Force reflow to allow transition restart
    void document.body.offsetHeight
    fadeClass.value = 'fade-in'
})

</script>

<template>
    <!-- Apply transition to the container -->
    <div class="description-container" :class="fadeClass" v-html="highlighted"></div>
</template>

<style scoped>
.description-container {
    background-color: var(--background-color-darker);
    color: var(--color-description);
    padding: 1rem;
    border-radius: 10px;
    font-family: 'Segoe UI', sans-serif;
    line-height: 1.6;
    text-align: left;
    width: 100%;
    overflow-y: auto;
    /* Enables vertical scrolling */
}

.fade-in {
    animation: fadeIn 0.8s ease-in;
}

@keyframes fadeIn {
    from {
        opacity: 0;
    }

    to {
        opacity: 1;
    }
}

/* Markdown element styling */
.description-container :deep(h1),
.description-container :deep(h2),
.description-container :deep(h3) {
    color: var(--color-description-header);
    margin-top: 1rem;
}

.description-container :deep(h1) {
    font-size: 1.75rem;
    /* Smaller font size for h1 */
}

.description-container :deep(h2) {
    font-size: 1.5rem;
    /* Smaller font size for h2 */
}

.description-container :deep(h3) {
    font-size: 1.25rem;
    /* Smaller font size for h3 */
}

.description-container p {
    margin-bottom: 0.75rem;
}

.description-container p {
    margin-bottom: 0.75rem;
}

.description-container code {
    background-color: var(--background-color-lighter);
    color: #c586c0;
    padding: 2px 6px;
    border-radius: 4px;
    font-family: 'Fira Code', monospace;
}

.description-container pre {
    background-color: var(--background-color-lighter);
    padding: 0.75rem;
    border-radius: 6px;
    overflow-x: auto;
    margin: 1rem 0;
}
</style>
