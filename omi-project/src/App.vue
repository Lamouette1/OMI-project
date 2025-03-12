<template>
  <div class="app">
    <header>
      <div class="header-content">
        <div class="logo">
          <i class="fa-solid fa-microphone-lines"></i>
          <h1>Omi Selfmade</h1>
        </div>
        <div class="theme-toggle">
          <button @click="toggleTheme" class="theme-button">
            <i class="fa-solid" :class="isDarkMode ? 'fa-sun' : 'fa-moon'"></i>
          </button>
        </div>
      </div>
    </header>
    
    <main>
      <VoiceRecognition />
    </main>
    
    <footer>
      <div class="footer-content">
        <p>Voice recognition powered by Web Speech API</p>
        <p>ESP32 Integration Ready</p>
      </div>
    </footer>
  </div>
</template>

<script>
import VoiceRecognition from './components/VoiceRecognition.vue'

export default {
  components: {
    VoiceRecognition
  },
  data() {
    return {
      isDarkMode: false
    }
  },
  methods: {
    toggleTheme() {
      this.isDarkMode = !this.isDarkMode;
      document.body.classList.toggle('dark-mode', this.isDarkMode);
    }
  }
}
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  --bg-color: #f7fafc;
  --text-color: #2d3748;
  --header-bg: #4a5568;
  --header-text: white;
  --footer-bg: #2d3748;
  --footer-text: white;
}

body {
  font-family: 'Inter', 'Segoe UI', sans-serif;
  line-height: 1.6;
  color: var(--text-color);
  background: var(--bg-color);
  min-height: 100vh;
  transition: background-color 0.3s, color 0.3s;
}

body.dark-mode {
  --bg-color: #1a202c;
  --text-color: #e2e8f0;
  --header-bg: #2d3748;
  --header-text: #f7fafc;
  --footer-bg: #2d3748;
  --footer-text: #e2e8f0;
}

.app {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

header {
  background: var(--header-bg);
  color: var(--header-text);
  padding: 15px 0;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  position: sticky;
  top: 0;
  z-index: 100;
}

.header-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.logo {
  display: flex;
  align-items: center;
  gap: 12px;
}

.logo i {
  font-size: 1.5rem;
  color: #4299e1;
}

.logo h1 {
  font-size: 1.5rem;
  font-weight: 700;
  margin: 0;
}

.theme-button {
  background: none;
  border: none;
  color: var(--header-text);
  font-size: 1.2rem;
  cursor: pointer;
  padding: 8px;
  border-radius: 50%;
  transition: all 0.3s;
}

.theme-button:hover {
  background: rgba(255, 255, 255, 0.1);
  transform: rotate(15deg);
}

main {
  flex: 1;
  padding: 40px 20px;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

footer {
  background: var(--footer-bg);
  color: var(--footer-text);
  padding: 20px 0;
  margin-top: 40px;
}

.footer-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.9rem;
}

@media (max-width: 768px) {
  .footer-content {
    flex-direction: column;
    gap: 10px;
    text-align: center;
  }
  
  .header-content {
    padding: 0 15px;
  }
  
  main {
    padding: 20px 15px;
  }
}

/* Add FontAwesome */
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.2/css/all.min.css');
/* Add Inter font */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
</style>
