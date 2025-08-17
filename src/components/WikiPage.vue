<template>
  <div class="wiki-page-container">
    <div v-if="loading" class="loading-placeholder">正在加载内容...</div>
    <div v-else-if="error" class="error-message">{{ error }}</div>
    <template v-else>
      <!-- New Wrapper for Content and Contributors -->
      <div class="main-content-wrapper">
        <!-- Contributor Info -->
        <div v-if="contributorsList.length > 0 && !isIndexPage" class="contributor-info">
          <span class="contributor-label">本文贡献者:</span>
          <span v-for="(contributor, index) in contributorsList" :key="contributor" class="contributor-name">
            <!-- Add link if needed, for now just text -->
            {{ contributor }}
            <!-- Add comma separation -->
            <span v-if="index < contributorsList.length - 1">, </span>
          </span>
        </div>

        <!-- 文章内容区域 -->
        <div class="wiki-content" ref="contentRef" v-html="renderedMarkdown"></div>

        <!-- Changelog for Index Page -->
        <Changelog v-show="isIndexPage" :logs="changelogData" id="changelog-section" />

      </div>
      <!-- End New Wrapper -->

      <!-- TOC FAB -->
      <button v-if="tocItems.length > 0 && isMobileView && !isMobileTocVisible" class="toc-fab" @click="toggleMobileToc" aria-label="展开目录">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="8" y1="6" x2="21" y2="6"></line><line x1="8" y1="12" x2="21" y2="12"></line><line x1="8" y1="18" x2="21" y2="18"></line><line x1="3" y1="6" x2="3.01" y2="6"></line><line x1="3" y1="12" x2="3.01" y2="12"></line><line x1="3" y1="18" x2="3.01" y2="18"></line></svg>
      </button>

      <!-- TOC Overlay for Mobile -->
      <div
        v-if="isMobileView"
        class="toc-overlay"
        :class="{ 'is-visible': isMobileTocVisible }"
        @click="closeMobileToc"
      ></div>

      <!-- TOC Container -->
      <aside
        class="toc-container"
        :class="{ 'mobile-toc-visible': isMobileTocVisible && isMobileView }"
        v-if="tocItems.length > 0 && (!isMobileView || isMobileTocVisible)"
      >
        <button v-if="isMobileView" class="toc-close-button" @click="closeMobileToc" aria-label="关闭目录">&times;</button>
        <h4>目录</h4>
        <ul ref="tocListRef" style="position: relative;">
          <div class="toc-highlighter" ref="tocHighlighterRef"></div>
          <li 
            v-for="item in tocItems" 
            :key="item.id" 
            :data-toc-id="item.id"
            :class="{ 
              'active-toc-item': item.id === activeTocId, 
              [`toc-level-${item.level}`]: true 
            }"
          >
            <a :href="`#${item.id}`" @click.prevent="handleTocLinkClick(item.id)" v-html="item.text"></a>
          </li>
        </ul>
      </aside>
    </template>

    <!-- Custom Lightbox -->
    <transition name="lightbox-fade">
      <div v-if="isLightboxVisible" class="custom-lightbox-overlay" @click="handleOverlayClick"> 
        <div class="custom-lightbox-content">
          <img 
            :src="lightboxImageUrl" 
            :alt="lightboxImageAlt" 
            class="custom-lightbox-image"
            :style="{ transform: `scale(${lightboxZoom})` }"
            @click.stop
            ref="lightboxImageRef"
          >
        </div>
      </div>
    </transition>

    <!-- Custom Lightbox Controls (Fixed at bottom) -->
    <transition name="slide-up">
      <div v-if="isLightboxVisible" class="custom-lightbox-controls">
          <button @click="lightboxZoomIn" aria-label="放大" title="放大">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <circle cx="11" cy="11" r="8"></circle>
              <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
              <line x1="11" y1="8" x2="11" y2="14"></line>
              <line x1="8" y1="11" x2="14" y2="11"></line>
            </svg>
          </button>
          <button @click="lightboxZoomOut" aria-label="缩小" title="缩小">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <circle cx="11" cy="11" r="8"></circle>
              <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
              <line x1="8" y1="11" x2="14" y2="11"></line>
            </svg>
          </button>
          <button @click="lightboxSaveImage" aria-label="保存" title="保存">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11l5 5v11a2 2 0 0 1-2 2z"></path>
              <polyline points="17 21 17 13 7 13 7 21"></polyline>
              <polyline points="7 3 7 8 15 8"></polyline>
            </svg>
          </button>
          <button @click="closeLightbox" aria-label="关闭" title="关闭">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <line x1="18" y1="6" x2="6" y2="18"></line>
              <line x1="6" y1="6" x2="18" y2="18"></line>
            </svg>
          </button>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, nextTick, onUnmounted, computed } from 'vue'
import { useRoute } from 'vue-router'
// Removed Marked imports as parsing is done at build time
import { pages } from '@/generated/content.js' // Import pre-generated content
// 导入highlight.js核心库和样式
import hljs from 'highlight.js'
// 导入主题样式 - 选择 atom-one-light (亮色) 和 github-dark (暗色)
import 'highlight.js/styles/atom-one-light.css'; // 新的亮色主题
import 'highlight.js/styles/github-dark.css'; // 暗色主题
// 导入常用语言
import 'highlight.js/lib/languages/javascript'
import 'highlight.js/lib/languages/typescript'
import 'highlight.js/lib/languages/python'
import 'highlight.js/lib/languages/bash'
import 'highlight.js/lib/languages/css'
import 'highlight.js/lib/languages/xml'
import 'highlight.js/lib/languages/markdown'
// 导入新组件
import Changelog from './Changelog.vue';

const route = useRoute()
const renderedMarkdown = ref('')
const tocItems = ref([])
const contentRef = ref(null)
const loading = ref(true)
const error = ref(null)
const isMobileView = ref(window.innerWidth <= 992) // Use 992px breakpoint consistent with CSS
const isMobileTocVisible = ref(false)
const activeTocId = ref(null) // Added for active TOC item
const tocListRef = ref(null) // Ref for the TOC ul element
const tocHighlighterRef = ref(null) // Ref for the highlighter div
let observer = null // IntersectionObserver instance

// --- Contributor Info State ---
const lastAuthorName = ref(null)
const lastCommitHash = ref(null)
const lastCommitTimestamp = ref(null)
const contributorsList = ref([]) // Added for the list of contributors
const isIndexPage = ref(false);
const changelogData = ref([]);
// --- End Contributor Info State ---

// --- Custom Lightbox State ---
const isLightboxVisible = ref(false)
const lightboxImageUrl = ref('')
const lightboxImageAlt = ref('')
const lightboxZoom = ref(1)
const lightboxImageRef = ref(null) // 引用图片元素
// 用于触摸手势
const touchStartDistance = ref(0)
// 缩放速度调整
const ZOOM_SPEED = 0.1
// --- End Custom Lightbox State ---

// --- Function to find the currently active TOC ID based on scroll position ---
const findCurrentActiveTocId = () => {
  if (!contentRef.value) return null;
  
  const headings = Array.from(contentRef.value.querySelectorAll('h1[id], h2[id], h3[id], h4[id], h5[id], h6[id]'));
  const offset = 85; // Pixel offset from the top of the viewport (slightly below header)
  let currentActiveId = null;

  // Find the last heading whose top is above the offset
  for (const heading of headings) {
    const rect = heading.getBoundingClientRect();
    if (rect.top < offset) {
      currentActiveId = heading.id;
    } else {
      // Stop checking once we find a heading below the offset
      break; 
    }
  }
  
  // If no heading is above the offset (e.g., scrolled to top), use the first heading ID
  if (!currentActiveId && headings.length > 0) {
      currentActiveId = headings[0].id;
  }

  return currentActiveId;
};

const handleResize = () => {
  isMobileView.value = window.innerWidth <= 992;
  if (!isMobileView.value) {
    isMobileTocVisible.value = false; // Close mobile TOC if switching to desktop
  }
}

