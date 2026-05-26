<template>
  <div class="game-world" ref="worldRef">
    <div class="farm-env">
      <div class="sun">☀️</div>
      <div class="cloud c1">☁️</div><div class="cloud c2">☁️</div><div class="cloud c3">☁️</div>
      <div class="mountain m-back"></div><div class="mountain m-front"></div>
      <div class="tree t1">🌳</div><div class="tree t2">🌲</div><div class="tree t3">🌳</div><div class="tree t4">🌲</div><div class="tree t5">🌳</div>
      <div class="grass-hill hill-back"></div><div class="grass-hill hill-middle"></div><div class="grass-hill hill-front"></div>
      <div class="bush b1">🌿</div><div class="bush b2">☘️</div><div class="bush b3">🌿</div><div class="bush b4">🌾</div><div class="bush b5">☘️</div><div class="bush b6">🌿</div>
    </div>

    <div class="game-hud">
      <div class="hud-item gold-box">
        <div class="hud-icon">💰</div>
        <div class="hud-value">{{ formatNumber(gold) }}</div>
      </div>
      <div class="hud-item pop-box">
        <div class="hud-icon">🐹</div>
        <div class="hud-value">{{ hamsters.length }}/{{ maxHamsters }}</div>
      </div>
      <button class="game-btn exit-btn" @click="$emit('back')">✖ ออก</button>
    </div>

    <transition name="slide-down">
      <div v-if="selectedHamster" class="top-dashboard">
        <div class="dash-header">
          <div class="dash-title">
            <span v-if="selectedHamster.isSad">😭</span>
            <span v-else-if="selectedHamster.isSleeping">😴</span>
            <span v-else>🐹</span> 
            <b>{{ selectedHamster.name }}</b>
            <span v-if="selectedHamster.isMature" class="mature-badge">✨ โตเต็มวัย</span>
          </div>
          <button class="close-dash-btn" @click="selectedHamsterId = null">✖</button>
        </div>

        <div class="dash-content">
          <div class="dash-stats">
            <div class="stat-row">
              <span title="ความหิว">🍗</span>
              <div class="bar-bg"><div class="bar-fill" :style="{ width: selectedHamster.hunger + '%', backgroundColor: getHungerColor(selectedHamster.hunger) }"></div></div>
            </div>
            <div class="stat-row">
              <span title="พลังงาน">⚡</span>
              <div class="bar-bg"><div class="bar-fill" :style="{ width: selectedHamster.energy + '%', backgroundColor: '#60a5fa' }"></div></div>
            </div>
            <div class="stat-row">
              <span title="ความสุข">😊</span>
              <div class="bar-bg"><div class="bar-fill" :style="{ width: selectedHamster.happiness + '%', backgroundColor: '#f472b6' }"></div></div>
            </div>
            <div class="stat-row">
              <span title="ความสะอาด">🛁</span>
              <div class="bar-bg"><div class="bar-fill" :style="{ width: selectedHamster.hygiene + '%', backgroundColor: '#2dd4bf' }"></div></div>
            </div>
            <div class="stat-row">
              <span title="การเติบโต">✨</span>
              <div class="bar-bg"><div class="bar-fill growth-fill" :style="{ width: selectedHamster.growth + '%' }"></div></div>
            </div>
          </div>

          <div class="dash-actions">
            <button @click="feedHamster(selectedHamster)" :disabled="selectedHamster.hunger >= 100 || selectedHamster.isSleeping" class="action-btn btn-feed">
              🍗 ป้อน <small>(-{{ foodPrice }}G)</small>
            </button>
            <button @click="playWithHamster(selectedHamster)" :disabled="selectedHamster.happiness >= 100 || selectedHamster.isSleeping" class="action-btn btn-play">
              🎾 เล่น <small>(ฟรี)</small>
            </button>
            <button @click="cleanHamster(selectedHamster)" :disabled="selectedHamster.hygiene >= 100 || selectedHamster.isSleeping" class="action-btn btn-clean">
              🛁 อาบน้ำ <small>(ฟรี)</small>
            </button>
            <button @click="toggleSleep(selectedHamster)" class="action-btn btn-sleep" :class="{'sleeping-btn': selectedHamster.isSleeping}">
              {{ selectedHamster.isSleeping ? '☀️ ปลุก' : '🌙 นอน' }}
            </button>
            <button v-if="selectedHamster.isMature" @click="sellHamster(selectedHamster.id)" class="action-btn btn-sell">
              💰 ขาย <small>(+{{ sellPrice }}G)</small>
            </button>
          </div>
        </div>
      </div>
    </transition>

    <div class="playground">
      <transition-group name="hamster-spawn">
        <div v-for="hamster in hamsters" :key="hamster.id" 
             class="hamster-entity" 
             :class="{ 'is-selected': selectedHamsterId === hamster.id }"
             @click="selectHamster(hamster.id)"
             :id="'hamster-' + hamster.id"
             :style="{ left: hamster.x + 'px', top: hamster.y + 'px', zIndex: selectedHamsterId === hamster.id ? 9999 : Math.floor(hamster.y) }">
             
          <div class="hamster-bubble">
            <div v-if="hamster.isEating" class="effect-icon pop-anim">❤️</div>
            <div v-if="hamster.isPlaying" class="effect-icon pop-anim">🎵</div>
            <div v-if="hamster.isCleaning" class="effect-icon bubble-anim">🫧</div>
            <div v-if="hamster.isSleeping" class="sleep-zzz">
              <span style="animation-delay: 0s">Z</span><span style="animation-delay: 0.2s">z</span><span style="animation-delay: 0.4s">z</span>
            </div>
            <div v-if="hamster.hygiene <= 30 && !hamster.isCleaning" class="stink-cloud">💨</div>
            
            <div class="img-container">
               <div v-if="selectedHamsterId === hamster.id" class="selection-arrow">▼</div>
               <img :src="hamsterPic" class="hamster-img-real" 
                    :class="{ 'is-mature': hamster.isMature, 'is-sleeping': hamster.isSleeping }" 
                    :style="{ transform: `scaleX(${hamster.direction}) ${hamster.isMature ? 'scale(1.3)' : 'scale(1)'}` }" />
               <div class="hamster-shadow"></div>
               <div v-if="hamster.isMature" class="mature-glow"></div>
            </div>
            
            <div class="hamster-label">
              <span v-if="hamster.isSad">😭</span>
              <span v-else-if="hamster.isSleeping">😴</span>
              <span v-else>🐹</span> 
              {{ hamster.name }}
            </div>
          </div>
        </div>
      </transition-group>

      <button @click="buyHamster" class="add-summon-btn" :disabled="gold < animalPrice || hamsters.length >= maxHamsters">
        <div class="summon-inner">
          <div class="plus-icon">🐹+</div>
          <div class="plus-text">
            <span v-if="hamsters.length >= maxHamsters">เต็มแล้ว!</span>
            <span v-else>ซื้อเพิ่ม<br><b>{{ animalPrice }} G</b></span>
          </div>
        </div>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, onUnmounted, nextTick } from 'vue';
