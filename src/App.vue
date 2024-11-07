<template>
  <div class="app-container">
    <main class="main-content">
      <FortuneWheel :items="items" />

      <div class="sidebar">
        <div class="upload-button" @click="triggerFileSelect">
          <span class="icon">+</span>
          <span>Add Sound</span>
        </div>
        <div ref="filesContainer" class="files-list">
          <div v-for="(item, index) in items" :key="index" class="file-item">
            <div
              class="file-color"
              :style="{ backgroundColor: item.color }"
            ></div>
            <span class="file-name">{{ item.label }}</span>
            <!-- Bouton de suppression -->
            <button class="delete-button" @click="removeItem(index)">×</button>
          </div>
        </div>
        <input
          ref="fileInput"
          type="file"
          multiple
          accept="audio/*"
          style="display: none"
          @change="onFileChange"
        />
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref } from "vue";
import FortuneWheel from "@/components/FortuneWheel.vue";

const items = ref([]);
const fileInput = ref(null);
const filesContainer = ref(null);

const colors = [
  "#FF6B6B",
  "#4ECDC4",
  "#45B7D1",
  "#96CEB4",
  "#FFBE0B",
  "#FF006E",
  "#8338EC",
  "#3A86FF",
];

const triggerFileSelect = () => {
  fileInput.value.click();
};

const processFiles = (files) => {
  for (let i = 0; i < files.length; i++) {
    const file = files[i];
    if (file.type.startsWith("audio/")) {
      const label = file.name.replace(/\.[^/.]+$/, "");
      const reader = new FileReader();
      reader.onload = (e) => {
        const soundUrl = e.target.result;
        const randomColor = colors[Math.floor(Math.random() * colors.length)];
        items.value.push({
          label,
          probability: 1,
          sound: soundUrl,
          color: randomColor,
        });

        setTimeout(() => {
          if (filesContainer.value) {
            filesContainer.value.scrollTop = filesContainer.value.scrollHeight;
          }
        }, 100);
      };
      reader.readAsDataURL(file);
    } else {
      alert("Veuillez sélectionner uniquement des fichiers audio.");
    }
  }
};

const onFileChange = (event) => {
  const files = event.target.files;
  processFiles(files);
  event.target.value = "";
};

const removeItem = (index) => {
  items.value.splice(index, 1);
};
</script>

<style>
:root {
  --primary-color: #ffd700;
  --text-color: #ffffff;
  --background-dark: #0e0e10;
  --background-light: #18181b;
  --accent-color: #9147ff;
}

body {
  margin: 0;
  padding: 0;
  background-color: var(--background-dark);
  font-family: "Inter", sans-serif;
  height: 100vh;
  overflow: hidden;
}

.app-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  background-color: var(--background-dark);
}

.logo h1 {
  text-align: center;
  color: var(--text-color);
  font-size: 1.5rem;
  font-weight: 600;
  margin: 0;
}

.main-content {
  flex: 1;
  display: flex;
  padding: 2rem;
  overflow: hidden;
}

.FortuneWheel {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
}

.sidebar {
  width: 300px;
  background-color: var(--background-light);
  padding: 1rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
  border-left: 1px solid rgba(255, 255, 255, 0.1);
  overflow-y: auto;
}

.upload-button {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  background-color: var(--accent-color);
  color: var(--text-color);
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
  justify-content: center;
}

.upload-button:hover {
  background-color: #7b3fd6;
}

.upload-button .icon {
  font-size: 1.2rem;
  font-weight: bold;
}

.files-list {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  overflow-y: auto;
  flex: 1;
  padding-right: 0.5rem;
}

.files-list::-webkit-scrollbar {
  width: 6px;
}

.files-list::-webkit-scrollbar-track {
  background: transparent;
}

.files-list::-webkit-scrollbar-thumb {
  background-color: var(--accent-color);
  border-radius: 3px;
}

.file-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  background-color: rgba(255, 255, 255, 0.1);
  padding: 0.5rem 1rem;
  border-radius: 4px;
}

.file-color {
  width: 12px;
  height: 12px;
  border-radius: 50%;
}

.file-name {
  color: var(--text-color);
  font-size: 0.9rem;
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.delete-button {
  background: none;
  border: none;
  color: var(--text-color);
  font-size: 1rem;
  cursor: pointer;
}

.delete-button:hover {
  color: #ff4d4d;
}

@media (max-width: 768px) {
  .main-content {
    flex-direction: column;
  }

  .sidebar {
    width: 100%;
    border-left: none;
    border-top: 1px solid rgba(255, 255, 255, 0.1);
  }
}
</style>