const toggleMobileToc = () => {
  isMobileTocVisible.value = !isMobileTocVisible.value;
  if (isMobileTocVisible.value) {
    // When opening the mobile TOC, find the current section and highlight it
    const currentId = findCurrentActiveTocId();
    if (currentId) {
        activeTocId.value = currentId;
        // Wait for the TOC container to be rendered/visible AND CSS transition potentially settled
        nextTick(() => {
          // Add a longer delay to allow CSS transition to finish
          setTimeout(() => {
             if (isMobileTocVisible.value && activeTocId.value === currentId) { // Check if still visible and ID matches
                 console.log(`Updating highlighter after TOC toggle timeout for ID: ${activeTocId.value}`); // Debug log
                 updateHighlighterPosition();
             }
          }); // Increased delay to 250ms
        });
    } else {
         // If no ID found, potentially hide highlighter
         if(tocHighlighterRef.value) tocHighlighterRef.value.style.opacity = '0';
    }
  }
}

const closeMobileToc = () => {
  isMobileTocVisible.value = false;
}

const scrollToHeader = (id) => {
  const container = document.documentElement; // Use the main document element for scrolling
  const element = document.getElementById(id);
  if (element) {
    const offset = 80; // Height of the fixed header
    const elementRect = element.getBoundingClientRect();
    // Calculate the target scroll position: current scroll + element's top relative to viewport - offset
    const targetScrollTop = container.scrollTop + elementRect.top - offset;

    console.log(`Scrolling to element #${id}: elementTop=${elementRect.top}, currentScrollTop=${container.scrollTop}, targetScrollTop=${targetScrollTop}`); // Debugging

    container.scrollTo({
      top: targetScrollTop,
      behavior: 'smooth'
    });
  } else {
    console.warn(`TOC scroll target not found: #${id}`);
  }
};

const handleTocLinkClick = (id) => { // 移除 async
  // 直接尝试滚动
  scrollToHeader(id);

  // 更新高亮位置的延迟逻辑保持不变
  setTimeout(() => {
     if(activeTocId.value === id) { // Only update if the ID hasn't changed again quickly
        console.log(`Updating highlighter after click timeout for ID: ${id}`); // Debug log
        updateHighlighterPosition();
     }
  }, 150); // Adjust delay if needed

  if (isMobileView.value) {
    closeMobileToc(); // Close mobile TOC after clicking a link
  }
}

const handleContentClick = (event) => {
  console.log('Content clicked:', event.target);
  if (event.target.tagName === 'IMG') {
    console.log('Image clicked:', event.target.src);
    event.preventDefault(); // Prevent default image behavior if needed

    // Ensure we are not clicking an image inside a link already
    if (event.target.closest('a')) { // Check if the image is inside a link
      return;
    }

    // Open the custom lightbox
    openLightbox(event.target.src, event.target.alt);
  } else if (event.target.classList.contains('code-copy-button')) {
    // 处理复制按钮点击
    handleCopyButtonClick(event);
  }
};

// --- Custom Lightbox Functions ---
const handleOverlayClick = (event) => {
  // Close lightbox when clicking anywhere except the image itself
  // (The image has @click.stop so this handler won't execute when image is clicked)
  closeLightbox();
};

// --- 处理鼠标滚轮缩放 ---
const handleWheel = (event) => {
  // 只在按住 Ctrl 键时才缩放
  if (event.ctrlKey) {
    event.preventDefault(); // 阻止默认滚动行为
    
    // deltaY 负值表示向上滚动（放大），正值表示向下滚动（缩小）
    const zoomDelta = event.deltaY > 0 ? -ZOOM_SPEED : ZOOM_SPEED;
    updateZoom(lightboxZoom.value + zoomDelta);
  }
};

// --- 处理触摸手势 ---
const handleTouchStart = (event) => {
  // 只在有两个或更多触摸点时处理
  if (event.touches.length >= 2) {
    // 计算两个触摸点之间的初始距离
    const touch1 = event.touches[0];
    const touch2 = event.touches[1];
    touchStartDistance.value = Math.hypot(
      touch2.clientX - touch1.clientX,
      touch2.clientY - touch1.clientY
    );
  }
};

const handleTouchMove = (event) => {
  // 阻止默认行为，避免页面滚动
  if (event.touches.length >= 2) {
    event.preventDefault();
    
    // 计算当前两个触摸点之间的距离
    const touch1 = event.touches[0];
    const touch2 = event.touches[1];
    const currentDistance = Math.hypot(
      touch2.clientX - touch1.clientX,
      touch2.clientY - touch1.clientY
    );
    
    // 如果有初始距离，计算缩放比例
    if (touchStartDistance.value > 0) {
      // 计算新的缩放级别
      const scale = currentDistance / touchStartDistance.value;
      const newZoom = Math.max(0.5, Math.min(3, lightboxZoom.value * scale));
      
      // 更新缩放值并重设初始距离（为下一次移动做准备）
      updateZoom(newZoom);
      touchStartDistance.value = currentDistance;
    }
  }
};

// 辅助函数：更新缩放值（限制范围在 0.5 到 3 之间）
const updateZoom = (newZoom) => {
  lightboxZoom.value = Math.max(0.5, Math.min(3, newZoom));
};

// 添加缩放相关的事件监听器
const addZoomEventListeners = () => {
  if (lightboxImageRef.value) {
    // 添加鼠标滚轮事件
    lightboxImageRef.value.addEventListener('wheel', handleWheel, { passive: false });
    
    // 添加触摸事件
    lightboxImageRef.value.addEventListener('touchstart', handleTouchStart, { passive: true });
    lightboxImageRef.value.addEventListener('touchmove', handleTouchMove, { passive: false });
  }
};

// 移除缩放相关的事件监听器
const removeZoomEventListeners = () => {
  if (lightboxImageRef.value) {
    // 移除鼠标滚轮事件
    lightboxImageRef.value.removeEventListener('wheel', handleWheel);
    
    // 移除触摸事件
    lightboxImageRef.value.removeEventListener('touchstart', handleTouchStart);
    lightboxImageRef.value.removeEventListener('touchmove', handleTouchMove);
  }
};

const openLightbox = (src, alt) => {
  lightboxImageUrl.value = src;
  lightboxImageAlt.value = alt || 'Image';
  lightboxZoom.value = 1; // Reset zoom
  isLightboxVisible.value = true;
  document.body.style.overflow = 'hidden'; // Prevent body scroll
  
  // 等待DOM更新后添加事件监听器
  nextTick(() => {
    addZoomEventListeners();
  });
};

const closeLightbox = () => {
  // 移除事件监听器
  removeZoomEventListeners();
  
  isLightboxVisible.value = false;
  lightboxImageUrl.value = '';
  lightboxImageAlt.value = '';
  document.body.style.overflow = ''; // Restore body scroll
};

const lightboxZoomIn = () => {
  lightboxZoom.value = Math.min(lightboxZoom.value + 0.2, 3); // Max zoom 3x
};

const lightboxZoomOut = () => {
  lightboxZoom.value = Math.max(lightboxZoom.value - 0.2, 0.5); // Min zoom 0.5x
};

const lightboxSaveImage = () => {
  const link = document.createElement('a');
  link.href = lightboxImageUrl.value;
  const filename = lightboxImageUrl.value.substring(lightboxImageUrl.value.lastIndexOf('/') + 1) || 'downloaded-image';
  link.download = filename;
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
};

// --- 代码复制功能 ---
const handleCopyButtonClick = (event) => {
  const button = event.target.closest('.code-copy-button');
  const pre = button.closest('pre');
  const code = pre.querySelector('code');
  
  if (code) {
    // 获取代码内容
    const text = code.textContent;
    
    // 复制到剪贴板
    navigator.clipboard.writeText(text)
      .then(() => {
        // 复制成功，更新按钮状态
        button.classList.add('copied');
        button.innerText = '已复制';
        
        // 2秒后恢复按钮状态
        setTimeout(() => {
          button.classList.remove('copied');
          button.innerText = '复制';
        }, 2000);
      })
      .catch(err => {
        console.error('复制失败:', err);
      });
  }
};