import { gsap } from 'gsap';
import hamsterPic from '../assets/hamster.png'; 

defineEmits(['back']);

const gold = ref(500);
const animalPrice = 100;
const foodPrice = 10;
const sellPrice = 250;
const maxHamsters = 5; // 🌟 จำกัด 5 ตัว
const hamsters = reactive([]);
let gameLoop = null;

const worldRef = ref(null);
const selectedHamsterId = ref(null);

const selectedHamster = computed(() => {
  return hamsters.find(h => h.id === selectedHamsterId.value) || null;
});

const selectHamster = (id) => {
  if (selectedHamsterId.value === id) {
    selectedHamsterId.value = null; 
  } else {
    selectedHamsterId.value = id;
  }
};

const startGameLoop = () => {
  gameLoop = setInterval(() => {
    hamsters.forEach(h => {
      
      h.isSad = (h.hunger < 30 || h.happiness < 30 || h.hygiene < 30);

      if (h.isSleeping) {
        h.energy = Math.min(100, h.energy + 5);
        if (h.energy >= 100) toggleSleep(h);
      } else {
        h.hunger = Math.max(0, h.hunger - 1.5);
        h.energy = Math.max(0, h.energy - 1);
        h.happiness = Math.max(0, h.happiness - 1);
        h.hygiene = Math.max(0, h.hygiene - 0.5);
      }

      // 🌟 ระบบหนีออกจากบ้าน!
      if (h.hunger === 0 || h.happiness === 0) {
        h.runawayTimer = (h.runawayTimer || 0) + 1;
        // ถ้าปล่อยไว้ 5 วินาทีโดยไม่ดูแล จะหนีไป
        if (h.runawayTimer > 5 && !h.isRunningAway) {
          h.isRunningAway = true;
          alert(`แงๆ 😭 "${h.name}" ทนไม่ไหว หนีออกจากฟาร์มไปแล้ว! (ดูแลน้องดีๆ หน่อยสิ)`);
          runAway(h.id);
        }
      } else {
        h.runawayTimer = 0;
      }

      if (!h.isMature && h.hunger > 30 && h.energy > 30 && h.happiness > 30 && h.hygiene > 30) {
        h.growth = Math.min(100, h.growth + 2);
        if (h.growth === 100) { 
          h.isMature = true; 
        }
      }
    });
  }, 1000); 
};

