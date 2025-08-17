<template>
  <div id="app" :class="{ 'dark-mode': isDarkMode, 'mobile-nav-open': isMobileNavOpen && isMobileView }">
    <TopNavBar 
      :logo-src="config.logoSrc" 
      :site-title="config.siteTitle" 
      :nav-links="config.navLinks"
      :is-dark-mode="isDarkMode" 
      @toggle-dark-mode="toggleDarkMode"
      @toggle-menu="toggleMobileNav"  
      :is-mobile-view="isMobileView" 
      :show-beta-tag="config.showBetaTag"
      class="top-nav-container" 
    />
    <div v-if="isMobileNavOpen && isMobileView" class="mobile-nav-overlay" @click="closeMobileNav"></div>
    <div class="page-body"> 
      <NavigationMenu 
        class="nav-container" 
        :class="{ 'is-mobile-open': isMobileNavOpen && isMobileView }" 
        @close-menu="closeMobileNav"
      />
      <div class="content-and-footer"> 
        <main class="main-content">
          <router-view />
        </main>
        <Footer class="footer-container"/>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, computed } from 'vue'
import NavigationMenu from './components/NavigationMenu.vue'
import Footer from './components/Footer.vue'
import TopNavBar from './components/TopNavBar.vue'
import { useRoute } from 'vue-router'

const config = ref({ 
  siteTitle: 'PureWiki', 
  footerText: '© 2024 PureWiki. 保留所有权利.',
  logoSrc: '',
  navLinks: [],
  showBetaTag: false
})
const route = useRoute()
const isDarkMode = ref(false)
const isMobileNavOpen = ref(false)
const isMobileView = ref(window.innerWidth <= 768)
const userPrefersDarkMode = ref(null);

const loadConfig = async () => {
  try {
    const response = await fetch('/config.json')
    if (response.ok) {
      config.value = await response.json()
      updateTitle()
      applyTheme()
    } else {
      console.error('Failed to load config.json:', response.statusText)
    }
  } catch (error) {
    console.error('Error loading config.json:', error)
  }
}

const applyTheme = () => {
  if (config.value.theme) {
    const root = document.documentElement
    const theme = config.value.theme
    
    // 只设置主要的可配置颜色
    root.style.setProperty('--primary-color', theme.primaryColor || '#ec4309')
    root.style.setProperty('--secondary-color', theme.secondaryColor || '#d63918')
    root.style.setProperty('--accent-color', theme.accentColor || '#10b981')
  }
}

const updateTitle = () => {
  const pageTitle = route.meta.title || config.value.siteTitle || 'Wiki'
  document.title = pageTitle
}

const toggleDarkMode = () => {
  const newMode = !isDarkMode.value;
  isDarkMode.value = newMode;
  userPrefersDarkMode.value = newMode;
  localStorage.setItem('darkMode', JSON.stringify(userPrefersDarkMode.value));
}

const loadDarkModePreference = () => {
  const storedPreference = localStorage.getItem('darkMode');
  if (storedPreference !== null) {
    userPrefersDarkMode.value = JSON.parse(storedPreference);
    isDarkMode.value = userPrefersDarkMode.value;
    console.log('Loaded user preference from localStorage:', isDarkMode.value);
  } else {
    userPrefersDarkMode.value = null;
    checkSystemPreference();
    console.log('No user preference found, checked system preference.');
  }
}

const checkSystemPreference = () => {
  if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
    isDarkMode.value = true;
    console.log('System prefers dark mode.');
  } else {
    isDarkMode.value = false;
    console.log('System prefers light mode.');
  }
}

const mediaQueryListener = (e) => {
  if (userPrefersDarkMode.value === null) {
    isDarkMode.value = e.matches;
    console.log('System preference changed, updating theme:', isDarkMode.value);
  } else {
    console.log('System preference changed, but user has manual override.');
  }
}

const toggleMobileNav = () => {
  isMobileNavOpen.value = !isMobileNavOpen.value
}

const closeMobileNav = () => {
  isMobileNavOpen.value = false
}

const handleResize = () => {
  const mobile = window.innerWidth <= 768
  isMobileView.value = mobile
  if (!mobile) {
    isMobileNavOpen.value = false
  }
}

onMounted(() => {
  loadConfig()
  updateTitle()
  loadDarkModePreference()
  window.addEventListener('resize', handleResize)
  handleResize()

  if (window.matchMedia) {
    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
    mediaQuery.addEventListener('change', mediaQueryListener);
    onUnmounted(() => {
      mediaQuery.removeEventListener('change', mediaQueryListener);
    });
  }
})

watch(() => route.path, () => {
  if (config.value.siteTitle) {
    updateTitle()
  }
})

watch(config, updateTitle, { deep: true })

watch(isDarkMode, (newValue) => {
  if (newValue) {
    document.body.classList.add('dark-mode-body')
  } else {
    document.body.classList.remove('dark-mode-body')
  }
}, { immediate: true })

import { onUnmounted } from 'vue'
onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})
</script>

