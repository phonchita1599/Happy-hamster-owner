<template>
  <div class="parallax-page">
    
    <div v-if="!isGameStarted">
      <svg class="parallax-svg" viewBox="0 0 800 600" preserveAspectRatio="xMidYMid slice">
        <defs>
          <filter id="softGlow" x="-20%" y="-20%" width="140%" height="140%">
            <feGaussianBlur stdDeviation="5" result="blur"/>
            <feComposite in="SourceGraphic" in2="blur" operator="over"/>
          </filter>
          <linearGradient id="bgGrad" x1="0%" y1="0%" x2="0%" y2="100%">
            <stop offset="0%" stop-color="#fffaf0" />
            <stop offset="100%" stop-color="#ffe6e6" />
          </linearGradient>
        </defs>

        <rect width="800" height="600" fill="url(#bgGrad)" />
        <circle id="shape-1" cx="150" cy="100" r="120" fill="#ffb7b2" opacity="0.6" filter="url(#softGlow)" />
        <circle id="shape-2" cx="700" cy="300" r="100" fill="#e2f0cb" opacity="0.6" filter="url(#softGlow)" />
        <circle id="shape-3" cx="200" cy="500" r="150" fill="#ffdac1" opacity="0.6" filter="url(#softGlow)" />

        <g id="hero-content">
          <text x="400" y="260" text-anchor="middle" class="hero-main-title">แฮปปี้คนเลี้ยงหนู</text>
          <text x="400" y="295" text-anchor="middle" class="hero-sub-title">2.5D PARALLAX FARMING GAME</text>
          <rect x="375" y="315" width="50" height="3" fill="#ff8b94" rx="1.5" />
        </g>
      </svg>

      <div class="scroll-element"></div>
      
      <div class="fixed-actions">
        <button @click="startGame" class="game-btn btn-play">
          <span class="btn-text">▶ เริ่มเล่นเกม</span>
        </button>
        <button @click="showTutorial = true" class="game-btn btn-info">
          <span class="btn-text">❓ วิธีการเล่น</span>
        </button>
      </div>

      <transition name="fade">
        <div v-if="showTutorial" class="tutorial-overlay" @click.self="showTutorial = false">
          <div class="tutorial-box">
            <h2>ยินดีต้อนรับสู่ฟาร์มแสนสุข! 🐹</h2>
            <p>ภารกิจของคุณคือการดูแลน้องหนูให้เติบโตอย่างแข็งแรง</p>
            <ul class="tutorial-list">
              <li><span>🌻</span> <b>ให้อาหาร (10G):</b> กดที่ตัวหนูแล้วเลือก "ให้กิน" เพื่อลดความหิว</li>
              <li><span>⏳</span> <b>การเติบโต:</b> หากหนูอิ่ม (ความหิว > 20) หนูจะค่อยๆ โตขึ้น</li>
              <li><span>💰</span> <b>ขายทำกำไร:</b> เมื่อหนูโตเต็มวัย (หลอดสีฟ้าเต็ม) จะขายได้ 250G</li>
              <li><span>➕</span> <b>ขยายฟาร์ม:</b> นำทองไปซื้อหนูตัวใหม่ (100G) มาเลี้ยงเพิ่ม</li>
            </ul>
            <button @click="showTutorial = false" class="game-btn btn-play" style="width: 100%; margin-top: 20px;">
              เข้าใจแล้ว ลุยเลย!
            </button>
          </div>
        </div>
      </transition>

    </div>

    <HamsterFarm v-else @back="backToHome" />

  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, nextTick, watch } from 'vue';
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import Lenis from '@studio-freight/lenis';
import HamsterFarm from './components/HamsterFarm.vue';

gsap.registerPlugin(ScrollTrigger);

const isGameStarted = ref(false);
const showTutorial = ref(false); // ควบคุมการแสดงหน้าต่างสอนเล่น

let lenisInstance = null;
let parallaxRafId = null;

const startGame = () => { isGameStarted.value = true; };
const backToHome = () => { isGameStarted.value = false; };

const initParallax = () => {
  lenisInstance = new Lenis({
    duration: 1.5,
    easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)),
    smoothWheel: true,
  });

  function raf(time) {
    if(!isGameStarted.value) { 
      lenisInstance.raf(time);
      parallaxRafId = requestAnimationFrame(raf);
    }
  }
  parallaxRafId = requestAnimationFrame(raf);

  gsap.to("#shape-1", { cy: -50, scrollTrigger: { trigger: ".scroll-element", start: "top top", end: "bottom top", scrub: 1 }});
  gsap.to("#shape-2", { cy: -200, scrollTrigger: { trigger: ".scroll-element", start: "top top", end: "bottom top", scrub: 1 }});
  gsap.to("#shape-3", { cy: -100, scrollTrigger: { trigger: ".scroll-element", start: "top top", end: "bottom top", scrub: 1 }});
};

