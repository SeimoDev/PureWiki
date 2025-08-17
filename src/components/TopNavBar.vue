<template>
  <header class="top-nav-bar">
    <div class="left-section">
      <!-- Hamburger Menu Toggle (Mobile Only) -->
      <button v-if="isMobileView" @click="emitToggleMenu" class="mobile-menu-toggle" aria-label="Toggle navigation menu">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="3" y1="12" x2="21" y2="12"></line><line x1="3" y1="6" x2="21" y2="6"></line><line x1="3" y1="18" x2="21" y2="18"></line></svg>
      </button>

      <router-link to="/" class="logo-container-link">
        <div class="logo-container">
          <img v-if="logoSrc" :src="logoSrc" alt="Logo" class="logo-img" />
          <span class="site-title">{{ siteTitle }}</span>
          <span v-if="showBetaTag" class="beta-tag">Beta</span>
        </div>
      </router-link>
    </div>

    <!-- Desktop Search -->
    <div v-if="!isMobileView" class="center-section">
      <div class="search-container">
        <input
          type="search"
          v-model.trim="searchQuery"
          placeholder="搜索文章..."
          class="search-input"
          @focus="handleFocus"
          @blur="handleBlur"
        />
        <ul v-if="showResults && searchQuery && searchResults.length > 0" class="search-results">
          <li v-for="result in searchResults" :key="result.path">
            <router-link :to="result.path" @mousedown.prevent="navigateToResult(result.path)" class="search-result-link">
               <div class="result-title">{{ result.title }}</div>
               <div v-if="result.highlightedContext" class="result-context" v-html="result.highlightedContext"></div>
            </router-link>
          </li>
        </ul>
         <div v-else-if="showResults && searchQuery && searchResults.length === 0" class="search-no-results">
            无匹配结果
          </div>
      </div>
    </div>

    <div class="right-section">
      <!-- Mobile Search Toggle -->
      <button v-if="isMobileView" @click="toggleMobileSearch" class="search-toggle-button" aria-label="Toggle search">
         <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line></svg>
      </button>

      <!-- Desktop Nav Links -->
      <nav v-if="!isMobileView" class="nav-links">
        <template v-for="link in navLinks" :key="link.url">
          <!-- 内部链接使用 router-link -->
          <router-link v-if="isInternalLink(link.url)" :to="link.url" class="nav-link">
            {{ link.text }}
          </router-link>
          <!-- 外部链接使用 a 标签 -->
          <a v-else :href="link.url" target="_blank" rel="noopener noreferrer" class="nav-link external-link">
            {{ link.text }}
          </a>
        </template>
      </nav>

       <!-- Mobile Nav Links Dropdown Toggle -->
       <div v-if="isMobileView" class="mobile-nav-links-container">
         <button @click="toggleMobileNavLinksDropdown" class="mobile-nav-links-toggle" aria-label="Toggle navigation links">
           <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="1"></circle><circle cx="12" cy="5" r="1"></circle><circle cx="12" cy="19" r="1"></circle></svg>
         </button>
         <!-- Mobile Nav Links Dropdown - Wrapped with transition -->
         <transition name="nav-links-dropdown-anim">
           <nav v-show="isMobileNavLinksDropdownOpen" class="mobile-nav-dropdown" @click="closeMobileNavLinksDropdownOnClickInside">
             <template v-for="link in navLinks" :key="link.url">
               <!-- 内部链接使用 router-link -->
               <router-link v-if="isInternalLink(link.url)" :to="link.url" class="nav-link">
                 {{ link.text }}
               </router-link>
               <!-- 外部链接使用 a 标签 -->
               <a v-else :href="link.url" target="_blank" rel="noopener noreferrer" class="nav-link external-link">
                 {{ link.text }}
               </a>
             </template>
           </nav>
         </transition>
       </div>

      <!-- Dark Mode Toggle Icon -->
      <div @click="emitToggleDarkMode" class="dark-mode-toggle" :aria-label="isDarkMode ? 'Switch to light mode' : 'Switch to dark mode'" role="button">
         <transition name="fade" mode="out-in">
          <svg v-if="isDarkMode" key="moon" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-moon"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path></svg>
          <svg v-else key="sun" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-sun"><circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line></svg>
        </transition>
      </div>
    </div>

    <!-- Mobile Search Overlay - Wrapped with transition -->
    <transition name="mobile-search-anim">
      <div v-if="isMobileSearchActive" class="mobile-search-overlay">
        <div class="mobile-search-header">
          <input
            ref="mobileSearchInput"
            type="search"
            v-model.trim="searchQuery"
            placeholder="搜索文章..."
            class="mobile-search-input"
            @focus="handleFocus"
            @blur="handleBlur"
          />
          <button @click="closeMobileSearch" class="mobile-search-close-button" aria-label="Close search">×</button>
        </div>
          <ul v-if="showResults && searchQuery && searchResults.length > 0" class="mobile-search-results">
            <li v-for="result in searchResults" :key="result.path">
              <router-link :to="result.path" @mousedown.prevent="navigateToResult(result.path)" class="search-result-link">
                <div class="result-title">{{ result.title }}</div>
                <div v-if="result.highlightedContext" class="result-context" v-html="result.highlightedContext"></div>
              </router-link>
            </li>
          </ul>
          <div v-else-if="showResults && searchQuery && searchResults.length === 0" class="search-no-results mobile-version">
              无匹配结果
            </div>
      </div>
    </transition>

  </header>