// 给代码块添加复制按钮
const addCopyButtonsToCodeBlocks = () => {
  if (!contentRef.value) return;
  
  // 查找所有代码块
  const preElements = contentRef.value.querySelectorAll('pre');
  
  preElements.forEach(pre => {
    // 检查是否已经有复制按钮，避免重复添加
    if (pre.querySelector('.code-copy-button')) return;
    
    // 创建复制按钮
    const copyButton = document.createElement('button');
    copyButton.className = 'code-copy-button';
    copyButton.textContent = '复制';
    
    // 设置按钮位置
    pre.style.position = 'relative';
    
    // 添加到代码块
    pre.appendChild(copyButton);
  });
};

// --- 代码高亮处理 ---
const applyCodeHighlight = () => {
  if (!contentRef.value) return;
  
  // 获取所有代码块
  const codeBlocks = contentRef.value.querySelectorAll('pre code');

  codeBlocks.forEach((codeBlock) => {
    // 应用高亮
    hljs.highlightElement(codeBlock);

    // 获取代码块语言
    const classNames = Array.from(codeBlock.classList);
    const languageClass = classNames.find(className => className.startsWith('language-'));
    const language = languageClass ? languageClass.replace('language-', '') : '文本';

    // 检查父元素是否已有语言标签
    const pre = codeBlock.parentElement;
    if (!pre.querySelector('.code-language')) {
      // 创建语言标签
      const languageTag = document.createElement('span');
      languageTag.className = 'code-language';
      languageTag.textContent = language;
      pre.appendChild(languageTag);
    }
    
    // 添加主题类名和标识类，用于切换亮色/暗色主题
    codeBlock.classList.add('light-theme-code'); // 标识亮色主题节点 (会被 atom-one-light.css 样式化)
    // 克隆一个用于暗色主题的节点
    const darkThemeCode = codeBlock.cloneNode(true);
    darkThemeCode.classList.remove('light-theme-code'); // 移除亮色标识
    darkThemeCode.classList.add('dark-theme-code'); // 添加暗色标识
    darkThemeCode.classList.add('hljs-github-dark'); // 保留暗色主题CSS需要的类
    
    // 将暗色主题代码块添加到pre中
    pre.appendChild(darkThemeCode);
  });
};

// --- Update Highlighter Position --- 
const updateHighlighterPosition = () => {
  if (!activeTocId.value || !tocListRef.value || !tocHighlighterRef.value) {
    if (tocHighlighterRef.value) tocHighlighterRef.value.style.opacity = '0';
    return;
  }

  const activeLi = tocListRef.value.querySelector(`li[data-toc-id="${activeTocId.value}"]`);
  const activeLink = activeLi ? activeLi.querySelector('a') : null; 

  if (activeLink && activeLi && tocListRef.value && tocHighlighterRef.value) { // Ensure refs are available
    const linkRect = activeLink.getBoundingClientRect();
    const listRect = tocListRef.value.getBoundingClientRect();
    const scrollTopOfList = tocListRef.value.scrollTop;

    // Padding adjustments from user
    const padding = 5;

    // Calculate position relative to the list container's viewport position,
    // adjusted by the list's own scroll position.
    const top = linkRect.top - listRect.top + scrollTopOfList - padding;
    const left = linkRect.left - listRect.left - (padding * 2); // Use linkRect.left relative to listRect.left

    const height = linkRect.height + (padding * 2);
    const width = linkRect.width + (padding * 4); // Keep user's horizontal padding

    // Debugging logs
    console.log(`Updating Highlighter for ${activeTocId.value}:`);
    console.log(`  LinkRect: top=${linkRect.top.toFixed(2)}, left=${linkRect.left.toFixed(2)}, width=${linkRect.width.toFixed(2)}, height=${linkRect.height.toFixed(2)}`);
    console.log(`  ListRect: top=${listRect.top.toFixed(2)}, left=${listRect.left.toFixed(2)}`);
    console.log(`  ScrollTop: ${scrollTopOfList.toFixed(2)}`);
    console.log(`  Calculated Top: ${top.toFixed(2)}`);
    console.log(`  Calculated Left: ${left.toFixed(2)}`);
    console.log(`  Calculated Height: ${height.toFixed(2)}`);
    console.log(`  Calculated Width: ${width.toFixed(2)}`);


    tocHighlighterRef.value.style.top = `${top}px`;
    tocHighlighterRef.value.style.height = `${height}px`;
    tocHighlighterRef.value.style.width = `${width}px`;
    tocHighlighterRef.value.style.left = `${left}px`;
    tocHighlighterRef.value.style.opacity = '1';
  } else {
    if (tocHighlighterRef.value) {
        tocHighlighterRef.value.style.opacity = '0';
    }
    // More detailed warning
    console.warn(`Could not find active elements/refs for highlighter update: activeId=${activeTocId.value}, activeLi=${!!activeLi}, activeLink=${!!activeLink}, listRef=${!!tocListRef.value}, highlighterRef=${!!tocHighlighterRef.value}`);
  }
};

// Watch for changes in activeTocId and update highlighter
watch(activeTocId, async (newId, oldId) => {
    // Wait for potential DOM updates if needed, especially on initial load
    await nextTick(); 
    updateHighlighterPosition();
});

// --- Intersection Observer Setup ---
const setupIntersectionObserver = () => {
  // Disconnect previous observer if exists
  if (observer) {
    observer.disconnect();
    observer = null;
  }

  if (!contentRef.value) return;

  // Select headings within the main content
  const headings = Array.from(contentRef.value.querySelectorAll('h1[id], h2[id], h3[id], h4[id], h5[id], h6[id]'));

  // Find the Changelog element if it's the index page
  let changelogElement = null;
  if (isIndexPage.value) {
      // Use nextTick to ensure the element exists after v-if renders
      nextTick(() => {
          changelogElement = document.getElementById('changelog-section');
          if (changelogElement && observer) { // Check if observer is already created
              observer.observe(changelogElement);
              console.log("Observer is now watching #changelog-section");
          } else if (!changelogElement) {
              console.warn("#changelog-section element not found for observer.");
          }
      });
  }

  // Combine headings and potentially the changelog element for observation setup
  const elementsToObserve = [...headings];
  // Note: We will observe the changelogElement inside nextTick if found

  if (elementsToObserve.length === 0 && !changelogElement) { // Check if there's anything to observe initially
      console.log("No headings or changelog section found to observe initially.");
      // Optionally hide highlighter if nothing to observe
       if(tocHighlighterRef.value) tocHighlighterRef.value.style.opacity = '0';
      return;
  }

  const observerOptions = {
    // Keep the rootMargin optimized for top headings,
    // but allow observing elements lower down.
    rootMargin: '-80px 0px -50% 0px', // Adjust bottom margin slightly if needed
    threshold: 0 // Trigger even if partially visible
  };

  observer = new IntersectionObserver((entries) => {
    let topVisibleHeadingEntry = null;
    let minTopHeading = Infinity;
    let isChangelogIntersecting = false;
    let changelogEntry = null;

    entries.forEach(entry => {
      // Check if it's the changelog entry
      if (entry.target.id === 'changelog-section') {
        changelogEntry = entry;
        isChangelogIntersecting = entry.isIntersecting;
      }
      // Check if it's a heading entry
      else if (entry.target.matches('h1, h2, h3, h4, h5, h6')) {
          if (entry.isIntersecting) {
              const rect = entry.target.getBoundingClientRect();
              // Consider headings mostly below the fixed header as potentially active
              if (rect.top >= 70 && rect.top < minTopHeading) {
                  minTopHeading = rect.top;
                  topVisibleHeadingEntry = entry;
              }
          }
      }
    });

    // --- Activation Logic ---
    // Prioritize changelog if it's intersecting and user is near the bottom,
    // or if no headings are actively visible near the top.
    const scrollNearBottom = (window.innerHeight + window.scrollY) >= (document.documentElement.scrollHeight - 150); // Check if near bottom (adjust threshold 150px)

    if (isChangelogIntersecting && (scrollNearBottom || !topVisibleHeadingEntry)) {
        // Check if the active ID is already the changelog, avoid unnecessary updates
        if (activeTocId.value !== 'changelog-section') {
             activeTocId.value = 'changelog-section';
             console.log("Observer activated: changelog-section");
        }
    } else if (topVisibleHeadingEntry) {
        // Fallback to the highest visible heading if changelog isn't active
        if (activeTocId.value !== topVisibleHeadingEntry.target.id) {
            activeTocId.value = topVisibleHeadingEntry.target.id;
            console.log(`Observer activated heading: ${activeTocId.value}`);
        }
    } else {
         // If nothing specific is active (e.g., scrolled past everything or way above),
         // try finding the last heading *above* the viewport, similar to findCurrentActiveTocId
         const headingsAbove = headings
             .map(h => ({ el: h, rect: h.getBoundingClientRect() }))
             .filter(item => item.rect.top < 80); // Find headings that have scrolled past the offset

         if (headingsAbove.length > 0) {
             headingsAbove.sort((a, b) => b.rect.top - a.rect.top); // Sort by closest to the top edge
             if (activeTocId.value !== headingsAbove[0].el.id) {
                 activeTocId.value = headingsAbove[0].el.id;
                 console.log(`Observer fallback activated heading above: ${activeTocId.value}`);
             }
         } else if (tocItems.value.length > 0 && activeTocId.value !== tocItems.value[0].id) {
             // Default to the first item if nothing else fits
             activeTocId.value = tocItems.value[0].id;
              console.log(`Observer fallback activated first item: ${activeTocId.value}`);
         }
    }


  }, observerOptions);

  // Observe all headings initially
  headings.forEach(heading => {
      if(heading && heading.id) { // Ensure heading and id exist
        observer.observe(heading);
      }
    });

  // Note: Changelog observation is handled inside the nextTick check above

  // Set initial active ID based on initial scroll position or first item
   const initialActiveId = findCurrentActiveTocId();
   if (initialActiveId && activeTocId.value !== initialActiveId) {
       activeTocId.value = initialActiveId;
   } else if (!activeTocId.value && tocItems.value.length > 0) { // Ensure an ID is set initially if possible
       activeTocId.value = tocItems.value[0].id;
   }

  // Initial position update after setup
  nextTick(updateHighlighterPosition);
};

