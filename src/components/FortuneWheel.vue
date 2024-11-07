<template>
  <div class="wheel-container">
    <div class="wheel-outer">
      <canvas ref="canvas" :width="size" :height="size"></canvas>
      <button :class="['spin-button', { disabled: isSpinning }]" @click="spin">
        SPIN
      </button>
    </div>
    <div class="volume-control">
      <label for="volume">Volume :</label>
      <input
        id="volume"
        v-model.number="volume"
        type="range"
        min="0"
        max="1"
        step="0.01"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, watchEffect } from "vue";

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
const spinDuration = 5000;
let animationFrameId = null;
let startTime = null;
let targetRotation = 0;

const volume = ref(1);

const tickSound = new Audio("/sound/click.mp3");
tickSound.preload = "auto";
tickSound.volume = volume.value;

// const spinButtonImage = "/images/spin-button.png";
// const imageStyle = computed(() => {
//   const degrees = (-rotation.value * 180) / Math.PI;
//   return {
//     transform: `translate(-50%, -50%) rotate(${degrees}deg)`,
//   };
// });

const sounds = ref({});

watchEffect(() => {
  sounds.value = {};
  props.items.forEach((item) => {
    if (item.sound) {
      const audio = new Audio(item.sound);
      audio.preload = "auto";
      audio.volume = volume.value;
      sounds.value[item.label] = audio;

      audio.onerror = (e) => {
        console.error(
          `Erreur lors du chargement du son pour ${item.label}:`,
          e
        );
      };
    }
  });
});

watch(volume, (newVolume) => {
  tickSound.volume = newVolume;
  Object.values(sounds.value).forEach((audio) => {
    audio.volume = newVolume;
  });
});

const drawArrow = (ctx, centerX, centerY) => {
  const arrowSize = 30;

  ctx.save();
  ctx.translate(centerX + size / 2 - 10, centerY);

  ctx.beginPath();
  ctx.moveTo(0, -arrowSize / 2);
  ctx.lineTo(-arrowSize, 0);
  ctx.lineTo(0, arrowSize / 2);
  ctx.closePath();

  ctx.fillStyle = "#FFFFFF";
  ctx.fill();

  ctx.strokeStyle = "#000000";
  ctx.lineWidth = 2;
  ctx.stroke();

  ctx.restore();
};

const drawWheel = () => {
  const ctx = canvas.value.getContext("2d");
  const centerX = size / 2;
  const centerY = size / 2;
  const radius = (size - 60) / 2;

  ctx.clearRect(0, 0, size, size);

  ctx.fillStyle = "#1a1a1a";
  ctx.fillRect(0, 0, size, size);

  ctx.beginPath();
  ctx.arc(centerX, centerY, radius, 0, Math.PI * 2);
  ctx.fillStyle = "#2a2a2a";
  ctx.fill();
  ctx.stroke();

  if (props.items.length === 0) {
    ctx.fillStyle = "#FFFFFF";
    ctx.font = "bold 24px Arial";
    ctx.textAlign = "center";
    ctx.fillText("Ajoutez des items à la roue", centerX, centerY);
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
    ctx.textAlign = "right";
    ctx.fillStyle = "#FFFFFF";
    ctx.font = "bold 20px Arial";
    ctx.strokeStyle = "#000000";
    ctx.lineWidth = 3;
    ctx.strokeText(item.label, radius - 30, 8);
    ctx.fillText(item.label, radius - 30, 8);
    ctx.restore();
  });

  drawArrow(ctx, centerX, centerY);
};

let lastSegment = null;
let lastTickTime = 0;

const animate = (currentTime) => {
  if (!startTime) startTime = currentTime;
  const elapsed = currentTime - startTime;

  const degreesPerItem = 360 / props.items.length;

  if (elapsed < spinDuration) {
    const easing = (t) => {
      return 1 - Math.pow(1 - t, 3);
    };

    const progress = easing(elapsed / spinDuration);
    rotation.value = progress * targetRotation;

    const normalizedRotation =
      ((((rotation.value * 180) / Math.PI) % 360) + 360) % 360;
    const currentSegment =
      Math.floor(normalizedRotation / degreesPerItem) % props.items.length;

    if (currentSegment !== lastSegment) {
      const timeSinceLastTick = currentTime - lastTickTime;

      if (timeSinceLastTick > 75) {
        tickSound.currentTime = 0;
        tickSound.play();
        lastTickTime = currentTime;
      }

      lastSegment = currentSegment;
    }

    drawWheel();
    animationFrameId = requestAnimationFrame(animate);
  } else {
    isSpinning.value = false;
    rotation.value = targetRotation;
    drawWheel();

    const normalizedRotation =
      ((((rotation.value * 180) / Math.PI) % 360) + 360) % 360;
    const winningIndex =
      Math.floor(normalizedRotation / degreesPerItem) % props.items.length;

    const winningItem = props.items[winningIndex];
    if (winningItem.sound && sounds.value[winningItem.label]) {
      sounds.value[winningItem.label].currentTime = 0;
      sounds.value[winningItem.label].play();
    }
  }
};

const spin = () => {
  if (isSpinning.value || props.items.length === 0) return;

  isSpinning.value = true;
  startTime = null;
  lastSegment = null;
  lastTickTime = 0;

  const minSpins = 8;
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

onMounted(() => {
  tickSound.volume = volume.value;
  drawWheel();
});
</script>

<style scoped>
.wheel-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  background-color: #1a1a1a;
  min-height: 100vh;
  justify-content: center;
}

.wheel-outer {
  position: relative;
  width: 500px;
  height: 500px;
  margin: 0 auto;
}

canvas {
  position: absolute;
  top: 0;
  left: 0;
}

.spin-button {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 100px;
  height: 100px;
  background-color: #ffd700;
  color: #ffffff;
  font-size: 24px;
  font-weight: bold;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  box-shadow: 0 0 10px #ffd700;
  transition: transform 0.2s, box-shadow 0.2s;
}

.spin-button:hover {
  transform: translate(-50%, -50%) scale(1.05);
  box-shadow: 0 0 20px #ffd700;
}

.spin-button:active {
  transform: translate(-50%, -50%) scale(0.95);
}

.spin-button.disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.volume-control {
  margin-top: 20px;
  color: #ffffff;
}

.volume-control input[type="range"] {
  width: 200px;
  margin-left: 10px;
}
</style>