</template>

<script setup>
import { ref, watch, computed, nextTick, onMounted, onUnmounted } from 'vue'; // Added onMounted, onUnmounted
import { pages } from '@/generated/content.js';
import { useRouter } from 'vue-router';

const props = defineProps({
  logoSrc: {
    type: String,
    default: ''
  },
  siteTitle: {
    type: String,
    required: true
  },
  navLinks: {
    type: Array,
    default: () => []
  },
  isDarkMode: {
    type: Boolean,
    required: true
  },
  isMobileView: {
    type: Boolean,
    default: false
  },
  showBetaTag: {
    type: Boolean,
    default: false
  }
});

const emit = defineEmits(['toggleDarkMode', 'toggle-menu']);

const emitToggleDarkMode = () => {
  emit('toggleDarkMode');
};

const emitToggleMenu = () => {
  emit('toggle-menu');
};

// 判断链接是内部链接还是外部链接
const isInternalLink = (url) => {
  // 如果链接以/开头或是相对路径则为内部链接
  return url.startsWith('/') || (!url.startsWith('http://') && !url.startsWith('https://'));
};

// --- Search State ---
const router = useRouter();
const searchQuery = ref('');
const searchResults = ref([]);
const showResults = ref(false);
const searchInputFocused = ref(false);
const isMobileSearchActive = ref(false);
const mobileSearchInput = ref(null);

// --- Mobile Nav Links Dropdown State ---
const isMobileNavLinksDropdownOpen = ref(false);

// Function to escape regex special characters
const escapeRegex = (string) => {
  return string.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
};

// Function to get highlighted context
const getHighlightedContext = (htmlContent, query) => {
  if (!htmlContent || !query) return null;
  const lowerCaseQuery = query.toLowerCase();
  const escapedQuery = escapeRegex(query);
  const highlightRegex = new RegExp(`(${escapedQuery})`, 'gi');
  const paragraphRegex = /<p>([\s\S]*?)<\/p>/gi;
  let match;
  let bestContext = null;
  while ((match = paragraphRegex.exec(htmlContent)) !== null) {
    const paragraphContent = match[1];
    const lowerCaseParagraphContent = paragraphContent.toLowerCase();
    if (lowerCaseParagraphContent.includes(lowerCaseQuery)) {
      const tempDiv = document.createElement('div');
      tempDiv.innerHTML = paragraphContent;
      const plainText = tempDiv.textContent || tempDiv.innerText || '';
      if (plainText.toLowerCase().includes(lowerCaseQuery)) {
         const highlightedText = plainText.replace(highlightRegex, '<mark class="search-highlight">$1</mark>');
         const maxLength = 150;
         bestContext = highlightedText.length > maxLength ? highlightedText.substring(0, maxLength) + '...' : highlightedText;
         break;
      }
    }
  }
  return bestContext;
};