<style>
/* Global styles */
body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  line-height: 1.6;
  color: #333;
  background-color: #fff;
}

body.dark-mode-body {
  color: #e0e0e0;
  background-color: #121212;
}

#app {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  box-sizing: border-box;
}

.page-body {
  flex: 1;
  padding-top: 60px;
  display: flex;
  position: relative;
}

.top-nav-container {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
}

.nav-container {
  position: fixed;
  top: 60px;
  left: 0;
  height: calc(100vh - 60px);
  width: 250px;
  overflow-y: auto;
  background-color: #f8f9fa;
  border-right: 1px solid #e9ecef;
  box-sizing: border-box;
  z-index: 1000;
}

.content-and-footer {
  margin-left: 300px;
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: calc(100vh - 60px);
}

.main-content {
  flex: 1;
  padding: 20px;
  box-sizing: border-box;
  text-align: left;
}

.footer-container {
  flex-shrink: 0;
  background-color: #f1f1f1;
  text-align: center;
  padding: 10px 0;
  border-top: 1px solid #e9ecef;
  height: auto;
}

/* Dark Mode Styles */
#app.dark-mode {
  color: #e0e0e0;
  background-color: #1e1e1e;
}

#app.dark-mode body {
  color: #e0e0e0;
  background-color: #121212;
}

#app.dark-mode .top-nav-container {
  background-color: rgba(42, 42, 42, 0.8) !important;
  backdrop-filter: blur(10px) !important;
  -webkit-backdrop-filter: blur(10px) !important;
  border-bottom: 1px solid #3a3a3a;
}

/* Explicitly set title color in dark mode */
#app.dark-mode .top-nav-container .site-title {
  color: #e0e0e0;
}

#app.dark-mode .nav-container {
  background-color: #2a2a2a;
  border-right: 1px solid #3a3a3a;
}


#app.dark-mode .main-content {
  background-color: #1e1e1e;
}

#app.dark-mode .footer-container {
  background-color: #2a2a2a;
  border-top: 1px solid var(--dark-border-color);
  color: #b0b0b0;
}

/* Adjust link colors for dark mode */
#app.dark-mode a {
  color: #f9f9f9;
}

#app.dark-mode a:hover {
  color: var(--primary-color);
}

/* Adjust heading colors */
#app.dark-mode h1, 
#app.dark-mode h2, 
#app.dark-mode h3, 
#app.dark-mode h4, 
#app.dark-mode h5, 
#app.dark-mode h6 {
  color: #ffffff;
}

/* Dark mode button style */
#app.dark-mode .toggle-button {
  background-color: #4f4f4f;
  border-color: #6f6f6f;
  color: #e0e0e0;
}

#app.dark-mode .toggle-button:hover {
  background-color: #5a5a5a;
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .page-body {
    padding-top: 60px;
    display: block;
    overflow-x: hidden;
  }

  .top-nav-container {
    position: fixed;
    height: 60px;
    background-color: rgba(248, 249, 250, 0.8) !important;
    backdrop-filter: blur(10px) !important;
    -webkit-backdrop-filter: blur(10px) !important;
    border-bottom: 1px solid rgba(233, 236, 239, 0.5);
  }

  .nav-container {
    position: fixed;
    top: 60px;
    left: 0;
    max-height: 70vh;
    width: 100%;
    background-color: #f8f9fa;
    border-bottom: 1px solid var(--light-border-color);
    transform: translateY(-100%);
    transition: transform 0.3s ease-in-out;
    z-index: 1000;
    overflow-y: auto;
  }

  .nav-container.is-mobile-open {
    transform: translateY(0);
  }

  .content-and-footer {
    margin-left: 0;
    min-height: calc(100vh - 60px);
    transition: transform 0.3s ease-in-out;
  }

  .main-content {
    padding: 15px;
  }


  #app.dark-mode .nav-container {
    background-color: #2a2a2a;
    border-bottom: 1px solid #3a3a3a;
  }

  #app.dark-mode .mobile-nav-overlay {
    background-color: rgba(0, 0, 0, 0.7);
  }
}

/* Add styles for mobile nav overlay */
.mobile-nav-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 999;
  display: none;
}

#app.mobile-nav-open .mobile-nav-overlay {
  display: block;
}

/* Reset overrides from previous mobile styles that are now handled differently */
@media (min-width: 769px) {
  .page-body {
    padding-top: 60px;
    display: flex;
  }
  .top-nav-container {
    position: fixed;
  }
  .nav-container {
    position: fixed;
    top: 60px;
    left: 0;
    height: calc(100vh - 60px);
    max-height: none;
    width: 250px;
    transform: none;
    transition: none;
    border-right: 1px solid #e9ecef;
    border-bottom: none;
  }
  #app.dark-mode .nav-container {
    border-right: 1px solid #3a3a3a;
    border-bottom: none;
  }
  .content-and-footer {
    margin-left: 300px;
  }
  .mobile-nav-overlay {
    display: none !important;
  }
}
</style>
