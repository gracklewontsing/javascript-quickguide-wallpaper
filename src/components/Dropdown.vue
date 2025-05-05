<template>
    <div class="dropdown" @click="toggleDropdown" tabindex="0" @blur="closeDropdown">
        <div class="selected">
            {{ selected || 'Choose a snippet' }}
        </div>
        <ul v-show="isDropdownOpen" class="dropdown-menu">
            <li v-for="(snippet, index) in snippets" :key="index" @click.stop="selectSnippet(snippet.title)">
                {{ snippet.title }}
            </li>
        </ul>
    </div>
</template>
<script setup lang="ts">
import { ref } from 'vue';

// Define props with correct types
defineProps<{
    snippets: Array<{ title: string }>;
    selected: string | null;
}>();

// Define emits with correct types
const emit = defineEmits<{
    (event: 'selectSnippet', snippet: string): void;
}>();

const selectedSnippet = ref<string | null>(null);
const isDropdownOpen = ref<boolean>(false);

function toggleDropdown(): void {
    isDropdownOpen.value = !isDropdownOpen.value;
}

function closeDropdown(): void {
    isDropdownOpen.value = false;
}

function selectSnippet(snippet: string): void {
    selectedSnippet.value = snippet;
    emit('selectSnippet', snippet);  // Emit event with snippet
    closeDropdown();
}
</script>

<style scoped>
.dropdown {
    position: relative;
    display: inline-block;
    cursor: pointer;
    user-select: none;
    padding: 4px 8px;
    background: #242424;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 12px;
    width: 250px;
    /* Fixed width to prevent wrapping */
}

.selected {
    min-width: 120px;
    font-size: 12px;
    white-space: nowrap;
    /* Prevent text wrapping */
    overflow: hidden;
    text-overflow: ellipsis;
    /* Adds '...' when text is too long */
}

.dropdown-menu {
    margin: 0;
    padding: 0;
    list-style: none;
    position: absolute;
    top: 100%;
    left: 0;
    right: 0;
    background: #242424;
    border: 1px solid #ccc;
    z-index: 1000;
    font-size: 12px;
    min-width: 99%;
    max-height: 600px;
    overflow-y: auto;
}

ul {
    margin: 0;
    padding: 0;
    width: 180px;
}

.dropdown-menu li {
    padding: 4px 8px;
    white-space: nowrap;
    max-width:100%
    /* Prevent text wrapping in list items */
}

.dropdown-menu li:hover {
    background-color: #2c2c2c;
}
</style>