const loadContent = async (path) => {
  // Disconnect observer before loading new content - MOVED TO START
  if (observer) {
    observer.disconnect();
    observer = null;
  }
  activeTocId.value = null; // Reset active ID

  // --- Reset states ---
  lastAuthorName.value = null;
  lastCommitHash.value = null;
  lastCommitTimestamp.value = null;
  contributorsList.value = [];
  changelogData.value = [];
  isIndexPage.value = false; // Reset index page flag *before* loading
  renderedMarkdown.value = ''; // Clear previous markdown
  tocItems.value = [];      // Clear previous TOC
  loading.value = true;     // Set loading true
  error.value = null;
  closeMobileToc();

  // Remove previous listener if contentRef exists
  if (contentRef.value) {
    contentRef.value.removeEventListener('click', handleContentClick);
  }

  await nextTick(); // Ensure DOM updates (like clearing content) happen

  try {
    // Normalize path
    const lookupPath = path.endsWith('/') && path.length > 1 ? path.slice(0, -1) : (path === '' ? '/' : path);
    let finalLookupPath = lookupPath || '/';
    finalLookupPath = decodeURIComponent(finalLookupPath);

    // Determine if it's the index page *early*
    isIndexPage.value = (finalLookupPath === '/');

    if (pages[finalLookupPath]) {
      const pageData = pages[finalLookupPath];

      renderedMarkdown.value = pageData.html;
      tocItems.value = pageData.toc || [];
      lastAuthorName.value = pageData.lastAuthorName || null;
      lastCommitHash.value = pageData.lastCommitHash || null;
      lastCommitTimestamp.value = pageData.lastCommitTimestamp || null;
      contributorsList.value = pageData.contributors || [];

      if (isIndexPage.value) { // Use the already set flag
        changelogData.value = pageData.changelog || [];

        const existingChangelogToc = tocItems.value.find(item => item.id === 'changelog-section');
        if (!existingChangelogToc && changelogData.value.length > 0) { // Only add if changelog exists
          const changelogTocItem = {
            level: 1,
            text: '最近更新',
            id: 'changelog-section',
            raw: '最近更新'
          };
          tocItems.value.push(changelogTocItem);
        }
      }
      // No 'else' needed for isIndexPage here

    } else {
      // Handle page not found
      renderedMarkdown.value = '<h1>页面未找到</h1><p>抱歉，无法在预生成的内容中找到您请求的页面。</p>';
      tocItems.value = [];
      error.value = '页面未找到';
      isIndexPage.value = false; // Ensure flag is false on error
    }
  } catch (err) {
    // Handle loading errors
    renderedMarkdown.value = '<h1>加载错误</h1><p>加载预生成内容时发生错误。</p>';
    tocItems.value = [];
    error.value = '加载内容时出错';
    isIndexPage.value = false; // Ensure flag is false on error
  } finally {
    loading.value = false;
    await nextTick(); // Ensure content is rendered before interacting

    // Add copy buttons and highlight code
    addCopyButtonsToCodeBlocks();
    applyCodeHighlight();

    // Add click listener for images/copy buttons
    if (contentRef.value) {
      contentRef.value.addEventListener('click', handleContentClick);
    } else {
      console.error('contentRef is null after loading finished. Cannot attach click listener.');
    }

    // Setup Intersection Observer *AFTER* content is loaded and rendered
    // It now relies on the 'isIndexPage' flag being correctly set.
    setupIntersectionObserver();

    // Update highlighter position *AFTER* observer setup
    await nextTick(); // Allow observer setup potentially
    updateHighlighterPosition(); // Use the initially determined activeTocId from observer setup
  }
}

// Fetch content when the component mounts
onMounted(() => {
  window.addEventListener('resize', handleResize);
  loadContent(route.path);
  // Update highlighter on resize as well
  window.addEventListener('resize', updateHighlighterPosition);
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
  window.removeEventListener('resize', updateHighlighterPosition); // Remove resize listener for highlighter
  // Disconnect the observer
  if (observer) {
    observer.disconnect();
  }
  // Clean up lightbox listener
  if (contentRef.value) {
    console.log('Removing click listener from contentRef on unmount.');
    contentRef.value.removeEventListener('click', handleContentClick);
  }
})

// Watch for route changes and fetch new content
watch(() => route.path, (newPath) => {
  loadContent(newPath)
})

// --- Computed property for contributor commit URL ---
const contributorCommitUrl = computed(() => {
  // Assuming your repo structure on GitHub is 'cooku-life/docs'
  // Replace with your actual org/repo if different
  const repoBaseUrl = 'https://github.com/cooku-life/docs/commit/'; 
  return lastCommitHash.value ? `${repoBaseUrl}${lastCommitHash.value}` : '#';
});

// --- Computed property for formatted commit date ---
const lastCommitDate = computed(() => {
  if (!lastCommitTimestamp.value) return null;
  try {
    // Multiply by 1000 because JS Date expects milliseconds
    const date = new Date(lastCommitTimestamp.value * 1000);
    // Format the date as YYYY-MM-DD HH:mm
    return date.toLocaleString('sv-SE', { // Use Swedish locale for YYYY-MM-DD format
      year: 'numeric',
      month: '2-digit',
      day: '2-digit',
      hour: '2-digit',
      minute: '2-digit',
      hour12: false // Use 24-hour format
    });
  } catch (e) {
    console.error("Error formatting commit date:", e);
    return '无效日期';
  }
});
</script>

<style scoped>
/* General styles */
.wiki-page-container {
  display: flex;
  gap: 20px; /* Gap between content and desktop TOC */
  position: relative;
  width: 100%;
  max-width: 100%;
  box-sizing: border-box;
  margin: 0 auto; /* 居中显示 */
  padding: 0;
}

/* Loading and Error states */
.loading-placeholder, .error-message {
  padding: 40px;
  text-align: center;
  font-size: 1.2em;
  color: #666;
  flex: 1; /* Take up space */
}