const getRandomPosition = () => {
  const paddingX = 100;
  const paddingYBottom = 180; 
  const minX = paddingX;
  const maxX = window.innerWidth - paddingX;
  const minY = window.innerHeight * 0.55; 
  const maxY = window.innerHeight - paddingYBottom;
  
  return {
    x: Math.floor(Math.random() * (maxX - minX + 1)) + minX,
    y: Math.floor(Math.random() * (maxY - minY + 1)) + minY
  };
};

const buyHamster = () => {
  if (gold.value >= animalPrice && hamsters.length < maxHamsters) {
    // 🌟 ให้ผู้เล่นตั้งชื่อ
    let inputName = prompt("ตั้งชื่อให้น้องหนูตัวใหม่:", `น้องหนู #${hamsters.length + 1}`);
    if (inputName === null) return; // กด Cancel ยกเลิกการซื้อ
    if (inputName.trim() === "") inputName = `น้องหนู #${hamsters.length + 1}`;

    gold.value -= animalPrice;
    
    const newHamster = { 
      id: Date.now(), 
      name: inputName, 
      hunger: 100,
      energy: 100,
      happiness: 100,
      hygiene: 100,
      growth: 0, 
      isMature: false,
      isEating: false,
      isPlaying: false,
      isCleaning: false,
      isSleeping: false,
      isSad: false,
      isRunningAway: false,
      runawayTimer: 0,
      x: window.innerWidth / 2, 
      y: window.innerHeight - 150,
      direction: 1
    };
    hamsters.push(newHamster);
    selectedHamsterId.value = newHamster.id;
    
    nextTick(() => {
      runToNewSpot(newHamster);
      startBreathingAnimation('#hamster-' + newHamster.id);
    });
  }
};

const runToNewSpot = (hamster) => {
  if (hamster.isSleeping || hamster.isRunningAway) return;

  const newPos = getRandomPosition();
  if (newPos.x < hamster.x) hamster.direction = -1; 
  else hamster.direction = 1;  

  const distance = Math.hypot(newPos.x - hamster.x, newPos.y - hamster.y);
  const duration = distance / 40; 

  const walkAnim = gsap.to(`#hamster-${hamster.id} .hamster-img-real`, {
    rotationZ: hamster.direction === 1 ? [ -5, 5 ] : [ 5, -5 ], 
    duration: 0.15, repeat: -1, yoyo: true, ease: "power1.inOut"
  });

  gsap.to(hamster, {
    x: newPos.x, y: newPos.y, duration: Math.max(duration, 2), ease: "linear",
    onUpdate: () => {
      if (!hamster.isSleeping && !hamster.isRunningAway) {
        gsap.set(`#hamster-${hamster.id} .img-container`, { y: Math.abs(Math.sin(Date.now() * 0.015)) * -8 });
      }
    },
    onComplete: () => {
      walkAnim.kill(); 
      gsap.to(`#hamster-${hamster.id} .hamster-img-real`, { rotationZ: 0, duration: 0.2 }); 
      gsap.set(`#hamster-${hamster.id} .img-container`, { y: 0 }); 

      setTimeout(() => {
        const h = hamsters.find(x => x.id === hamster.id);
        if (h && !h.isSleeping && !h.isRunningAway) { runToNewSpot(h); }
      }, gsap.utils.random(2000, 5000)); 
    }
  });
};

