<template>
  <div class="app-container">
    <main class="main-content">
      <FortuneWheel :items="items" @spin-end="onSpinEnd" />

      <div class="sidebar">
        <div class="upload-button" @click="triggerFileSelect">
          <span class="icon">+</span>
          <span>Add Sound</span>
        </div>
        <div ref="filesContainer" class="files-list">
          <div
            v-for="(item, index) in items"
            :key="index"
            :class="['file-item', { selected: item === selectedItem }]"
          >
            <div
              class="file-color"
              :style="{ backgroundColor: item.color }"
            ></div>
            <span class="file-name">{{ item.label }}</span>
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

    <footer>
      <p class="disclaimer" @click="showCGUModal = true">
        Avertissement : Ce site est un outil temporaire et n'enregistre aucun
        contenu. Toute responsabilité en cas d'utilisation abusive ou illégale
        incombe à l'utilisateur. Les créateurs du site déclinent toute
        responsabilité en cas de non-respect des droits d'auteur ou des lois en
        vigueur.
      </p>
    </footer>

    <!-- Modal d'introduction -->
    <div v-if="showIntroModal" class="modal-overlay">
      <div class="modal-content">
        <h2>Bienvenue sur la Roue des Sons!</h2>
        <div class="modal-body">
          <div class="intro-step">
            <h3>1. Ajoutez vos sons</h3>
            <p>
              Cliquez sur le bouton <strong>"Add Sound"</strong> pour importer
              vos fichiers audio.
            </p>
          </div>

          <div class="intro-step">
            <h3>2. Faites tourner la roue</h3>
            <p>
              Cliquez sur la roue pour la faire tourner. Le son sélectionné sera
              joué automatiquement.
            </p>
          </div>

          <div class="intro-step">
            <h3>3. Gérez vos sons</h3>
            <p>
              Retrouvez tous vos sons dans la liste à droite. Le son sélectionné
              sera mis en évidence.
            </p>
          </div>

          <div class="intro-note">
            <p>
              Note : Pour consulter les conditions générales d'utilisation,
              cliquez sur "Avertissement" en bas de page.
            </p>
          </div>
        </div>
        <button class="start-button" @click="showIntroModal = false">
          Commencer
        </button>
      </div>
    </div>

    <!-- Modal CGU existante -->
    <div
      v-if="showCGUModal"
      class="modal-overlay"
      @click.self="showCGUModal = false"
    >
      <!-- Contenu CGU existant -->
      <div class="modal-content">
        <h2>Conditions Générales d'Utilisation</h2>
        <div class="modal-body">
          <p><strong>1. Objet</strong></p>
          <p>
            Le présent site est un outil de création sonore permettant aux
            utilisateurs d’importer des sons temporaires et de les utiliser de
            manière éphémère. Aucun fichier ou contenu n’est sauvegardé ni
            conservé après la fermeture de la page.
          </p>
          <p><strong>2. Responsabilité de l'Utilisateur</strong></p>
          <p>
            L’utilisateur est seul responsable du contenu qu’il importe et des
            actions réalisées sur le site. En utilisant ce service,
            l’utilisateur s’engage à ne pas importer ou utiliser de contenu en
            violation des droits d’auteur, droits voisins, ou autres lois en
            vigueur dans son pays.
          </p>
          <p><strong>3. Avertissement sur la Légalité des Contenus</strong></p>
          <p>
            Le site ne conserve aucune donnée et n’a pas pour objectif de
            permettre une utilisation illégale. Tout contenu doit être utilisé
            dans le respect des lois applicables et ne doit enfreindre aucun
            droit de propriété intellectuelle ou autre.
          </p>
          <p>
            L’utilisateur est tenu de vérifier qu’il dispose des droits
            nécessaires pour toute utilisation de contenu tiers sur la
            plateforme.
          </p>
          <p><strong>4. Clause de Limitation de Responsabilité</strong></p>
          <p>
            Le site et ses créateurs ne peuvent en aucun cas être tenus
            responsables de l’utilisation abusive ou illégale de l’outil par
            l’utilisateur. En utilisant ce site, l’utilisateur reconnaît qu’il
            est seul responsable de ses actions, et il exonère le site et ses
            créateurs de toute responsabilité en cas d’infraction ou
            d’utilisation non autorisée.
          </p>
        </div>
        <button class="close-button" @click="showCGUModal = false">
          Fermer
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import FortuneWheel from "@/components/FortuneWheel.vue";

const items = ref([]);
const selectedItem = ref(null);
const fileInput = ref(null);
const filesContainer = ref(null);
const showCGUModal = ref(false);
const showIntroModal = ref(true);

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

const onSpinEnd = (item) => {
  selectedItem.value = item;
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

.file-item.selected {
  background-color: var(--accent-color);
  color: #fff;
}

.file-item.selected .file-name {
  color: #fff;
}

.file-item.selected .delete-button {
  color: #fff;
}

.file-item.selected .delete-button:hover {
  color: #ff4d4d;
}

.disclaimer {
  color: white;
  padding-bottom: 5px;
  padding-left: 10px;
  padding-right: 10px;
  font-size: 10px;
  cursor: pointer;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background-color: var(--background-light);
  padding: 2rem;
  border-radius: 8px;
  max-width: 600px;
  max-height: 80vh;
  overflow-y: auto;
  color: var(--text-color);
}

.modal-content h2 {
  margin-top: 0;
}

.modal-body p {
  margin-bottom: 1rem;
}

.close-button {
  margin-top: 1rem;
  background-color: var(--accent-color);
  color: #fff;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.close-button:hover {
  background-color: #7b3fd6;
}

.intro-step {
  margin-bottom: 1.5rem;
}

.intro-step h3 {
  color: var(--accent-color);
  margin-bottom: 0.5rem;
  font-size: 1.1rem;
}

.intro-step p {
  margin: 0;
  line-height: 1.4;
}

.intro-note {
  margin-top: 2rem;
  padding-top: 1rem;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.start-button {
  margin-top: 1rem;
  background-color: var(--accent-color);
  color: #fff;
  padding: 0.75rem 2rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1.1rem;
  width: 100%;
}

.start-button:hover {
  background-color: #7b3fd6;
}
@media (max-width: 768px) {
  .modal-content {
    margin: 1rem;
    padding: 1.5rem;
  }

  .intro-step h3 {
    font-size: 1rem;
  }

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
