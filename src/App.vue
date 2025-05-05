<script setup lang="ts">
import { onMounted, ref } from 'vue';

declare global {
  interface Window {
    wallpaperPropertyListener?: {
      applyUserProperties: (properties: any) => void;
    };
  }
}
import CodeSelector from './components/CodeSelector.vue';
const images = import.meta.glob('/src/assets/*.svg', { eager: true, import: 'default' })

interface Snippet {
  title: string;
  fileName: string;
  fileType: string;
  code: string;
  description: string;
  output: string;
}

// Store the list of languages for which we have logos and JSON files
const languages = ref([
  'vuejs',
  'reactjs',
  'angular',
  'javascript',
  'typescript',
  'jasmine'
]);

const getLogoSrc = (lang: string) => {
  return images[`/src/assets/${lang}.svg`] as string || '';
};

const showEasterEgg = ref(false);

const triggerEasterEggEasterEgg = async () => {
  showEasterEgg.value = true;
  await loadLanguageData('easteregg');
  if (!languages.value.includes('easteregg')) {
    const middleIndex = Math.floor(languages.value.length / 2);
    languages.value.splice(middleIndex, 0, 'easteregg');
  }
};

const languageData = ref<any>(null);
const snippetData = ref<Snippet>({
  title: 'Hello, world!',
  fileName: 'example',
  fileType: '.js',
  code: `const greeting = 'Hello, world!';\nconsole.log(greeting);`,
  description: `# Code Example\nThis is a simple JavaScript code snippet that prints a greeting message.`,
  output: 'Hello, world!'
});

// Track the selected language
const selectedLanguage = ref(languages.value[0]);
const currentSnippetIndex = ref(0);

//wallpaper engine properties
const theme = ref('dark'); // default theme
const backgroundImageUrl = ref(''); // default background image URL
const backgroundSize = ref('cover'); // default background size
const backgroundPositionX = ref('50%'); // default background position X
const backgroundPositionY = ref('50%'); // default background position Y


// Save selected language and snippet to localStorage
const saveToLocalStorage = () => {
  localStorage.setItem('selectedLanguage', selectedLanguage.value);
  localStorage.setItem('currentSnippetIndex', currentSnippetIndex.value.toString());
};

// Load selected language and snippet from localStorage
const loadFromLocalStorage = async () => {

  const savedLanguage = localStorage.getItem('selectedLanguage');
  const savedSnippetIndex = localStorage.getItem('currentSnippetIndex');

  if (savedLanguage) {
    if (savedLanguage == 'easteregg') {
      triggerEasterEggEasterEgg()
    }
    await loadLanguageData(savedLanguage || selectedLanguage.value);
  }
  if (savedSnippetIndex) {
    handleSnippetChange(savedSnippetIndex ? +savedSnippetIndex : 0);
  }
};

const loadLanguageData = async (language: string) => {
  currentSnippetIndex.value = 0;

  try {
    const module = await import(`./assets/${language}.json`);
    languageData.value = module.default; // Assuming JSON files export an object with the structure you need

    // Update snippetData based on loaded language data
    snippetData.value = {
      title: languageData.value.snippets[0].title,
      fileName: languageData.value.snippets[0].fileName,
      fileType: languageData.value.snippets[0].fileType,
      code: languageData.value.snippets[0].code,
      description: languageData.value.snippets[0].description,
      output: languageData.value.snippets[0].output
    };

    // Update the selected language
    selectedLanguage.value = language;

    // Save to localStorage after loading new language
    saveToLocalStorage();
  } catch (err) {
    console.error(`Error loading ${language} JSON:`, err);
  }

  if (language !== 'easteregg') {
    const index = languages.value.indexOf('easteregg');
    if (index !== -1) {
      languages.value.splice(index, 1);
    }
    showEasterEgg.value = false;
  }
};

const handleSnippetChange = (action: string | number) => {
  const snippets = languageData.value?.snippets || [];
  if (!snippets.length) return;

  // Handle case where action is an index (a number)
  if (typeof action === 'number') {
    if (action >= 0 && action < snippets.length) {
      currentSnippetIndex.value = action;
    } else {
      console.error("Invalid snippet index");
      return;
    }
  } else {
    // Handle other actions (previous, next, random, first)
    switch (action) {
      case 'previous':
        currentSnippetIndex.value = (currentSnippetIndex.value - 1 + snippets.length) % snippets.length;
        break;
      case 'next':
        currentSnippetIndex.value = (currentSnippetIndex.value + 1) % snippets.length;
        break;
      case 'random':
        let randomIndex;
        do {
          randomIndex = Math.floor(Math.random() * snippets.length);
        } while (randomIndex === currentSnippetIndex.value && snippets.length > 1);
        currentSnippetIndex.value = randomIndex;
        break;

      default:
        console.error("Invalid action");
        return;
    }
  }

  // Update snippet data based on currentSnippetIndex
  const snippet = snippets[currentSnippetIndex.value];
  snippetData.value = {
    title: snippet.title,
    fileName: snippet.fileName,
    fileType: snippet.fileType,
    code: snippet.code,
    description: snippet.description,
    output: snippet.output
  };

  // Save to localStorage after snippet change
  saveToLocalStorage();
};

