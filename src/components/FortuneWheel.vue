<template>
  <div class="wheel-container">
    <div class="volume-control">
      <input
        id="volume"
        v-model.number="volume"
        type="range"
        min="0"
        max="1"
        step="0.01"
        orient="vertical"
      />
      <label for="volume">Volume</label>
    </div>

    <div class="wheel-outer">
      <canvas
        ref="canvas"
        :width="size"
        :height="size"
        :class="{ disabled: isSpinning }"
        @click="spin"
      ></canvas>
    </div>

    <button class="stop-button" :disabled="!isPlaying" @click="stopSound">
      Stop
    </button>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, nextTick, defineEmits } from "vue";

const emit = defineEmits(["spin-end"]);

const props = defineProps({
  items: {
    type: Array,
    required: true,
  },
});

const canvas = ref(null);
const size = 500;
const rotation = ref(0);
const isSpinning = ref(false);
const isPlaying = ref(false);
const spinDuration = 5000;
const volume = ref(1);
let animationFrameId = null;
let startTime = null;
let targetRotation = 0;
let currentSource = null;

const audioContext = new (window.AudioContext || window.webkitAudioContext)();
const gainNode = audioContext.createGain();
gainNode.gain.value = volume.value;
gainNode.connect(audioContext.destination);

const loadAudio = async (url) => {
  const response = await fetch(url);
  const arrayBuffer = await response.arrayBuffer();
  const audioBuffer = await audioContext.decodeAudioData(arrayBuffer);
  return audioBuffer;
};

const sounds = ref({});

const stopSound = () => {
  if (currentSource) {
    currentSource.stop();
    currentSource = null;
    isPlaying.value = false;
  }
};

const drawArrow = (ctx, centerX, centerY) => {
  const arrowSize = 30;

  ctx.save();
  ctx.translate(centerX + size / 2 - arrowSize, centerY);

  ctx.beginPath();
  ctx.moveTo(0, -arrowSize / 2);
  ctx.lineTo(0, arrowSize / 2);
  ctx.lineTo(-arrowSize, 0);
  ctx.closePath();

  ctx.fillStyle = "#FFFFFF";
  ctx.fill();
  ctx.strokeStyle = "#000000";
  ctx.lineWidth = 2;
  ctx.stroke();

  ctx.restore();
};

const drawWheel = () => {
  if (!canvas.value) return;
  const ctx = canvas.value.getContext("2d");
  const centerX = size / 2;
  const centerY = size / 2;
  const radius = (size - 60) / 2;

  ctx.clearRect(0, 0, size, size);

  if (props.items.length === 0) {
    ctx.beginPath();
    ctx.arc(centerX, centerY, radius, 0, Math.PI * 2);
    ctx.fillStyle = "#2a2a2a";
    ctx.fill();
    ctx.strokeStyle = "#1a1a1a";
    ctx.lineWidth = 2;
    ctx.stroke();
    return;
  }

  const anglePerItem = (Math.PI * 2) / props.items.length;

  props.items.forEach((item, index) => {
    const startAngle = index * anglePerItem - rotation.value;
    const endAngle = startAngle + anglePerItem;

    ctx.beginPath();
    ctx.moveTo(centerX, centerY);
    ctx.arc(centerX, centerY, radius, startAngle, endAngle);
    ctx.closePath();
    ctx.fillStyle = item.color;
    ctx.fill();
    ctx.strokeStyle = "#1a1a1a";
    ctx.lineWidth = 2;
    ctx.stroke();

    ctx.save();
    ctx.translate(centerX, centerY);
    ctx.rotate(startAngle + anglePerItem / 2);
    ctx.textAlign = "left";
    ctx.fillStyle = "#FFFFFF";
    ctx.font = "bold 20px Arial";
    ctx.strokeStyle = "#000000";
    ctx.lineWidth = 3;

    let labelText =
      item.label.length > 13 ? item.label.substring(0, 13) + "..." : item.label;

    ctx.strokeText(labelText, 60, 8);
    ctx.fillText(labelText, 60, 8);
    ctx.restore();
  });

  drawArrow(ctx, centerX, centerY);
};

const getWinningIndex = (rotation) => {
  const itemCount = props.items.length;
  if (itemCount === 0) return -1;

  const normalizedRotation = rotation % (2 * Math.PI);
  const positiveRotation =
    normalizedRotation < 0
      ? normalizedRotation + 2 * Math.PI
      : normalizedRotation;
  const anglePerItem = (2 * Math.PI) / itemCount;

  return Math.floor(positiveRotation / anglePerItem) % itemCount;
};

onMounted(() => {
  gainNode.gain.value = volume.value;
  drawWheel();

  watch(
    () => props.items,
    async () => {
      sounds.value = {};
      const promises = props.items.map(async (item) => {
        if (item.sound) {
          const audioBuffer = await loadAudio(item.sound);
          sounds.value[item.label] = audioBuffer;
        }
      });
      await Promise.all(promises);
      await nextTick();
      if (canvas.value) {
        drawWheel();
      }
    },
    { deep: true, flush: "post" }
  );

  const volumeControl = document.querySelector(
    '.volume-control input[type="range"]'
  );
  if (volumeControl) {
    volumeControl.style.setProperty(
      "--volume-percentage",
      `${volume.value * 100}%`
    );
  }
});