const extractTitle = (path, html) => {
  const h1Regex = new RegExp('<h1[^>]*>(.*?)<\\/h1>', 'i');
  const h1Match = html.match(h1Regex);
  if (h1Match && h1Match[1]) {
     const tempDiv = document.createElement('div');
     tempDiv.innerHTML = h1Match[1];
     return tempDiv.textContent || tempDiv.innerText || path.split('/').pop() || path;
  }
  return path.split('/').pop() || path;
};

const performSearch = (query) => {
  if (!query) {
    searchResults.value = [];
    return;
  }
  const lowerCaseQuery = query.toLowerCase();
  const results = [];
  for (const [path, pageData] of Object.entries(pages)) {
     if (pageData.html && pageData.html.toLowerCase().includes(lowerCaseQuery)) {
       const title = extractTitle(path, pageData.html);
       const highlightedContext = getHighlightedContext(pageData.html, query);
       results.push({ path, title, highlightedContext });
     }
     // if (results.length >= 10) break; // Optional limit
  }
  searchResults.value = results;
};

watch(searchQuery, (newQuery) => {
  performSearch(newQuery);
  // Always show results container if search is active (mobile or desktop with query)
  showResults.value = !!newQuery || (props.isMobileView && isMobileSearchActive.value);
});

const clearSearch = () => {
  searchQuery.value = '';
  searchResults.value = [];
  showResults.value = false;
};

const handleFocus = () => {
  searchInputFocused.value = true;
   // Show results if there is a query OR if mobile search is active
  showResults.value = !!searchQuery.value || (props.isMobileView && isMobileSearchActive.value);
  closeMobileNavLinksDropdown(); // Close nav dropdown when focusing search
};

const handleBlur = () => {
  searchInputFocused.value = false;
  // Delay hiding results to allow click, but NOT if mobile search is active
  if (!isMobileSearchActive.value) {
      setTimeout(() => {
         // Check focus again in timeout in case user clicked back into input
         if (!searchInputFocused.value) {
             showResults.value = false;
         }
      }, 150);
  }
  // Do not close mobile nav links dropdown on blur, handle separately
};

const navigateToResult = (path) => {
  clearSearch();
  closeMobileSearch(); // Close mobile search overlay if open
  closeMobileNavLinksDropdown(); // Close nav dropdown on navigation
  router.push(path);
};

// --- Mobile Search Specific ---
const toggleMobileSearch = () => {
  isMobileSearchActive.value = !isMobileSearchActive.value;
  if (isMobileSearchActive.value) {
     showResults.value = true; // Keep results pane open when mobile search is active
     nextTick(() => { // Focus input when overlay opens
       mobileSearchInput.value?.focus();
     });
  } else {
      // Optionally clear search when closing overlay
      // clearSearch();
      showResults.value = false;
  }
  closeMobileNavLinksDropdown(); // Close nav dropdown when opening search
};

const closeMobileSearch = () => {
    isMobileSearchActive.value = false;
    showResults.value = false;
    // clearSearch(); // Optionally clear search
};

// Close mobile search if switching to desktop view
watch(() => props.isMobileView, (isMobile) => {
  if (!isMobile && isMobileSearchActive.value) {
    closeMobileSearch();
  }
});

// --- Mobile Nav Links Dropdown Methods ---
const toggleMobileNavLinksDropdown = () => {
  isMobileNavLinksDropdownOpen.value = !isMobileNavLinksDropdownOpen.value;
  if (isMobileNavLinksDropdownOpen.value) {
      closeMobileSearch(); // Close search when opening nav dropdown
  }
};

const closeMobileNavLinksDropdown = () => {
  isMobileNavLinksDropdownOpen.value = false;
};

