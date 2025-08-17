<template>
  <li :class="{ 'directory': item.type === 'directory', 'file-item': item.type === 'file' }">
    <div @click="toggleExpand" :class="{ 'nav-item-header': true, 'clickable': item.type === 'directory' }">
      <span v-if="item.type === 'directory'" class="icon">{{ isExpanded ? '📂' : '📁' }}</span>
      <router-link v-if="item.type === 'file'" :to="item.path" class="nav-link" @click="handleItemClick">{{ item.name }}</router-link>
      <span v-else>{{ item.name }}</span>
    </div>
    <ul v-if="item.type === 'directory' && isExpanded && item.children && item.children.length">
      <NavItem v-for="child in item.children" :key="child.path" :item="child" @item-clicked="$emit('item-clicked')" />
    </ul>
  </li>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  item: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['item-clicked'])

const isExpanded = ref(false)

const toggleExpand = () => {
  if (props.item.type === 'directory') {
    isExpanded.value = !isExpanded.value
  }
}

const handleItemClick = () => {
  // 只有文件类型点击时才触发收起菜单
  if (props.item.type === 'file') {
    emit('item-clicked')
  }
}

// Automatically expand if the current route is within this directory
// This logic might need refinement based on exact routing structure
// import { useRoute } from 'vue-router'
// const route = useRoute()
// if (props.item.type === 'directory' && route.path.startsWith(props.item.path)) {
//   isExpanded.value = true
// }

</script>

<style scoped>
li {
  margin-left: 15px;
  list-style-type: none; /* Remove default list bullets */
}

.nav-item-header {
  padding: 5px 0;
  cursor: default;
  display: flex;
  align-items: center;
}

.nav-item-header.clickable {
  cursor: pointer;
}

.icon {
  margin-right: 5px;
  display: inline-block;
  width: 1.2em; /* Ensure icons align */
  text-align: center;
}

.nav-link {
  text-decoration: none;
  color: #6c757d;
  transition: color 0.3s ease;
}

.nav-link:hover,
.router-link-active {
  color: var(--primary-color);
  font-weight: 600;
}

/* Indent nested lists */
ul {
  padding-left: 10px; /* Further indent nested items */
  margin: 5px 0 5px 5px; /* Adjust spacing */
  list-style: none;
}

/* Remove left margin for top-level items in the main menu */
.navigation-menu > ul > li {
  margin-left: 0;
}

/* Adjust spacing for file items directly under the root */
.navigation-menu > ul > li.file-item > .nav-item-header {
  padding-left: calc(1.2em + 5px); /* Align with folder content */
}
</style>