const applyTheme = (themeName: string) => {  
  document.documentElement.setAttribute('data-theme', themeName);
  theme.value = themeName;
};

// Set background image and properties
const updateBackgroundProperties = (
  imageUrl = backgroundImageUrl.value,
  size = backgroundSize.value,
  posX = backgroundPositionX.value,
  posY = backgroundPositionY.value
) => {
  const rootElement = document.documentElement;

  // Update CSS variables for background properties
  rootElement.style.setProperty('background-image', imageUrl ? `url(${imageUrl})` : 'none');
  rootElement.style.setProperty('background-size', size);
  rootElement.style.setProperty('background-position-x', posX);
  rootElement.style.setProperty('background-position-y', posY);

  // Update reactive state variables
  backgroundImageUrl.value = imageUrl;
  backgroundSize.value = size;
  backgroundPositionX.value = posX;
  backgroundPositionY.value = posY;
};

// Wallpaper Engine property listener setup
const setupWallpaperEngineListeners = () => {
  applyTheme('light');
  window.wallpaperPropertyListener = {
    applyUserProperties: function (properties) {
      // Handle theme property
      if (properties.theme) {
        const newTheme = properties.theme.value;
        applyTheme(newTheme);
      }

      // Handle background image property
      if (properties.backgroundimage) {
        const imagePath = properties.backgroundimage.value
          ? 'file:///' + properties.backgroundimage.value
          : '';
        updateBackgroundProperties(imagePath);
      }

      // Handle background size property
      if (properties.backgroundsize) {
        updateBackgroundProperties(
          backgroundImageUrl.value,
          properties.backgroundsize.value
        );
      }

      // Handle background position X property
      if (properties.backgroundpositionx) {
        const posX = properties.backgroundpositionx.value + '%';
        updateBackgroundProperties(
          backgroundImageUrl.value,
          backgroundSize.value,
          posX
        );
      }

      // Handle background position Y property
      if (properties.backgroundpositiony) {
        const posY = properties.backgroundpositiony.value + '%';
        updateBackgroundProperties(
          backgroundImageUrl.value,
          backgroundSize.value,
          backgroundPositionX.value,
          posY
        );
      }
    }
  };
};



onMounted(async () => {
  // Load last used language and snippet from localStorage

  if (localStorage.getItem('selectedLanguage') === null && localStorage.getItem('currentSnippetIndex') === null) {
    // If no language is saved, load the default language    
    await loadLanguageData(selectedLanguage.value);
  } else {
    // Load the last used language and snippet
    loadFromLocalStorage();
  }

  setupWallpaperEngineListeners();
});


</script>



<template>
  <div>
    <!-- Logos as clickable buttons -->
    <transition-group name="fade" tag="div" class="logos">
      <div v-for="lang in languages" :key="lang" class="logo-container"
        :class="{ 'selected': lang === selectedLanguage }">
        <img :src="getLogoSrc(lang)" :alt="`${lang} logo`" class="logo" @click="loadLanguageData(lang)" />
      </div>
    </transition-group>
    <div class="hidden-trigger" @click="triggerEasterEggEasterEgg">
    </div>
    <!-- Pass the loaded language data to the CodeSelector component -->
    <CodeSelector class="selector" @changeSnippet="handleSnippetChange" :data="snippetData"
      :language="languageData?.language || 'Vue.js'"
      :languageDocumentationUrl="languageData?.documentationUrl || 'https://vuejs.org/guide/introduction.html'"
      :snippets="languageData?.snippets || []" :theme="theme || 'dark'" />
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  width: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
  cursor: pointer;
}

.logo:hover {
  filter: drop-shadow(0 0 1em #6a71fae8);
  transition: filter 500ms;
}

.logos {
  display: flex;
  gap: 2rem;
  justify-content: center;
  margin-bottom: 1rem;
}

.logo-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 0.5rem;
  border-radius: 50%;
  transition: background-color 1s, border 1s;
}

.selected {
  background-color: var(--selected-background-color);
  filter: drop-shadow(0 0 1em #6ab9fa57);
}

.selected .logo {
  filter: none;
  /* You can disable hover effect or adjust it for the selected state */
}

.hidden-trigger {
  position: absolute;
  top: 0;
  left: 0;
  border-radius: 50%;
  width: 1rem;
  height: 1rem;
  opacity: 0;
  /*z-index: 100;*/
}

.hidden-trigger:hover {
  cursor: grab;
}

/* Optional: Animate EasterEgg logo differently */
.logo-container.selected img[alt~="easteregg"] {
  filter: drop-shadow(0 0 1em #478cbf);
  animation: eastereggPulse 2s infinite alternate;
}

@keyframes eastereggPulse {
  from {
    transform: scale(1);
    filter: drop-shadow(0 0 1em #478cbf);
  }

  to {
    transform: scale(1.1);
    filter: drop-shadow(0 0 2em #478cbf);
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: all 0.6s ease;
  position: relative;
  /* helps with smoother stacking */
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: scale(0.9);
}

.fade-enter-to,
.fade-leave-from {
  opacity: 1;
  transform: scale(1);
}

/* When an element is removed from a transition-group, Vue applies this class */
.fade-leave-active {
  position: absolute;
  z-index: -1;
}

/* Movement transition (when the other elements shift around) */
.fade-move {
  transition: transform 0.6s ease;
}
</style>