onMounted(() => { initParallax(); });

watch(isGameStarted, (newVal) => {
  if (newVal) {
    if (lenisInstance) { lenisInstance.destroy(); lenisInstance = null; }
    if (parallaxRafId) cancelAnimationFrame(parallaxRafId);
    ScrollTrigger.getAll().forEach(t => t.kill());
    nextTick(() => { window.scrollTo(0, 0); document.body.style.overflow = "auto"; });
  } else {
    nextTick(() => { window.scrollTo(0, 0); initParallax(); });
  }
});

onUnmounted(() => {
  if (lenisInstance) lenisInstance.destroy();
  if (parallaxRafId) cancelAnimationFrame(parallaxRafId);
  ScrollTrigger.getAll().forEach(t => t.kill());
});
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;500;800&family=Kanit:wght@300;400;600;800&display=swap');
body { margin: 0; padding: 0; background-color: #fffaf0; overflow-x: hidden; font-family: 'Kanit', sans-serif; }
</style>

<style scoped>
.parallax-page { width: 100%; min-height: 100vh; }
.parallax-svg { position: fixed; top: 0; left: 0; width: 100%; height: 100vh; z-index: 1; pointer-events: none; }
.scroll-element { height: 150vh; position: relative; z-index: 2; } /* ลดความสูงลงเพื่อไม่ให้ต้องเลื่อนยาว */

.hero-main-title { font-size: 70px; font-weight: 800; fill: #ff8b94; filter: drop-shadow(0px 4px 0px #d46b73); }
.hero-sub-title { font-family: 'Inter', sans-serif; font-size: 14px; font-weight: 800; fill: #a5938d; letter-spacing: 4px; }

/* 🌟 สไตล์ปุ่มแบบเกม (Game-like Buttons) */
.fixed-actions { position: fixed; bottom: 50px; left: 50%; transform: translateX(-50%); z-index: 100; display: flex; gap: 15px; flex-wrap: wrap; justify-content: center;}
.game-btn {
  border: none; padding: 15px 30px; border-radius: 50px; cursor: pointer;
  font-family: 'Kanit', sans-serif; font-size: 18px; font-weight: 800;
  transition: all 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  box-shadow: 0 6px 0 rgba(0,0,0,0.15), 0 15px 20px rgba(0,0,0,0.1);
  color: white; text-transform: uppercase;
}
.game-btn:active { transform: translateY(6px); box-shadow: 0 0px 0 rgba(0,0,0,0.15), 0 5px 10px rgba(0,0,0,0.1); }
.btn-play { background: linear-gradient(135deg, #ff8b94, #ff5e6c); }
.btn-play:hover { background: linear-gradient(135deg, #ff9ca4, #ff6f7c); transform: translateY(-3px) scale(1.05); }
.btn-info { background: linear-gradient(135deg, #a8e6cf, #6ee7b7); color: #374151;}
.btn-info:hover { background: linear-gradient(135deg, #bbf7d0, #86efac); transform: translateY(-3px) scale(1.05); }

/* 🌟 สไตล์หน้าต่างสอนเล่น (Tutorial Modal) */
.tutorial-overlay {
  position: fixed; top: 0; left: 0; width: 100%; height: 100%;
  background: rgba(0,0,0,0.6); backdrop-filter: blur(5px);
  display: flex; justify-content: center; align-items: center; z-index: 999;
}
.tutorial-box {
  background: white; padding: 40px; border-radius: 30px; width: 90%; max-width: 450px;
  box-shadow: 0 20px 50px rgba(0,0,0,0.3); border: 5px solid #ffdac1;
  text-align: center; color: #5d4037;
}
.tutorial-box h2 { color: #ff8b94; margin-top: 0; font-size: 28px; }
.tutorial-list { list-style: none; padding: 0; text-align: left; margin-top: 20px;}
.tutorial-list li { background: #f0fdf4; margin-bottom: 10px; padding: 15px; border-radius: 15px; font-size: 16px; display: flex; align-items: center; gap: 10px; border-left: 5px solid #6ee7b7;}
.tutorial-list li span { font-size: 24px; }

.fade-enter-active, .fade-leave-active { transition: opacity 0.3s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
</style>