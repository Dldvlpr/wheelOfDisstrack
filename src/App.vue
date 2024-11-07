<template>
  <div id="app">
    <div
      class="drag-drop-area"
      @click="triggerFileSelect"
      @dragover.prevent
      @dragenter.prevent
      @dragleave.prevent
      @drop.prevent="onDrop"
    >
      Click or drag and drop
    </div>
    <input
      ref="fileInput"
      type="file"
      multiple
      accept="audio/*"
      style="display: none"
      @change="onFileChange"
    />
    <FortuneWheel :items="items" />
  </div>
</template>

<script setup>
import { ref } from "vue";
import FortuneWheel from "@/components/FortuneWheel.vue";

const items = ref([]);

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

const fileInput = ref(null);

const triggerFileSelect = () => {
  fileInput.value.click();
};

const processFiles = (files) => {
  for (let i = 0; i < files.length; i++) {
    const file = files[i];
    if (file.type.startsWith("audio/")) {
      const label = prompt("Enter name for: " + file.name);
      if (label) {
        const reader = new FileReader();
        reader.onload = (e) => {
          const soundUrl = e.target.result;

          let randomColor;
          do {
            randomColor = colors[Math.floor(Math.random() * colors.length)];
          } while (
            items.value.length > 0 &&
            randomColor === items.value[items.value.length - 1].color
          );

          items.value.push({
            label: label,
            probability: 1,
            sound: soundUrl,
            color: randomColor,
          });
        };
        reader.readAsDataURL(file);
      }
    } else {
      alert("Please select audio files only.");
    }
  }
};

const onDrop = (event) => {
  const files = event.dataTransfer.files;
  processFiles(files);
};

const onFileChange = (event) => {
  const files = event.target.files;
  processFiles(files);
  event.target.value = "";
};
</script>

<style>
body {
  margin: 0;
  padding: 0;
  background-color: #1a1a1a;
}

.drag-drop-area {
  width: 80%;
  height: 150px;
  border: 2px dashed #ccc;
  border-radius: 10px;
  margin: 20px auto;
  text-align: center;
  line-height: 150px;
  color: #ccc;
  font-size: 18px;
  cursor: pointer;
}

.drag-drop-area:hover {
  border-color: #aaa;
  color: #aaa;
}
</style>
