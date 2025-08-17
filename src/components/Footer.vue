<template>
  <footer class="app-footer">
    <p>{{ footerText }}</p>
  </footer>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const footerText = ref('');

onMounted(async () => {
  try {
    const response = await fetch('/config.json');
    if (response.ok) {
      const config = await response.json();
      footerText.value = config.footerText || 'Default Footer Text';
    } else {
      console.error('Failed to load config.json for footer');
      footerText.value = 'Failed to load footer text.';
    }
  } catch (error) {
    console.error('Error fetching config.json for footer:', error);
    footerText.value = 'Error loading footer text.';
  }
});
</script>

<style scoped>
.app-footer {
  padding: 15px 20px;
  text-align: center;
  background-color: #f1f1f1;
  color: #555;
  border-top: 1px solid #dddolor);
  margin-top: auto; /* Push footer to the bottom */
  font-size: 0.9em;
}

.app-footer p {
  margin: 0;
}
</style>