// Close dropdown if clicking a link inside it
const closeMobileNavLinksDropdownOnClickInside = (event) => {
    // Check if the clicked element or its parent is a router-link
    let targetElement = event.target;
    while (targetElement && targetElement !== event.currentTarget) {
      if (targetElement.tagName === 'A' || targetElement.classList.contains('nav-link')) {
        closeMobileNavLinksDropdown();
        return; // Exit after closing
      }
      targetElement = targetElement.parentNode;
    }
}

// Close dropdown if switching to desktop view
watch(() => props.isMobileView, (isMobile) => {
  if (!isMobile) {
      // We already close mobile search in its own watcher
    //   closeMobileSearch();
      closeMobileNavLinksDropdown();
  }
});

// Close dropdown on click outside
const handleClickOutside = (event) => {
    const navLinksContainer = document.querySelector('.mobile-nav-links-container');
    // Check if the click happened outside the container
    if (navLinksContainer && !navLinksContainer.contains(event.target)) {
        closeMobileNavLinksDropdown();
    }

    // Optional: Close search on click outside (careful not to interfere with results clicking)
    // const searchContainer = document.querySelector('.mobile-search-overlay');
    // const searchToggleButton = document.querySelector('.search-toggle-button');
    // if (isMobileSearchActive.value && searchContainer && !searchContainer.contains(event.target) && event.target !== searchToggleButton && !searchToggleButton?.contains(event.target)) {
    //     closeMobileSearch();
    // }
};

onMounted(() => {
  document.addEventListener('click', handleClickOutside, true);
});

onUnmounted(() => {
  document.removeEventListener('click', handleClickOutside, true);
});

</script>

<style scoped>
.top-nav-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 15px; /* Adjust padding */
  background-color: rgba(248, 249, 250, 0.8); /* Semi-transparent background */
  backdrop-filter: blur(10px); /* Frosted glass effect */
  -webkit-backdrop-filter: blur(10px); /* Safari support */
  border-bottom: 1px solid rgba(233, 236, 239, 0.5); /* Semi-transparent border */
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05); /* Softer shadow */
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 60px;
  z-index: 1100;
  box-sizing: border-box;
  transition: background-color 0.3s ease, border-bottom-color 0.3s ease;
}

.left-section, .right-section {
  display: flex;
  align-items: center;
  gap: 10px; /* Adjust gap */
}

.left-section {
  flex-shrink: 0;
}

.right-section {
  flex-shrink: 0;
  position: relative; /* Needed for dropdown positioning */
}

.center-section {
  flex-grow: 1;
  display: flex;
  justify-content: center;
  padding: 0 15px; /* Adjust padding */
  min-width: 150px; /* Adjust min width */
}

/* Mobile Toggle Button Common Styles */
.mobile-menu-toggle, .search-toggle-button, .mobile-nav-links-toggle {
  background: none;
  border: none;
  padding: 5px;
  margin: 0;
  cursor: pointer;
  color: #6c757d;
  display: flex; /* Use flex to center icon */
  align-items: center;
  justify-content: center;
  transition: color 0.2s ease;
}
.mobile-menu-toggle:hover, .search-toggle-button:hover, .mobile-nav-links-toggle:hover {
  color: #333;
}
.mobile-menu-toggle svg, .search-toggle-button svg, .mobile-nav-links-toggle svg {
   width: 22px; /* Slightly larger */
   height: 22px;
}

#app.dark-mode .mobile-menu-toggle,
#app.dark-mode .search-toggle-button,
#app.dark-mode .mobile-nav-links-toggle {
  color: #6c757d;
}
#app.dark-mode .mobile-menu-toggle:hover,
#app.dark-mode .search-toggle-button:hover,
#app.dark-mode .mobile-nav-links-toggle:hover {
  color: #f8f9fa;
}

.logo-container-link {
  text-decoration: none; /* Remove underline from link */
  color: inherit; /* Inherit color from parent */
  display: flex; /* Ensure it behaves like a block for layout purposes */
  align-items: center;
}

.logo-container {
  display: flex;
  align-items: center;
}

.logo-img {
  height: 35px; /* Slightly smaller logo */
  margin-right: 8px;
}