.error-message {
  color: #dc3545; /* Bootstrap danger color */
}

/* Contributor Info Styles */
.contributor-info {
  /* margin-top: 0px; */ /* Adjust as needed */
  margin-bottom: 5px; /* Further reduce space above main content/title */
  font-size: 0.85em; /* Slightly smaller font */
  color: #6c757d; /* Subdued text color */
  text-align: left;
}

/* Style for the "Contributors:" label */
.contributor-label {
  font-weight: 600; /* Make the label slightly bolder */
  margin-right: 8px;
}

/* Style for individual contributor names */


#app.dark-mode .contributor-info {
  color: #adb5bd; /* Lighter subdued color for dark mode */
}

/* Remove dark mode link styles if not using links */
/*
#app.dark-mode .contributor-info a {
  color: #80bdff;
}
*/

/* NEW: Style for the main content wrapper */
.main-content-wrapper {
  flex: 1; /* Take up the available space */
  min-width: 0; /* Prevent flex item from overflowing */
  display: flex; /* Optional: if you need flex inside, but default block is fine */
  flex-direction: column; /* Ensure vertical stacking inside */
  max-width: 900px; /* 设置最大宽度 */
  margin: 0 auto; /* 居中显示 */
  width: 100%; /* 占满可用宽度，但不超过max-width */
}

/* Main Content Area - Removed flex: 1 */
.wiki-content {
  /* flex: 1; */ /* Removed: Now handled by wrapper */
  min-width: 0; /* Prevent flex item from overflowing */
  padding: 0; /* Padding is now handled by parent main-content */
  line-height: 1.6;
  text-align: left;
  word-wrap: break-word;
  overflow-wrap: break-word;
  word-break: break-word;
  width: 100%; /* 占满容器宽度 */
  box-sizing: border-box; /* 确保padding不会增加总宽度 */
}

.wiki-content > * {
  text-align: left;
  max-width: 100%; /* 确保所有子元素不超过容器宽度 */
  box-sizing: border-box; /* 计算宽度时包含padding和border */
}

/* Desktop Table of Contents */
.toc-container {
  width: 200px;
  flex-shrink: 0;
  padding: 20px 0 20px 15px; /* Add top padding */
  position: sticky;
  top: 80px; /* Adjust based on TopNavBar height + desired gap */
  max-height: calc(100vh - 100px); /* Adjust max-height, consider header/footer */
  overflow-y: auto;
  border-left: 1px solid #e9ecef;
  align-self: flex-start; /* Align to the top of the flex container */
  box-sizing: border-box;
}

.toc-container h4 {
  margin-top: 0;
  margin-bottom: 10px;
  font-size: 1.1em;
  color: #6c757d;
}

.toc-container ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.toc-container li {
  margin-bottom: 6px;
}

.toc-container a {
  text-decoration: none;
  color: var(--primary-color);
  font-size: 0.9em;
  transition: color 0.2s ease;
  position: relative; /* Ensure links are positioned */
  z-index: 1; /* Ensure links are above the highlighter */
}

.toc-container a:hover {
  color: var(--primary-color);
  text-decoration: underline;
}

/* TOC Level Indentation */
.toc-level-2 { padding-left: 15px; }
.toc-level-3 { padding-left: 30px; }
.toc-level-4 { padding-left: 45px; }
.toc-level-5 { padding-left: 60px; }
.toc-level-6 { padding-left: 75px; }

/* Active TOC item styling - Now primarily handles text color */
.toc-container li.active-toc-item > a {
  color: #ffffff !important; /* White text when highlighter is behind, ensure it overrides dark mode defaults */
  /* Remove background, padding, margin adjustments from previous step */
}

/* Styles for the moving highlighter */
.toc-highlighter {
  position: absolute;
  /* Remove fixed left/right */
  background-color: var(--primary-color) !important; /* Force the background color */
  border-radius: 4px;
  z-index: 0; /* Behind the link text */
  opacity: 0; /* Initially hidden until positioned */
  transition: top 0.25s ease-in-out, height 0.25s ease-in-out, 
              width 0.25s ease-in-out, left 0.25s ease-in-out, /* Add width and left */
              opacity 0.2s ease-in-out; 
}

/* --- Mobile TOC Styles --- */

.toc-fab {
  position: fixed;
  bottom: 25px;
  right: 25px;
  width: 56px;
  height: 56px;
  background-color: var(--primary-color); /* User updated color */
  color: white;
  border: none;
  border-radius: 50%;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1050; /* Above content, below mobile TOC */
  transition: background-color 0.3s ease, transform 0.15s ease-out, box-shadow 0.15s ease-out; /* Added transform and box-shadow */
}

.toc-fab:hover {
  background-color: #d43f17; /* Darker shade for hover */
  transform: scale(1.05); /* Slightly larger on hover */
}

/* Add active state for click animation */
.toc-fab:active {
  transform: scale(0.95); /* Slightly smaller when pressed */
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3); /* Adjust shadow when pressed */
  transition-duration: 0.1s; /* Faster transition for active state */
}

.toc-fab svg {
  width: 24px;
  height: 24px;
}

.toc-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 1090; /* Below mobile TOC, above FAB */
  opacity: 0;
  visibility: hidden; /* Start hidden */
  transition: opacity 0.3s ease, visibility 0.3s ease; /* Smooth transition for both */
}

/* Apply visibility when overlay should be active */
.toc-overlay.is-visible {
    opacity: 1;
    visibility: visible;
}

/* Mobile TOC Container - Overlay Style */
/* .toc-container.mobile-toc-visible { ... } */ /* Remove general rules here if they duplicate @media ones */

.toc-close-button {
  position: absolute;
  top: 10px;
  right: 15px;
  background: none;
  border: none;
  font-size: 28px;
  font-weight: bold;
  color: #6c757d;
  cursor: pointer;
  line-height: 1;
  padding: 0;
}
.toc-close-button:hover {
  color: #343a40;
}

/* --- Dark Mode Adjustments --- */

#app.dark-mode .loading-placeholder, 
#app.dark-mode .error-message {
  color: #aaa;
}
#app.dark-mode .error-message {
  color: #f8d7da; /* Light danger color */
}
#app.dark-mode .toc-container {
  border-left-color: #3a3a3a;
  background-color: transparent; /* Desktop TOC background should match main content */
}
#app.dark-mode .toc-container h4 {
  color: #e0e0e0;
}
#app.dark-mode .toc-container a {
  color: var(--primary-color); /* Adjust link color */
}
#app.dark-mode .toc-container a:hover {
    color: #d1b3ff;
}
#app.dark-mode .toc-fab {
  background-color: var(--primary-color); /* Use theme accent */
  color: #121212;
}
#app.dark-mode .toc-fab:hover {
  background-color: #a06ee1;
}
#app.dark-mode .toc-overlay {
  background-color: rgba(0, 0, 0, 0.7);
}
#app.dark-mode .toc-container.mobile-toc-visible {
  background-color: #2a2a2a; /* Dark background for mobile overlay */
  border-left: none;
  box-shadow: -2px 0 8px rgba(0, 0, 0, 0.5);
}
#app.dark-mode .toc-close-button {
  color: #adb5bd;
}
#app.dark-mode .toc-close-button:hover {
  color: #f8f9fa;
}

/* BasicLightbox Dark Mode Compatibility */

.basicLightbox__placeholder > img {
  max-width: 90vw;
  max-height: 90vh;
}
#app.dark-mode .basicLightbox__placeholder {
   color: #e0e0e0; /* Ensure text/icons inside lightbox are visible if any */
}
/* Force dark theme on lightbox if app is in dark mode */
#app.dark-mode .basicLightbox {
  background: rgba(20, 20, 20, 0.9); /* Darker overlay for dark mode */
}

/* Lightbox Styles */
.lightbox-image-container { 
  position: relative;
  display: flex; /* Use flex to help center image if needed */
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
}

/* Controls (Appended Dynamically to placeholder) */
.lightbox-controls {
  position: absolute;
  bottom: 15px;
  left: 50%;
  transform: translateX(-50%);
  background-color: rgba(30, 30, 30, 0.7);
  padding: 8px 12px;
  border-radius: 20px;
  display: flex;
  gap: 10px;
  z-index: 1150;
}