watch(volume, (newVolume) => {
  gainNode.gain.value = newVolume;
});

const playSound = (audioBuffer) => {
  if (!audioBuffer) return;
  stopSound();

  const source = audioContext.createBufferSource();
  source.buffer = audioBuffer;
  source.connect(gainNode);
  source.start(0);
  currentSource = source;
  isPlaying.value = true;

  source.onended = () => {
    isPlaying.value = false;
    currentSource = null;
  };
};

let lastSegment = null;

const animate = (currentTime) => {
  if (!startTime) startTime = currentTime;
  const elapsed = currentTime - startTime;

  if (elapsed < spinDuration) {
    const easing = (t) => {
      return 1 - Math.pow(1 - t, 3);
    };

    const progress = easing(elapsed / spinDuration);
    rotation.value = progress * targetRotation;

    const currentSegment = getWinningIndex(rotation.value);
    if (currentSegment !== lastSegment) {
      lastSegment = currentSegment;
    }

    drawWheel();
    animationFrameId = requestAnimationFrame(animate);
  } else {
    isSpinning.value = false;
    canvas.value.classList.remove("disabled");
    rotation.value = targetRotation;
    drawWheel();

    const winningIndex = getWinningIndex(rotation.value);
    const winningItem = props.items[winningIndex];
    emit("spin-end", winningItem);
    if (winningItem?.sound && sounds.value[winningItem.label]) {
      playSound(sounds.value[winningItem.label]);
    }
  }
};

const spin = () => {
  if (isSpinning.value || props.items.length === 0) return;

  isSpinning.value = true;
  canvas.value.classList.add("disabled");

  startTime = null;
  lastSegment = null;

  const minSpins = 6;
  const maxSpins = 12;
  const randomSpins = minSpins + Math.random() * (maxSpins - minSpins);
  const baseRotation = Math.PI * 2 * randomSpins;

  const totalProbability = props.items.reduce(
    (sum, item) => sum + (item.probability || 1),
    0
  );
  let random = Math.random() * totalProbability;
  let selectedIndex = 0;

  for (let i = 0; i < props.items.length; i++) {
    random -= props.items[i].probability || 1;
    if (random <= 0) {
      selectedIndex = i;
      break;
    }
  }

  const anglePerItem = (Math.PI * 2) / props.items.length;
  const randomOffset = Math.random() * (anglePerItem * 0.8);
  targetRotation = baseRotation + selectedIndex * anglePerItem + randomOffset;

  if (animationFrameId) {
    cancelAnimationFrame(animationFrameId);
  }
  animationFrameId = requestAnimationFrame(animate);
};
</script>

<style scoped>
.wheel-container {
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  width: 100%;
  height: 100%;
  padding: 20px;
}

.wheel-outer {
  position: relative;
  width: min(600px, 80vh);
  height: min(600px, 80vh);
  margin: 0 auto;
}

canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  filter: drop-shadow(0 0 20px rgba(0, 0, 0, 0.5));
  background: transparent;
  cursor: pointer;
}

canvas.disabled {
  cursor: not-allowed;
}

.volume-control {
  position: absolute;
  left: 20px;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0;
  background: rgba(255, 255, 255, 0.1);
  padding: 15px 8px;
  border-radius: 20px;
  z-index: 20;
  height: 200px;
  width: 30px;
}

.volume-control label {
  color: #ffffff;
  font-size: 10px;
  writing-mode: vertical-lr;
  transform: rotate(180deg);
  text-transform: uppercase;
  letter-spacing: 1px;
  margin-top: auto;
  padding-bottom: 5px;
}

.volume-control input[type="range"] {
  -webkit-appearance: none;
  appearance: none;
  width: 100px;
  height: 3px;
  background: rgba(255, 255, 255, 0.1);
  outline: none;
  transform: rotate(-90deg);
  margin: 50px 0;
}

.volume-control input[type="range"]::-webkit-slider-runnable-track {
  width: 100%;
  height: 3px;
  background: linear-gradient(
    to right,
    #ffffff var(--volume-percentage, 50%),
    rgba(255, 255, 255, 0.1) var(--volume-percentage, 50%)
  );
  border-radius: 3px;
}

.volume-control input[type="range"]::-moz-range-track {
  width: 100%;
  height: 3px;
  background: linear-gradient(
    to right,
    #ffffff var(--volume-percentage, 50%),
    rgba(255, 255, 255, 0.1) var(--volume-percentage, 50%)
  );
  border-radius: 3px;
}

.volume-control input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 12px;
  height: 12px;
  background: #ffffff;
  border-radius: 50%;
  cursor: pointer;
  margin-top: -4.5px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.2);
}

.volume-control input[type="range"]::-moz-range-thumb {
  width: 12px;
  height: 12px;
  background: #ffffff;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.2);
}

.stop-button {
  position: absolute;
  right: 20px;
  top: 50%;
  transform: translateY(-50%);
  padding: 10px 20px;
  background-color: #ff4444;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  transition: background-color 0.3s ease;
}

.stop-button:hover:not(:disabled) {
  background-color: #ff0000;
}

.stop-button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
  opacity: 0.7;
}

@media (max-width: 768px) {
  .volume-control {
    left: 10px;
    padding: 15px 8px;
  }

  .volume-control input[type="range"] {
    height: 100px;
  }

  .stop-button {
    right: 10px;
  }
}
</style>