const startBreathingAnimation = (selector) => {
  gsap.to(`${selector} .hamster-img-real`, {
    scaleY: 0.96, scaleX: 1.02, duration: 0.8, repeat: -1, yoyo: true, ease: "sine.inOut"
  });
};

// 🌟 ฟังก์ชันวิ่งหนีออกจากบ้าน
const runAway = (id) => {
  const i = hamsters.findIndex(x => x.id === id);
  if (i !== -1) {
    const hamster = hamsters[i];
    if (selectedHamsterId.value === id) selectedHamsterId.value = null;

    gsap.killTweensOf(hamster); 
    gsap.killTweensOf(`#hamster-${id} .hamster-img-real`);
    
    // หันหน้าซ้าย วิ่งจู๊ดออกจากจอ
    hamster.direction = -1;
    gsap.to(`#hamster-${id} .hamster-img-real`, { rotationZ: -15, duration: 0.1, repeat: -1, yoyo: true });
    
    gsap.to(hamster, {
      x: -200, // วิ่งหลุดขอบซ้าย
      duration: 1.5,
      ease: "power1.in"
    });

    gsap.to(`#hamster-${id}`, {
      opacity: 0,
      duration: 1.5,
      ease: "power1.in",
      onComplete: () => {
        const indexToRemove = hamsters.findIndex(x => x.id === id);
        if(indexToRemove !== -1) hamsters.splice(indexToRemove, 1);
      }
    });
  }
};

const feedHamster = (hamster) => {
  if (gold.value >= foodPrice && hamster.hunger < 100) {
    gold.value -= foodPrice;
    hamster.hunger = Math.min(100, hamster.hunger + 30);
    hamster.happiness = Math.min(100, hamster.happiness + 10);
    
    hamster.isEating = true;
    gsap.fromTo(`#hamster-${hamster.id} .img-container`, 
      { scale: 1 }, { scale: 1.1, duration: 0.2, yoyo: true, repeat: 1 }
    );
    setTimeout(() => { hamster.isEating = false; }, 1000);
  }
};

const playWithHamster = (hamster) => {
  hamster.happiness = Math.min(100, hamster.happiness + 30);
  hamster.energy = Math.max(0, hamster.energy - 10); 
  
  hamster.isPlaying = true;
  gsap.fromTo(`#hamster-${hamster.id} .img-container`, 
    { y: 0 }, { y: -25, duration: 0.2, yoyo: true, repeat: 3, ease: "power1.out" }
  );
  setTimeout(() => { hamster.isPlaying = false; }, 800);
};

const cleanHamster = (hamster) => {
  hamster.hygiene = 100;
  hamster.isCleaning = true;
  setTimeout(() => { hamster.isCleaning = false; }, 1500);
};

const toggleSleep = (hamster) => {
  hamster.isSleeping = !hamster.isSleeping;
  
  if (hamster.isSleeping) {
    gsap.killTweensOf(hamster);
    gsap.to(`#hamster-${hamster.id} .img-container`, { y: 10, duration: 0.3 });
  } else {
    gsap.to(`#hamster-${hamster.id} .img-container`, { y: 0, duration: 0.3 });
    runToNewSpot(hamster);
  }
};

const sellHamster = (id) => {
  const i = hamsters.findIndex(x => x.id === id);
  if (i !== -1) {
    if (selectedHamsterId.value === id) selectedHamsterId.value = null;

    gsap.killTweensOf(hamsters[i]); 
    gsap.to(`#hamster-${id}`, {
      scale: 0, opacity: 0, y: "-=80", rotation: 180, duration: 0.6, ease: "back.in(1.7)",
      onComplete: () => {
        hamsters.splice(i, 1);
        gold.value += sellPrice;
      }
    });
  }
};