.lightbox-btn {
  background: none;
  border: none;
  color: white;
  font-size: 20px;
  cursor: pointer;
  padding: 5px;
  line-height: 1;
  transition: color 0.2s ease;
}

.lightbox-btn:hover {
  color: #ccc;
}

/* Dark mode adjustments for controls */
#app.dark-mode .lightbox-controls {
  background-color: rgba(50, 50, 50, 0.8);
}
#app.dark-mode .lightbox-btn {
  color: #e0e0e0;
}
#app.dark-mode .lightbox-btn:hover {
  color: #ffffff;
}

/* Styles for the image itself within the container */
.lightbox-image {
  display: block; /* Ensure image behaves predictably */
  max-width: 90%; /* Adjust max-width/height as needed */
  max-height: 85%; /* Leave space for controls */
  object-fit: contain; /* Ensure aspect ratio is maintained */
  /* Zoom styles are applied via JS */
}

/* Markdown content styles (:deep) */
.wiki-content :deep(h1),
.wiki-content :deep(h2),
.wiki-content :deep(h3),
.wiki-content :deep(h4),
.wiki-content :deep(h5),
.wiki-content :deep(h6) {
  margin-top: 1.5em;
  margin-bottom: 0.5em;
  font-weight: 600;
  scroll-margin-top: 80px; /* Add top margin for scroll target (adjust for fixed header) */
  max-width: 100%; /* 确保标题不超出容器 */
}
.wiki-content :deep(h1) { font-size: 2em; border-bottom: 1px solid #eee; padding-bottom: 0.3em; }
.wiki-content :deep(h2) { font-size: 1.5em; border-bottom: 1px solid #eee; padding-bottom: 0.3em; }
.wiki-content :deep(h3) { font-size: 1.25em; }
.wiki-content :deep(p) { margin-bottom: 1em; max-width: 100%; }
.wiki-content :deep(ul), .wiki-content :deep(ol) { margin-bottom: 1em; padding-left: 2em; max-width: 100%; }
.wiki-content :deep(li) { margin-bottom: 0.5em; }
.wiki-content :deep(code:not(pre code)) { background-color: #f8f8f8; padding: 0.2em 0.4em; border-radius: 3px; font-family: 'Courier New', Courier, monospace; font-size: 0.9em; word-break: break-all; white-space: pre-wrap; }
.wiki-content :deep(pre) { 
  background-color: #f8f8f8; 
  padding: 1em; 
  border-radius: 5px; 
  overflow-x: auto; 
  margin-bottom: 1em;
  position: relative !important; /* 在这里添加 !important */
  padding-top: 2.5rem; 
  max-width: 100%; /* 确保宽度不超过容器 */
  box-sizing: border-box; /* 计算宽度时包含padding和border */
}
.wiki-content :deep(pre code) { padding: 0; background-color: transparent; border-radius: 0; font-size: 1em; white-space: pre; overflow-x: auto; }
.wiki-content :deep(blockquote) { border-left: 4px solid #eee; padding-left: 1em; margin-left: 0; color: #666; margin-bottom: 1em; max-width: 100%; }
.wiki-content :deep(table) { border-collapse: collapse; width: 100%; margin-bottom: 1em; display: block; overflow-x: auto; max-width: 100%; }
.wiki-content :deep(th), .wiki-content :deep(td) { border: 1px solid #ddd; padding: 8px; text-align: left; }
.wiki-content :deep(th) { background-color: #f2f2f2; }
.wiki-content :deep(img) { max-width: 100%; height: auto; }
.wiki-content :deep(hr) { border: none; border-top: 1px solid #eee; margin: 1em 0; max-width: 100%; }

/* Markdown 强调样式 */
.wiki-content :deep(em) { font-style: italic; color: #333; }
.wiki-content :deep(strong) { font-weight: bold; color: #333; }
.wiki-content :deep(del) { text-decoration: line-through; color: #666; }

/* 夜间模式下的 Markdown 强调样式 */
#app.dark-mode .wiki-content :deep(em) { color: #e0e0e0; }
#app.dark-mode .wiki-content :deep(strong) { color: #ffffff; }
#app.dark-mode .wiki-content :deep(del) { color: #aaaaaa; }
#app.dark-mode .wiki-content :deep(h1),
#app.dark-mode .wiki-content :deep(h2),
#app.dark-mode .wiki-content :deep(h3),
#app.dark-mode .wiki-content :deep(h4),
#app.dark-mode .wiki-content :deep(h5),
#app.dark-mode .wiki-content :deep(h6) {
  border-bottom-color: #3a3a3a;
}
#app.dark-mode .wiki-content :deep(blockquote) {
  border-left-color: #3a3a3a;
  color: #aaaaaa;
}
#app.dark-mode .wiki-content :deep(hr) {
  border-top-color: #3a3a3a;
}

/* 代码复制按钮样式 */
.wiki-content :deep(.code-copy-button) {
  position: absolute;
  top: 0.5rem;
  right: 0.5rem;
  padding: 0.25rem 0.5rem;
  font-size: 0.75rem;
  color: var(--primary-color);
  background-color: transparent;
  border: 1px solid var(--primary-color);
  border-radius: 3px;
  cursor: pointer;
  opacity: 1;
  transition: all 0.2s ease-in-out;
  z-index: 1;
}

.wiki-content :deep(pre:hover .code-copy-button) {
  opacity: 1;
}

.wiki-content :deep(.code-copy-button:hover) {
  background-color: var(--primary-color);
  color: white;
}

.wiki-content :deep(.code-copy-button.copied) {
  background-color: #28a745;
  color: white;
  border-color: #28a745;
}

.wiki-content :deep(.code-copy-button.copied:hover) {
  background-color: #218838;
}

/* 暗色模式下的复制按钮样式 */
#app.dark-mode .wiki-content :deep(.code-copy-button) {
  background-color: transparent;
  color: var(--primary-color);
  border-color: var(--primary-color);
}

#app.dark-mode .wiki-content :deep(.code-copy-button:hover) {
  background-color: var(--primary-color);
  color: #ffffff;
}

#app.dark-mode .wiki-content :deep(.code-copy-button.copied) {
  background-color: #28a745;
  color: white;
  border-color: #28a745;
}

/* Dark mode Markdown styles */
#app.dark-mode .wiki-content :deep(code:not(pre code)) { background-color: #3a3a3a; color: #e0e0e0; }

/* UPDATE Theme switching logic */
/* Light mode (default) */
.wiki-content :deep(.light-theme-code) {
  display: block;
}
.wiki-content :deep(.dark-theme-code) {
  display: none;
}

/* Dark mode */
#app.dark-mode .wiki-content :deep(.light-theme-code) {
  display: none;
}
#app.dark-mode .wiki-content :deep(.dark-theme-code) {
  display: block;
}

/* 高亮样式覆盖（使用:deep确保能影响到hljs的样式） */
.wiki-content :deep(.hljs) {
  background: transparent; /* 让背景透明，使用pre的背景色 */
  padding: 0; /* 移除默认padding */
}

#app.dark-mode .wiki-content :deep(pre) {
  background-color: #2a2a2a;
}

#app.dark-mode .wiki-content :deep(.hljs) {
  /* 暗色模式下可能需要覆盖某些高亮样式 */
  color: #e0e0e0;
}

/* 修改预定义的代码块样式，添加足够的padding以适应语言标签 */
.wiki-content :deep(pre) {
  padding-top: 2.5rem; /* 增加顶部padding给语言标签留空间 */
}

/* Dark mode Markdown styles */
#app.dark-mode .wiki-content :deep(code:not(pre code)) { background-color: #3a3a3a; color: #e0e0e0; }

/* 改进代码高亮配色方案 - 亮色模式 */
.wiki-content :deep(pre code.hljs) {
  font-family: 'Fira Code', Consolas, Monaco, 'Andale Mono', 'Ubuntu Mono', monospace;
  font-size: 0.95em;
  line-height: 1.5;
}

