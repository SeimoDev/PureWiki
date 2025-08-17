<template>
  <div class="changelog-container">
    <h2>最近更新</h2>
    <div v-for="logGroup in logs" :key="logGroup.date" class="log-group">
      <h3>{{ logGroup.date }}</h3>
      <ul>
        <li v-for="commit in logGroup.commits" :key="commit.hash">
          <span>{{ commit.message }}</span>
          <!-- 可选：添加指向 commit 的链接 -->
          <!-- <a :href="`https://github.com/cooku-life/docs/commit/${commit.hash}`" target="_blank" rel="noopener noreferrer"> ({{ commit.hash.substring(0, 7) }})</a> -->
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>

const props = defineProps({
  logs: {
    type: Array,
    required: true,
    default: () => []
  }
});
</script>

<style scoped>
.changelog-container {
  margin-top: 30px;
  padding-top: 20px;
  border-top: 1px solid #eee;
}

#app.dark-mode .changelog-container {
  border-top-color: #3a3a3a;
}

.changelog-container h2 {
  margin-bottom: 20px;
  font-size: 1.5em;
  border-bottom: none; /* Remove default heading border if present */
}

.log-group {
  margin-bottom: 20px;
}

.log-group h3 {
  margin-bottom: 10px;
  font-size: 1.1em;
  color: #555;
}

#app.dark-mode .log-group h3 {
  color: #bbb;
}

.log-group ul {
  list-style: disc;
  padding-left: 20px;
  margin: 0;
}

.log-group li {
  margin-bottom: 8px;
  font-size: 0.95em;
}

/* 可选：链接样式 */
.log-group a {
  color: var(--primary-color);
  text-decoration: none;
  font-size: 0.9em;
  margin-left: 5px;
}

.log-group a:hover {
  text-decoration: underline;
}

#app.dark-mode .log-group a {
  color: #ff9d80; /* 调整暗色模式下的链接颜色 */
}
</style>