.site-title {
  font-size: 1.3em; /* Adjust font size */
  font-weight: bold;
  color: #333;
  transition: color 0.3s ease;
  white-space: nowrap; /* Prevent title wrapping */
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 150px; /* Limit title width if needed */
}

.beta-tag {
  display: inline-block;
  margin-left: 8px;
  padding: 2px 6px;
  font-size: 0.7em;
  font-weight: bold;
  color: var(--primary-color);
  border: 1px solid var(--primary-color);
  border-radius: 4px;
  line-height: 1;
  vertical-align: middle; /* Align with title */
  background-color: transparent;
}

.nav-links {
  display: flex;
  gap: 15px;
  align-items: center;
}

.nav-link {
  text-decoration: none;
  color: var(--primary-color);
  font-size: 1em;
  padding: 5px 0;
  border-bottom: 2px solid transparent;
  transition: color 0.3s ease, border-bottom-color 0.3s ease;
  white-space: nowrap;
}

.nav-link.external-link {
  display: inline-flex;
  align-items: center;
  gap: 4px;
}

.nav-link.external-link::after {
  content: "";
  display: inline-block;
  width: 14px;
  height: 14px;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='14' height='14' viewBox='0 0 24 24' fill='none' stroke='%23ec4319' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6'%3E%3C/path%3E%3Cpolyline points='15 3 21 3 21 9'%3E%3C/polyline%3E%3Cline x1='10' y1='14' x2='21' y2='3'%3E%3C/line%3E%3C/svg%3E");
  background-size: contain;
  background-repeat: no-repeat;
}

#app.dark-mode .nav-link.external-link::after {
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='14' height='14' viewBox='0 0 24 24' fill='none' stroke='%23ffffff' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6'%3E%3C/path%3E%3Cpolyline points='15 3 21 3 21 9'%3E%3C/polyline%3E%3Cline x1='10' y1='14' x2='21' y2='3'%3E%3C/line%3E%3C/svg%3E");
}

.nav-link:hover,
.nav-link.router-link-exact-active {
  color: var(--primary-color);
  border-bottom: 2px solid var(--primary-color);
}

/* --- Search Styles --- */
.search-container {
  position: relative;
  width: 100%;
  max-width: 450px; /* Adjust max width */
}

.search-input {
  width: 100%;
  padding: 7px 12px; /* Adjust padding */
  border-radius: 15px;
  border: 1px solid #ced4da;
  font-size: 0.9em; /* Adjust font size */
  box-sizing: border-box;
  transition: all 0.3s ease;
  background-color: rgba(255, 255, 255, 0.7); /* Semi-transparent background for search input */
  backdrop-filter: blur(5px); /* Frosted glass for search input */
  -webkit-backdrop-filter: blur(5px); /* Safari support */
}