.wiki-content :deep(pre) {
  background-color: #f8f9fa;
  border: 1px solid #e9ecef;
}

/* 亮色模式下的代码高亮调整 */
.wiki-content :deep(.light-theme-code .hljs-keyword) { color: #0550ae; }
.wiki-content :deep(.light-theme-code .hljs-built_in) { color: #6f42c1; }
.wiki-content :deep(.light-theme-code .hljs-type) { color: #0550ae; }
.wiki-content :deep(.light-theme-code .hljs-literal) { color: #0550ae; }
.wiki-content :deep(.light-theme-code .hljs-number) { color: #0550ae; }
.wiki-content :deep(.light-theme-code .hljs-string) { color: #0a3069; }
.wiki-content :deep(.light-theme-code .hljs-comment) { color: #6a737d; }
.wiki-content :deep(.light-theme-code .hljs-function) { color: #6f42c1; }
.wiki-content :deep(.light-theme-code .hljs-title) { color: #6f42c1; }
.wiki-content :deep(.light-theme-code .hljs-attr) { color: #0550ae; }
.wiki-content :deep(.light-theme-code .hljs-tag) { color: #22863a; }
.wiki-content :deep(.light-theme-code .hljs-attr) { color: #005cc5; }

/* 暗色模式下的代码高亮调整 */
#app.dark-mode .wiki-content :deep(pre) {
  background-color: #1a1a1a;
  border: 1px solid #2c2c2c;
}

.wiki-content :deep(.dark-theme-code .hljs-keyword) { color: #79b8ff; }
.wiki-content :deep(.dark-theme-code .hljs-built_in) { color: #b392f0; }
.wiki-content :deep(.dark-theme-code .hljs-type) { color: #79b8ff; }
.wiki-content :deep(.dark-theme-code .hljs-literal) { color: #79b8ff; }
.wiki-content :deep(.dark-theme-code .hljs-number) { color: #79b8ff; }
.wiki-content :deep(.dark-theme-code .hljs-string) { color: #9ecbff; }
.wiki-content :deep(.dark-theme-code .hljs-comment) { color: #8b949e; }
.wiki-content :deep(.dark-theme-code .hljs-function) { color: #b392f0; }
.wiki-content :deep(.dark-theme-code .hljs-title) { color: #b392f0; }
.wiki-content :deep(.dark-theme-code .hljs-tag) { color: #7ee787; }
.wiki-content :deep(.dark-theme-code .hljs-attr) { color: #79b8ff; }

/* 替换为新的代码高亮配色方案 - 更高对比度 */

/* 亮色模式下的代码高亮配色 - 更深色文本 */
.wiki-content :deep(.light-theme-code) { color: #121212; } /* 基础文本颜色加深 */
.wiki-content :deep(.light-theme-code .hljs-keyword) { color: #d73a49; font-weight: bold; } /* 红色关键字 */
.wiki-content :deep(.light-theme-code .hljs-built_in) { color: #6f42c1; font-weight: bold; } /* 紫色内置函数 */
.wiki-content :deep(.light-theme-code .hljs-type) { color: #005cc5; font-weight: bold; } /* 蓝色类型 */
.wiki-content :deep(.light-theme-code .hljs-literal) { color: #005cc5; } /* 蓝色字面量 */
.wiki-content :deep(.light-theme-code .hljs-number) { color: #005cc5; } /* 蓝色数字 */
.wiki-content :deep(.light-theme-code .hljs-string) { color: #032f62; } /* 深蓝色字符串 */
.wiki-content :deep(.light-theme-code .hljs-comment) { color: #5c6370; font-style: italic; } /* 灰色注释 */
.wiki-content :deep(.light-theme-code .hljs-function) { color: #6f42c1; } /* 紫色函数 */
.wiki-content :deep(.light-theme-code .hljs-title) { color: #24292e; font-weight: bold; } /* 黑色标题 */
.wiki-content :deep(.light-theme-code .hljs-symbol) { color: #e36209; } /* 橙色符号 */
.wiki-content :deep(.light-theme-code .hljs-tag) { color: #22863a; } /* 绿色标签 */
.wiki-content :deep(.light-theme-code .hljs-name) { color: #22863a; font-weight: bold; } /* 绿色名称 */
.wiki-content :deep(.light-theme-code .hljs-attr) { color: #005cc5; } /* 蓝色属性 */
.wiki-content :deep(.light-theme-code .hljs-variable) { color: #e36209; } /* 橙色变量 */
.wiki-content :deep(.light-theme-code .hljs-params) { color: #24292e; } /* 黑色参数 */
.wiki-content :deep(.light-theme-code .hljs-quote) { color: #e36209; } /* 橙色变量 */
.wiki-content :deep(.light-theme-code .hljs-strong) { color: #24292e; }
.wiki-content :deep(.light-theme-code .hljs-emphasis) { color: #24292e; }

/* 暗色模式下的代码高亮配色 - 增强对比度 */
.wiki-content :deep(.dark-theme-code) { color: #e6e6e6; } /* 基础文本颜色提亮 */
.wiki-content :deep(.dark-theme-code .hljs-keyword) { color: #ff7b72; font-weight: bold; } /* 亮红色关键字 */
.wiki-content :deep(.dark-theme-code .hljs-built_in) { color: #d2a8ff; font-weight: bold; } /* 亮紫色内置函数 */
.wiki-content :deep(.dark-theme-code .hljs-type) { color: #79c0ff; font-weight: bold; } /* 亮蓝色类型 */
.wiki-content :deep(.dark-theme-code .hljs-literal) { color: #79c0ff; } /* 亮蓝色字面量 */
.wiki-content :deep(.dark-theme-code .hljs-number) { color: #79c0ff; } /* 亮蓝色数字 */
.wiki-content :deep(.dark-theme-code .hljs-string) { color: #a5d6ff; } /* 浅蓝色字符串 */
.wiki-content :deep(.dark-theme-code .hljs-comment) { color: #8b949e; font-style: italic; } /* 灰色注释 */
.wiki-content :deep(.dark-theme-code .hljs-function) { color: #d2a8ff; } /* 亮紫色函数 */
.wiki-content :deep(.dark-theme-code .hljs-title) { color: #e6e6e6; font-weight: bold; } /* 白色标题 */
.wiki-content :deep(.dark-theme-code .hljs-symbol) { color: #ffa657; } /* 橙色符号 */
.wiki-content :deep(.dark-theme-code .hljs-tag) { color: #7ee787; } /* 亮绿色标签 */
.wiki-content :deep(.dark-theme-code .hljs-name) { color: #7ee787; font-weight: bold; } /* 亮绿色名称 */
.wiki-content :deep(.dark-theme-code .hljs-attr) { color: #79c0ff; } /* 亮蓝色属性 */
.wiki-content :deep(.dark-theme-code .hljs-variable) { color: #ffa657; } /* 橙色变量 */
.wiki-content :deep(.dark-theme-code .hljs-params) { color: #e6e6e6; } /* 白色参数 */

/* --- Enhance more common elements in light mode --- */

/* 代码语言标签样式 - Ensure positioning */
.wiki-content :deep(.code-language) {
  position: absolute !important; /* 在这里添加 !important */
  top: 0.5rem;
  left: 0.5rem;
  padding: 0.25rem 0.5rem;
  font-size: 0.75rem;
  color: var(--primary-color);
  background-color: transparent;
  border: 1px solid var(--primary-color);
  border-radius: 3px;
  font-family: 'Courier New', Courier, monospace;
  z-index: 2 !important; /* 提高 z-index 并添加 !important */
}

/* 夜间模式表格样式 */
#app.dark-mode .wiki-content :deep(table) { border-color: #3a3a3a; }
#app.dark-mode .wiki-content :deep(th), 
#app.dark-mode .wiki-content :deep(td) { 
  border-color: #3a3a3a; 
  color: #e0e0e0;
}
#app.dark-mode .wiki-content :deep(th) { 
  background-color: #2a2a2a; 
}
#app.dark-mode .wiki-content :deep(tr) {
  background-color: #1e1e1e;
}

/* Responsive adjustments - Define base mobile TOC state here */
@media (max-width: 992px) {
  .wiki-page-container {
    flex-direction: column; /* Keep content taking full width */
    gap: 0; /* Remove gap on mobile */
  }

  .main-content-wrapper {
    max-width: 90%; /* 在移动设备上允许内容占满宽度 */
    padding: 0 15px; /* 两侧添加间距 */
  }

  /* Base styles for TOC on mobile - Hidden by default */
  .toc-container {
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    width: 80vw; 
    max-width: 300px; 
    height: 100vh;
    background-color: #ffffff;
    box-shadow: -2px 0 8px rgba(0, 0, 0, 0.15);
    z-index: 1100; 
    padding: 20px;
    padding-top: 50px; 
    overflow-y: auto; 
    box-sizing: border-box;
    border-left: none; /* Ensure no desktop border remains */
    max-height: 100vh;
    align-self: auto;
    
    /* --- Animation --- */
    transform: translateX(100%); /* Start off-screen right */
    visibility: hidden; /* Start hidden and not interactable */
    /* Apply transition to the transform property */
    transition: transform 0.3s ease-in-out;
    /* Add a small delay to visibility change AFTER transform starts/ends */
    transition-property: transform, visibility; 
    transition-duration: 0.3s, 0s; /* Duration for transform, instant for visibility */
    transition-delay: 0s, 0.3s; /* Delay visibility change until transform ends (when hiding) */
  }

  /* State when mobile TOC is visible (slid in) */
  .toc-container.mobile-toc-visible {
      transform: translateX(0); /* Slide in */
      visibility: visible; /* Make visible and interactable */
      /* Adjust transition delay for visibility when showing */
      transition-delay: 0s, 0s; 
  }

  .wiki-content {
     /* Padding already handled by parent main-content */
  }
}

/* Styles for Dark Mode Mobile TOC */
#app.dark-mode {
  @media (max-width: 992px) {
    .toc-overlay {
       background-color: rgba(0, 0, 0, 0.7); /* Darker overlay */
    }
    .toc-container {
      background-color: #2a2a2a;
      box-shadow: -2px 0 8px rgba(0, 0, 0, 0.5);
    }
    .toc-close-button {
      color: #adb5bd;
    }
    .toc-close-button:hover {
      color: #f8f9fa;
    }
  }
}

/* Custom Lightbox Styles */
.custom-lightbox-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.85);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2000; /* High z-index */
  padding-bottom: 60px; /* Space for controls */
  box-sizing: border-box;
}

.custom-lightbox-content {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 90%; /* Reduced from 100% to ensure there's clickable space around it */
  height: 90%; /* Reduced from 100% to ensure there's clickable space around it */
  margin: auto; /* Center the content in the overlay */
  overflow: hidden; /* Prevent zoomed image spilling */
  position: relative; /* Ensure it's positioned relative to the overlay */
  /* Add a subtle border for debugging - can be removed later */
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.custom-lightbox-image {
  display: block;
  max-width: 95%; /* Max width within overlay */
  max-height: 95%; /* Max height within overlay */
  object-fit: contain;
  transition: transform 0.2s ease;
  transform-origin: center center;
}

.custom-lightbox-controls {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: rgba(30, 30, 30, 0.9);
  padding: 10px 0;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 24px;
  z-index: 2010; /* Above overlay */
  height: 50px; /* Fixed height */
  box-sizing: border-box;
}

.custom-lightbox-controls button {
  background: none;
  border: none;
  color: white;
  padding: 8px;
  line-height: 0; /* 修正行高，让SVG图标垂直居中 */
  border-radius: 4px;
  cursor: pointer;
  transition: color 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.custom-lightbox-controls button:hover {
  color: #e0e0e0;
  background-color: rgba(255, 255, 255, 0.1);
}

.custom-lightbox-controls button:active {
  transform: scale(0.95);
}

/* Ensure SVG icons stay properly sized and colored */
.custom-lightbox-controls button svg {
  width: 20px;
  height: 20px;
  stroke: currentColor;
  stroke-width: 2;
  stroke-linecap: round;
  stroke-linejoin: round;
  fill: none;
}

/* Transitions for lightbox */
.lightbox-fade-enter-active,
.lightbox-fade-leave-active {
  transition: opacity 0.3s ease;
}
.lightbox-fade-enter-from,
.lightbox-fade-leave-to {
  opacity: 0;
}

.slide-up-enter-active,
.slide-up-leave-active {
  transition: transform 0.3s ease;
}
.slide-up-enter-from,
.slide-up-leave-to {
  transform: translateY(100%);
}

/* --- Enhance more common elements in light mode --- */
.wiki-content :deep(pre code.hljs-github .hljs-keyword) {
  color: #d73a49; /* Dark red/magenta for keywords */
}
.wiki-content :deep(pre code.hljs-github .hljs-literal) {
  color: #005cc5; /* Same dark blue as built-ins for literals */
}
.wiki-content :deep(pre code.hljs-github .hljs-number) {
  color: #005cc5; /* Same dark blue for numbers */
}
.wiki-content :deep(pre code.hljs-github .hljs-params) {
  color: #24292e; /* Dark grey/black for parameters */
}
.wiki-content :deep(pre code.hljs-github .hljs-function),
.wiki-content :deep(pre code.hljs-github .hljs-class) .hljs-title {
  color: #6f42c1; /* Purple for function/class names */
}
.wiki-content :deep(pre code.hljs-github .hljs-tag),
.wiki-content :deep(pre code.hljs-github .hljs-name), /* For tag names */
.wiki-content :deep(pre code.hljs-github .hljs-attr) {
  color: #22863a; /* Green for HTML/XML tags and attributes */
}
.wiki-content :deep(pre code.hljs-github .hljs-variable) {
  color: #e36209; /* Orange for variables */
}
.wiki-content :deep(pre code.hljs-github .hljs-regexp) {
  color: #032f62; /* Dark blue for regex */
}
/* --- Add styles for quote, symbol/bullet in light mode --- */
.wiki-content :deep(pre code.hljs-github .hljs-quote) {
  color: #22863a; /* Use green like tags for quotes */
}
.wiki-content :deep(pre code.hljs-github .hljs-symbol),
.wiki-content :deep(pre code.hljs-github .hljs-bullet) {
  color: #e36209; /* Use orange like variables for symbols/bullets (like ***) */
}
/* If escape characters have a specific class */
.wiki-content :deep(pre code.hljs-github .hljs-escape) {
  color: #e36209; /* Use orange for escapes as well */
}
/* --- End enhancements --- */

/* Make inline code darker in light mode */
.wiki-content :deep(code:not(pre code)) {
  background-color: #eff1f3; /* Slightly adjust background */
  color: #24292e; /* Darker text for inline code */
}

/* 代码语言标签样式 - Ensure positioning */
.wiki-content :deep(.code-language) {
  position: absolute !important; /* 在这里添加 !important */
  top: 0.5rem;
  left: 0.5rem;
  padding: 0.25rem 0.5rem;
  font-size: 0.75rem;
  color: var(--primary-color);
  background-color: transparent;
  border: 1px solid var(--primary-color);
  border-radius: 3px;
  font-family: 'Courier New', Courier, monospace;
  z-index: 2 !important; /* 提高 z-index 并添加 !important */
}

/* 夜间模式表格样式 */
#app.dark-mode .wiki-content :deep(table) { border-color: #3a3a3a; }
#app.dark-mode .wiki-content :deep(th), 
#app.dark-mode .wiki-content :deep(td) { 
  border-color: #3a3a3a; 
  color: #e0e0e0;
}
#app.dark-mode .wiki-content :deep(th) { 
  background-color: #2a2a2a; 
}
#app.dark-mode .wiki-content :deep(tr) {
  background-color: #1e1e1e;
}

/* Styles specific to Changelog in WikiPage context if needed */
.main-content-wrapper .changelog-container {
  /* Add styles here if you want to override Changelog component styles specifically within WikiPage */
  max-width: 100%; /* Ensure it doesn't overflow the wrapper */
}
</style>