const formatNumber = (num) => num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
const getHungerColor = (h) => h > 60 ? '#34d399' : h > 20 ? '#fbbf24' : '#f87171';

onMounted(() => { startGameLoop(); });
onUnmounted(() => { if (gameLoop) clearInterval(gameLoop); });
</script>

<style scoped>
.game-world { position: relative; width: 100vw; height: 100vh; background: linear-gradient(to bottom, #bae6fd, #e0f2fe); overflow: hidden; font-family: 'Kanit', sans-serif; touch-action: manipulation;}

/* ฉากหลัง */
.farm-env { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; z-index: 1; }
.sun { position: absolute; top: 5%; right: 10%; font-size: 80px; animation: spin-sun 20s linear infinite; filter: drop-shadow(0 0 20px rgba(253, 224, 71, 0.8)); }
@keyframes spin-sun { 100% { transform: rotate(360deg); } }
.cloud { position: absolute; font-size: 60px; opacity: 0.6; animation: drift 35s linear infinite; }
.c1 { top: 12%; left: -100px; font-size: 70px; } .c2 { top: 25%; left: -200px; animation-delay: -15s; font-size: 50px; } .c3 { top: 8%; left: -300px; animation-delay: -25s; font-size: 80px; }
@keyframes drift { to { transform: translateX(120vw); } }
.mountain { position: absolute; bottom: 35%; border-radius: 50% 50% 0 0; transition: all 0.3s; }
.m-back { width: 70vw; height: 35vh; left: -10%; background: #99f6e4; animation: float-mtn 8s ease-in-out infinite alternate; }
.m-front { width: 60vw; height: 25vh; right: -5%; background: #6ee7b7; animation: float-mtn 6s ease-in-out infinite alternate-reverse; }
@keyframes float-mtn { 0% { transform: translateY(0px); } 100% { transform: translateY(15px); } }
.tree { position: absolute; bottom: 45%; transform-origin: bottom center; animation: sway 4s ease-in-out infinite alternate; z-index: 2; filter: drop-shadow(0 10px 10px rgba(0,0,0,0.1));}
.t1 { left: 8%; font-size: 90px; animation-delay: 0s; bottom: 40%; } .t2 { left: 25%; font-size: 70px; animation-delay: 0.5s; bottom: 43%; } .t3 { left: 45%; font-size: 110px; animation-delay: 1s; bottom: 39%; } .t4 { right: 20%; font-size: 80px; animation-delay: 1.5s; bottom: 42%; } .t5 { right: 5%; font-size: 100px; animation-delay: 0.3s; bottom: 40%; }
@keyframes sway { 0% { transform: rotate(-6deg); } 100% { transform: rotate(6deg); } }
.grass-hill { position: absolute; width: 120%; left: -10%; border-radius: 50% 50% 0 0 / 20% 20% 0 0; transform-origin: bottom center; }
.hill-back { bottom: 0; height: 50%; background: #dcfce7; z-index: 2; border-top: 8px solid #bbf7d0; animation: wave-back 6s ease-in-out infinite alternate; }
.hill-middle { bottom: 0; height: 40%; background: #f0fdf4; z-index: 3; border-top: 6px solid #dcfce7; animation: wave-mid 5s ease-in-out infinite alternate-reverse; }
.hill-front { bottom: -5%; height: 35%; background: #ffffff; z-index: 4; border-top: 4px solid #f0fdf4; animation: wave-front 4s ease-in-out infinite alternate; }
@keyframes wave-back { 0% { transform: translateX(-2%) rotate(-1deg); } 100% { transform: translateX(2%) rotate(1deg); } }
@keyframes wave-mid { 0% { transform: translateX(-1%) rotate(-2deg); } 100% { transform: translateX(1%) rotate(2deg); } }
@keyframes wave-front { 0% { transform: translateX(-1.5%) rotate(-1.5deg); } 100% { transform: translateX(1.5%) rotate(1.5deg); } }
.bush { position: absolute; font-size: 40px; z-index: 5; animation: sway-bush 3s ease-in-out infinite alternate; transform-origin: bottom center; filter: drop-shadow(0 5px 5px rgba(0,0,0,0.1));}
@keyframes sway-bush { 0% { transform: skewX(-8deg) rotate(-5deg); } 100% { transform: skewX(8deg) rotate(5deg); } }
.b1 { bottom: 35%; left: 15%; font-size: 45px; animation-delay: 0.2s; } .b2 { bottom: 25%; left: 35%; font-size: 35px; animation-delay: 0.7s; } .b3 { bottom: 40%; right: 25%; font-size: 50px; animation-delay: 0.4s; } .b4 { bottom: 15%; right: 10%; font-size: 60px; animation-delay: 0.9s; } .b5 { bottom: 30%; right: 45%; font-size: 30px; animation-delay: 0.1s; } .b6 { bottom: 20%; left: 5%; font-size: 55px; animation-delay: 0.6s; }

/* HUD */
.game-hud { position: absolute; top: 15px; left: 15px; right: 15px; display: flex; gap: 10px; z-index: 1000; pointer-events: none; flex-wrap: wrap;}
.game-hud > * { pointer-events: auto; }
.hud-item { display: flex; align-items: center; background: white; border-radius: 50px; padding: 4px 15px 4px 4px; box-shadow: 0 4px 0 rgba(0,0,0,0.1); border: 2px solid white; }
.gold-box { border-color: #fde047; }
.pop-box { border-color: #a78bfa; }
.hud-icon { background: #f3f4f6; width: 35px; height: 35px; border-radius: 50%; display: flex; justify-content: center; align-items: center; font-size: 20px; margin-right: 8px; }
.hud-value { font-size: 18px; font-weight: 800; color: #374151; }
.game-btn { border: none; padding: 8px 20px; border-radius: 50px; cursor: pointer; font-family: 'Kanit', sans-serif; font-size: 15px; font-weight: 800; box-shadow: 0 4px 0 rgba(0,0,0,0.15); color: white; }
.game-btn:active { transform: translateY(4px); box-shadow: 0 0px 0 rgba(0,0,0,0.15); }
.exit-btn { background: #ef4444; margin-left: auto; }

/* 🌟 แผงควบคุมด้านบน (Responsive) */
.top-dashboard {
  position: absolute;
  top: 75px; 
  left: 50%;
  transform: translateX(-50%);
  width: 95%;
  max-width: 600px;
  background: rgba(255, 255, 255, 0.96);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  border: 3px solid #fff;
  padding: 15px;
  box-shadow: 0 10px 25px rgba(0,0,0,0.2);
  z-index: 1000;
  pointer-events: auto;
}
.dash-header { display: flex; justify-content: space-between; align-items: center; border-bottom: 2px dashed #f3f4f6; padding-bottom: 8px; margin-bottom: 12px;}
.dash-title { font-size: 18px; color: #374151; display: flex; align-items: center; gap: 8px; }
.mature-badge { background: #fde047; font-size: 11px; padding: 2px 8px; border-radius: 20px; color: #854d0e; font-weight: bold;}
.close-dash-btn { background: #f3f4f6; border: none; border-radius: 50%; width: 32px; height: 32px; cursor: pointer; font-size: 14px; color: #6b7280; font-weight: bold; transition: 0.2s;}
.close-dash-btn:hover { background: #ef4444; color: white;}

.dash-content { display: flex; flex-direction: column; gap: 12px; }

/* Dash Stats & Bars */
.dash-stats { display: grid; grid-template-columns: repeat(auto-fit, minmax(100px, 1fr)); gap: 10px; }
.stat-row { display: flex; align-items: center; gap: 6px; }
.stat-row span { font-size: 16px; line-height: 1; }
.bar-bg { flex: 1; height: 10px; background: #e5e7eb; border-radius: 5px; overflow: hidden; box-shadow: inset 0 2px 2px rgba(0,0,0,0.1); }
.bar-fill { height: 100%; border-radius: 5px; transition: width 0.3s, background-color 0.3s; }
.growth-fill { background: linear-gradient(90deg, #c084fc, #a855f7); }

/* Dash Buttons (เล่นในมือถือให้กดง่าย) */
.dash-actions { display: grid; grid-template-columns: repeat(auto-fit, minmax(75px, 1fr)); gap: 8px; }
.action-btn {
  border: none; padding: 6px; border-radius: 12px;
  color: white; font-weight: 800; cursor: pointer; box-shadow: 0 4px 0 rgba(0,0,0,0.15);
  font-family: 'Kanit'; display: flex; flex-direction: column; align-items: center; line-height: 1.2; font-size: 13px;
}
.action-btn small { font-size: 10px; font-weight: 600; opacity: 0.9;}
.action-btn:active:not(:disabled) { transform: translateY(4px); box-shadow: 0 0px 0 rgba(0,0,0,0.15); }
.action-btn:disabled { background: #d1d5db !important; cursor: not-allowed; box-shadow: none; transform: none; color: #9ca3af; }

.btn-feed { background: #34d399; } .btn-feed:hover:not(:disabled) { background: #10b981; }
.btn-play { background: #f472b6; } .btn-play:hover:not(:disabled) { background: #ec4899; }
.btn-clean { background: #2dd4bf; } .btn-clean:hover:not(:disabled) { background: #14b8a6; }
.btn-sleep { background: #818cf8; } .btn-sleep:hover:not(:disabled) { background: #6366f1; }
.sleeping-btn { background: #fbbf24 !important; box-shadow: 0 4px 0 #d97706;} .sleeping-btn:hover:not(:disabled) { background: #f59e0b !important; }
.btn-sell { background: #f87171; grid-column: 1 / -1;} .btn-sell:hover:not(:disabled) { background: #ef4444; }

.slide-down-enter-active, .slide-down-leave-active { transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
.slide-down-enter-from, .slide-down-leave-to { opacity: 0; transform: translate(-50%, -20px); }

/* ปุ่มซื้อเพิ่ม (Add Summon) */
.add-summon-btn { position: absolute; bottom: 30px; left: 50%; transform: translateX(-50%); background: transparent; border: none; cursor: pointer; outline: none; z-index: 1000; transition: transform 0.2s; }
.add-summon-btn:hover:not(:disabled) { transform: translateX(-50%) scale(1.05) translateY(-5px); }
.add-summon-btn:active:not(:disabled) { transform: translateX(-50%) scale(0.95); }
.add-summon-btn:disabled { opacity: 0.6; cursor: not-allowed; filter: grayscale(1); }
.summon-inner { background: linear-gradient(135deg, #fbbf24, #f59e0b); border: 4px solid white; width: 90px; height: 90px; border-radius: 50%; display: flex; flex-direction: column; justify-content: center; align-items: center; box-shadow: 0 6px 15px rgba(0,0,0,0.2); }
.plus-icon { font-size: 26px; font-weight: bold; color: white; line-height: 1; margin-bottom: 2px;}
.plus-text { font-size: 11px; color: white; text-align: center; line-height: 1.2;}

/* 🌟 Playground & Hamster Hitbox */
.playground { position: absolute; top: 0; left: 0; width: 100%; height: 100%; overflow: hidden; z-index: 10; }

/* สร้าง Hitbox รอบตัวหนูให้กว้างขึ้น เพื่อให้จิ้มง่ายๆ */
.hamster-entity { position: absolute; transform: translate(-50%, -50%); transition: z-index 0.2s; cursor: pointer; padding: 30px; }
.hamster-entity:hover { z-index: 999 !important; } 

.is-selected .img-container { filter: drop-shadow(0 0 15px rgba(253, 224, 71, 0.9)); }
.selection-arrow { position: absolute; top: -35px; left: 50%; transform: translateX(-50%); color: #fbbf24; font-size: 24px; animation: bounce-arrow 1s infinite; text-shadow: 0 2px 4px rgba(0,0,0,0.2);}
@keyframes bounce-arrow { 0%, 100% { transform: translateX(-50%) translateY(0); } 50% { transform: translateX(-50%) translateY(8px); } }

.hamster-bubble { position: relative; width: 140px; text-align: center; }
.img-container { position: relative; display: inline-block; padding-bottom: 10px; pointer-events: none; /* ป้องกันไปกวน Hitbox นอก */ }
.hamster-img-real { width: 130px; height: auto; object-fit: contain; filter: drop-shadow(0 10px 15px rgba(0,0,0,0.2)); z-index: 2; position: relative; transform-origin: center bottom; transition: filter 0.3s; }
.hamster-img-real.is-sleeping { filter: brightness(0.6) drop-shadow(0 10px 15px rgba(0,0,0,0.2)); }
.hamster-shadow { width: 70px; height: 10px; background: rgba(0,0,0,0.2); border-radius: 50%; position: absolute; bottom: 0px; left: 50%; transform: translateX(-50%); filter: blur(3px); z-index: 1; }
.mature-glow { position: absolute; top: -20px; left: -20px; right: -20px; bottom: -20px; border-radius: 50%; background: radial-gradient(circle, rgba(253,224,71,0.4) 0%, rgba(253,224,71,0) 70%); animation: pulse-glow 2s infinite alternate; z-index: -1; }
@keyframes pulse-glow { 0% { transform: scale(0.8); opacity: 0.5; } 100% { transform: scale(1.2); opacity: 1; } }

.hamster-label { margin-top: -5px; font-weight: 800; color: #4b5563; background: white; border-radius: 20px; padding: 4px 15px; display: inline-block; box-shadow: 0 4px 10px rgba(0,0,0,0.1); font-size: 13px; border: 2px solid #f3f4f6;}

/* Animations & Effects */
.hamster-spawn-enter-active { animation: pop-in 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
@keyframes pop-in { 0% { transform: scale(0) translateY(50px); opacity: 0; } 100% { transform: scale(1) translateY(0); opacity: 1; } }

.effect-icon { position: absolute; top: -30px; left: 50%; transform: translateX(-50%); font-size: 28px; z-index: 10; filter: drop-shadow(0 2px 4px rgba(0,0,0,0.2)); }
.pop-anim { animation: fly 1s forwards cubic-bezier(0.175, 0.885, 0.32, 1.275); }
@keyframes fly { 0% { transform: translate(-50%, 0) scale(0.5); opacity: 1; } 50% { transform: translate(-50%, -30px) scale(1.2); } 100% { transform: translate(-50%, -60px) scale(1); opacity: 0; } }

.bubble-anim { animation: float-bubble 1.5s forwards ease-out; font-size: 35px; }
@keyframes float-bubble { 0% { transform: translate(-50%, 0) scale(0.5); opacity: 1; } 100% { transform: translate(-50%, -80px) scale(1.5); opacity: 0; } }

.sleep-zzz { position: absolute; top: -40px; left: 50%; transform: translateX(-50%); font-size: 24px; font-weight: bold; color: #60a5fa; z-index: 10; display: flex; gap: 2px;}
.sleep-zzz span { animation: zzz-fly 1.5s infinite linear; opacity: 0; display: inline-block;}
@keyframes zzz-fly { 0% { transform: translate(0, 0) scale(0.5); opacity: 0; } 50% { opacity: 1; } 100% { transform: translate(15px, -30px) scale(1.2); opacity: 0; } }

.stink-cloud { position: absolute; top: 10px; right: 10px; font-size: 30px; animation: stink-wobble 2s infinite ease-in-out; z-index: 10; opacity: 0.8; filter: hue-rotate(90deg); pointer-events: none;}
@keyframes stink-wobble { 0% { transform: translate(0,0) scale(1); } 50% { transform: translate(10px, -10px) scale(1.1); } 100% { transform: translate(0,0) scale(1); } }

/* Media Query สำหรับจอมือถือเล็กๆ ให้เลย์เอาต์ไม่แตก */
@media (max-width: 400px) {
  .dash-actions { grid-template-columns: repeat(2, 1fr); }
  .btn-sell { grid-column: 1 / -1; }
  .hamster-entity { padding: 40px; } /* บนจอมือถือเล็ก ให้ hitbox ใหญ่ขึ้นไปอีก */
}
</style>