.search-input:focus {
  border-color: #80bdff;
  outline: 0;
  box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

.search-results, .search-no-results {
  position: absolute;
  top: calc(100% + 5px);
  left: 0;
  right: 0;
  background-color: rgba(255, 255, 255, 0.85); /* Semi-transparent background */
  backdrop-filter: blur(10px); /* Frosted glass effect */
  -webkit-backdrop-filter: blur(10px); /* Safari support */
  border: 1px solid rgba(206, 212, 218, 0.5); /* Semi-transparent border */
  border-radius: 4px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  list-style: none;
  padding: 5px 0;
  margin: 0;
  max-height: 350px; /* Adjust max height */
  overflow-y: auto;
  z-index: 1200;
}

.search-no-results {
  padding: 12px; /* Adjust padding */
  color: #6c757d;
  text-align: center;
  font-size: 0.85em;
}

.search-result-link {
  display: block;
  padding: 8px 12px; /* Adjust padding */
  color: #212529;
  text-decoration: none;
  transition: background-color 0.2s ease;
  cursor: pointer;
}
.search-result-link:hover {
  background-color: #f0f0f0;
}

.result-title {
  font-weight: 600;
  margin-bottom: 3px; /* Adjust margin */
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  font-size: 0.95em;
}

.result-context {
  font-size: 0.8em; /* Adjust font size */
  color: #555;
  line-height: 1.3;
  white-space: normal;
}

.search-highlight {
  background-color: #ffe082;
  padding: 0.1em 0;
  margin: 0 -0.1em;
  border-radius: 2px;
  font-weight: bold;
  color: #333;
}

/* --- Mobile Search Overlay Styles --- */
.mobile-search-overlay {
  position: absolute; /* Position relative to top-nav-bar */
  top: 0;
  left: 0;
  right: 0;
  height: 60px; /* Match nav bar height */
  background-color: #ffffff; /* Same as nav bar or slightly different */
  z-index: 1150; /* Above nav bar content, below results dropdown */
  display: flex;
  flex-direction: column; /* Stack header and results */
  padding: 0;
  border-bottom: 1px solid #e9ecef;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.mobile-search-header {
  display: flex;
  align-items: center;
  padding: 0 10px;
  height: 100%;
}

.mobile-search-input {
  flex-grow: 1;
  padding: 8px 10px;
  border-radius: 15px;
  border: 1px solid #ced4da;
  font-size: 0.95em;
  box-sizing: border-box;
  margin-right: 10px; /* Space before close button */
  height: 38px; /* Control height */
  background-color: rgba(255, 255, 255, 0.7); /* Semi-transparent background */
  backdrop-filter: blur(5px); /* Frosted glass */
  -webkit-backdrop-filter: blur(5px); /* Safari support */
}
.mobile-search-input:focus {
  border-color: #80bdff;
  outline: 0;
  box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

.mobile-search-close-button {
  background: none;
  border: none;
  font-size: 24px;
  font-weight: bold;
  color: #6c757d;
  cursor: pointer;
  padding: 0 5px;
  line-height: 1;
}
.mobile-search-close-button:hover {
  color: #333;
}

.mobile-search-results {
  position: absolute; /* Position below the overlay header */
  top: 60px; /* Start below the nav bar */
  left: 0;
  right: 0;
  /* Inherit styles from .search-results */
  background-color: rgba(255, 255, 255, 0.85); /* Semi-transparent */
  backdrop-filter: blur(10px); /* Frosted glass */
  -webkit-backdrop-filter: blur(10px); /* Safari support */
  border: 1px solid rgba(206, 212, 218, 0.5);
  border-top: none; /* Avoid double border */
  border-radius: 0 0 4px 4px; /* Round bottom corners only */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  list-style: none;
  padding: 5px 0;
  margin: 0;
  max-height: calc(100vh - 120px); /* Limit height */
  overflow-y: auto;
  z-index: 1200; /* Above overlay */
}

.search-no-results.mobile-version {
   position: absolute;
   top: 60px;
   left: 0;
   right: 0;
   background-color: rgba(255, 255, 255, 0.85); /* Semi-transparent */
   backdrop-filter: blur(10px); /* Frosted glass */
  -webkit-backdrop-filter: blur(10px); /* Safari support */
   border: 1px solid rgba(206, 212, 218, 0.5);
   border-top: none;
   border-radius: 0 0 4px 4px;
   padding: 15px;
   color: #6c757d;
   text-align: center;
   font-size: 0.9em;
   box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
   z-index: 1200;
}

/* Dark Mode Mobile Search */
#app.dark-mode .mobile-search-overlay {
  background-color: #2a2a2a;
  border-bottom-color: #3a3a3a;
}
#app.dark-mode .mobile-search-input {
  background-color: rgba(58, 58, 58, 0.7); /* Semi-transparent dark */
  backdrop-filter: blur(5px); /* Frosted glass */
  -webkit-backdrop-filter: blur(5px); /* Safari support */
  border-color: #555;
  color: #e0e0e0;
}
#app.dark-mode .mobile-search-input::placeholder {
  color: #999;
}
#app.dark-mode .mobile-search-input:focus {
  border-color: #777or);
  box-shadow: 0 0 0 0.2rem rgba(119, 119, 119, 0.25);
  background-color: rgba(68, 68, 68, 0.8); /* Slightly less transparent on focus */
}
#app.dark-mode .mobile-search-close-button {
  color: #6c757d;
}
#app.dark-mode .mobile-search-close-button:hover {
  color: #f8f9fa;
}
#app.dark-mode .mobile-search-results,
#app.dark-mode .search-no-results.mobile-version {
  background-color: rgba(47, 47, 47, 0.85); /* Semi-transparent dark */
  backdrop-filter: blur(10px); /* Frosted glass */
  -webkit-backdrop-filter: blur(10px); /* Safari support */
  border-color: rgba(68, 68, 68, 0.5);
  color: #aaa; /* For no-results text */
}

/* --- Dark Mode Toggle --- */
.dark-mode-toggle {
  /* Styles remain largely the same */
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 30px;
  height: 30px;
}
.dark-mode-toggle svg {
  width: 20px;
  height: 20px;
  color: #6c757d;
  transition: color 0.3s ease, transform 0.3s ease;
}
.dark-mode-toggle:hover svg {
  color: #333;
  transform: scale(1.1);
}
#app.dark-mode .dark-mode-toggle svg {
  color: #6c757d;
}
#app.dark-mode .dark-mode-toggle:hover svg {
  color: #f8f9fa;
}

/* --- Transitions --- */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.2s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

/* --- Dark Mode General Bar --- */
#app.dark-mode .top-nav-bar {
  background-color: rgba(42, 42, 42, 0.8); /* Semi-transparent dark background */
  backdrop-filter: blur(10px); /* Frosted glass effect for dark mode */
  -webkit-backdrop-filter: blur(10px); /* Safari support for dark mode */
  border-bottom-color: rgba(58, 58, 58, 0.5); /* Semi-transparent dark border */
}
#app.dark-mode .site-title {
  color: #e0e0e0;
}
#app.dark-mode .nav-link {
  color: #6c757d;
}
#app.dark-mode .nav-link:hover,
#app.dark-mode .nav-link.router-link-exact-active {
  color: #ffffff;
  border-bottom-color: #ffffff;
}
#app.dark-mode .search-input {
  background-color: rgba(58, 58, 58, 0.7); /* Semi-transparent dark for search input */
  backdrop-filter: blur(5px); /* Frosted glass for search input */
  -webkit-backdrop-filter: blur(5px); /* Safari support */
  border-color: #555;
  color: #e0e0e0;
}
#app.dark-mode .search-input::placeholder { color: #999; }
#app.dark-mode .search-input:focus {
  border-color: #777or);
  box-shadow: 0 0 0 0.2rem rgba(119, 119, 119, 0.25);
  background-color: rgba(68, 68, 68, 0.8); /* Slightly less transparent on focus */
}
#app.dark-mode .search-results, #app.dark-mode .search-no-results {
  background-color: rgba(47, 47, 47, 0.85); /* Semi-transparent dark */
  backdrop-filter: blur(10px); /* Frosted glass */
  -webkit-backdrop-filter: blur(10px); /* Safari support */
  border-color: rgba(68, 68, 68, 0.5);
}
#app.dark-mode .search-no-results { color: #aaa; }
#app.dark-mode .search-result-link { color: #e0e0e0; }
#app.dark-mode .search-result-link:hover { background-color: #3f3f3f; }
#app.dark-mode .result-context { color: #bbb; }
#app.dark-mode .search-highlight {
  background-color: #fdd835;
  color: #111;
}

/* --- Responsive Adjustments --- */
@media (max-width: 768px) {
  .top-nav-bar {
    padding: 0 10px; /* Reduce padding on mobile */
    background-color: rgba(248, 249, 250, 0.8); /* Semi-transparent background for mobile */
    backdrop-filter: blur(10px); /* Frosted glass effect for mobile */
    -webkit-backdrop-filter: blur(10px); /* Safari support for mobile */
  }

  /* Explicit rule for dark mode on mobile to ensure glassmorphism */
  #app.dark-mode .top-nav-bar {
    background-color: rgba(42, 42, 42, 0.8); /* Semi-transparent dark background for mobile */
    backdrop-filter: blur(10px); /* Frosted glass effect for dark mode on mobile */
    -webkit-backdrop-filter: blur(10px); /* Safari support for dark mode on mobile */
  }

  .center-section {
    display: none; /* Hide desktop search section */
  }

  .nav-links {
    display: none; /* Hide desktop nav links */
  }

  .left-section {
    /* Hamburger menu is added here by v-if */
  }

  .right-section {
    /* Search toggle and dark mode toggle are here */
    gap: 5px; /* Reduce gap */
  }

  .site-title {
     max-width: calc(100vw - 180px); /* Adjust max width dynamically */
     font-size: 1.1em; /* Smaller title on mobile */
  }
  .logo-img {
     height: 30px; /* Smaller logo */
  }
}

/* --- Mobile Nav Links Dropdown Styles --- */
.mobile-nav-links-container {
    position: relative; /* Context for absolute dropdown */
}
.mobile-nav-dropdown {
    position: absolute;
    top: calc(100% + 10px); /* Position below the toggle */
    right: 0;
    background-color: #ffffff;
    border: 1px solid #e0e0e0;
    border-radius: 4px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    padding: 8px 0;
    min-width: 150px; /* Minimum width */
    z-index: 1110; /* Above nav bar content */
    display: flex;
    flex-direction: column;
    gap: 5px;
    transform-origin: top right; /* Set origin for scale transform */
}

/* Base styles for dropdown links */
.mobile-nav-dropdown .nav-link {
    display: block;
    padding: 8px 15px;
    border-bottom: none;
    color: #333;
    text-decoration: none; /* Ensure no underline */
    transition: background-color 0.2s ease, color 0.2s ease; /* Smooth hover */
}
.mobile-nav-dropdown .nav-link:hover {
    background-color: #f0f0f0;
    color: var(--primary-color);
    border-bottom: none;
}
.mobile-nav-dropdown .nav-link.router-link-exact-active {
    font-weight: bold;
    color: var(--primary-color);
    border-bottom: none;
}

/* Dark mode for dropdown */
#app.dark-mode .mobile-nav-dropdown {
    background-color: #3a3a3a;
    border-color: #555;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
}
#app.dark-mode .mobile-nav-dropdown .nav-link {
    color: #e0e0e0;
}
#app.dark-mode .mobile-nav-dropdown .nav-link:hover {
    background-color: #4f4f4f;
    color: #ffffff;
}
#app.dark-mode .mobile-nav-dropdown .nav-link.router-link-exact-active {
    color: #ffffff;
    font-weight: bold;
}

/* --- Add Nav Links Dropdown Animation --- */
.nav-links-dropdown-anim-enter-active,
.nav-links-dropdown-anim-leave-active {
  transition: opacity 0.2s ease, transform 0.2s ease;
}

.nav-links-dropdown-anim-enter-from,
.nav-links-dropdown-anim-leave-to {
  opacity: 0;
  transform: scale(0.95) translateY(-5px); /* Scale down and move up slightly */
}

/* --- Add Mobile Search Animation --- */
.mobile-search-anim-enter-active,
.mobile-search-anim-leave-active {
  transition: opacity 0.25s ease, transform 0.25s ease;
}

.mobile-search-anim-enter-from,
.mobile-search-anim-leave-to {
  opacity: 0;
  transform: translateY(-10px); /* Start slightly above */
}

/* --- Responsive Adjustments (Existing) --- */
@media (max-width: 768px) {
  .top-nav-bar { padding: 0 10px; }
  .center-section { display: none; }
  .nav-links { display: none; } /* Hide desktop nav links */
  .right-section { gap: 5px; }
  .site-title { max-width: calc(100vw - 200px); font-size: 1.1em; }
  .logo-img { height: 30px; }
}

/* Add styles for the new link wrapper */
.logo-container-link {
  text-decoration: none; /* Remove underline from link */
  color: inherit; /* Inherit color from parent */
  display: flex; /* Ensure it behaves like a block for layout purposes */
  align-items: center;
}

</style> 