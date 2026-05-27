<template>
  <div class="app">
    <div class="container">
      <h1 class="title">🎨 AI智能修图 V1.0</h1>
      <p class="subtitle">让AI为爱发电 ✨</p>

      <!-- 图片容器：原图 + 结果图叠加 -->
      <div
        class="image-stage"
        :class="{ 'has-image': imageUrl, dragging, 'hide-interactive': done || processing }"
        @click="triggerUpload"
        @dragover.prevent="dragging = true"
        @dragleave="dragging = false"
        @drop.prevent="handleDrop"
      >
        <template v-if="!imageUrl">
          <div class="upload-placeholder">
            <span class="upload-icon">📷</span>
            <p>点击或拖拽上传图片</p>
          </div>
        </template>
        <template v-else>
          <img :src="imageUrl" class="preview-image" alt="预览" />
        </template>
        <input
          ref="fileInput"
          type="file"
          accept="image/*"
          style="display: none"
          @change="handleFileChange"
        />

        <!-- 结果图覆盖层 -->
        <div
          v-if="revealActive"
          class="result-overlay"
          :class="{ 'reveal-done': revealDone }"
        >
          <img :src="resultImageUrl" class="result-image" alt="修图结果" />
          <!-- 扫描光线 -->
          <div v-if="!revealDone" class="reveal-line"></div>
        </div>
      </div>

      <!-- 输入需求 -->
      <transition name="slide-fade">
        <div v-if="!done" class="input-section">
          <div class="input-wrapper">
            <span class="input-icon">✏️</span>
            <input
              v-model="prompt"
              type="text"
              class="prompt-input"
              placeholder="请输入您的修图需求..."
              :disabled="processing"
            />
          </div>
        </div>
      </transition>

      <!-- 一键修图按钮 -->
      <transition name="slide-fade">
        <button
          v-if="!done"
          class="magic-btn"
          :class="{ processing }"
          :disabled="!canProcess"
          @click="startProcess"
        >
          <span v-if="!processing">🪄 一键修图</span>
          <span v-else class="btn-loading">🔮 AI正在处理中...</span>
        </button>
      </transition>

      <!-- 进度条 -->
      <transition name="slide-fade">
        <div v-if="processing" class="progress-section">
          <div class="progress-bar-wrapper">
            <div class="progress-bar" :style="{ width: progress + '%' }"></div>
            <span class="progress-text">{{ progress }}%</span>
          </div>
          <p class="loading-text">{{ loadingText }}</p>
        </div>
      </transition>

      <!-- 结果提示 -->
      <transition name="msg-pop">
        <div v-if="done" class="result-area">
          <div class="result-message">
            <span class="result-icon"></span>
            <p>AI提示：成功！ 将图中男人变成了单身狗</p>
            <span class="result-icon"></span>
          </div>
          <button class="retry-btn" @click="resetAll">🔄 再P一个</button>
        </div>
      </transition>
    </div>

    <!-- 粒子背景 -->
    <div class="particles">
      <div v-for="i in 20" :key="i" class="particle" :style="particleStyle(i)"></div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      imageUrl: null,
      prompt: '',
      processing: false,
      done: false,
      progress: 0,
      dragging: false,
      revealActive: false,
      revealDone: false,
      resultImageUrl: '/ps1.png',
      loadingTexts: [
        '正在扫描图片...',
        '正在加载用户需求...',
        'AI大模型生成中...',
      ],
      loadingText: '',
      loadingPhase: 0,
    }
  },
  computed: {
    canProcess() {
      return this.imageUrl && this.prompt.trim() && !this.processing
    },
  },
  methods: {
    triggerUpload() {
      if (!this.processing && !this.done) {
        this.$refs.fileInput.click()
      }
    },
    handleFileChange(e) {
      const file = e.target.files[0]
      if (file) {
        this.imageUrl = URL.createObjectURL(file)
        this.done = false
        this.revealActive = false
        this.revealDone = false
      }
    },
    handleDrop(e) {
      this.dragging = false
      const file = e.dataTransfer.files[0]
      if (file && file.type.startsWith('image/')) {
        this.imageUrl = URL.createObjectURL(file)
        this.done = false
        this.revealActive = false
        this.revealDone = false
      }
    },
    startProcess() {
      if (!this.canProcess) return
      this.processing = true
      this.done = false
      this.revealActive = false
      this.revealDone = false
      this.progress = 0
      this.loadingPhase = 0
      this.loadingText = this.loadingTexts[0]

      const totalDuration = 5000
      const interval = 50
      const steps = totalDuration / interval
      let currentStep = 0

      const timer = setInterval(() => {
        currentStep++

        const t = currentStep / steps
        if (t < 0.3) {
          this.progress = Math.floor(t / 0.3 * 33)
          this.loadingPhase = 0
        } else if (t < 0.65) {
          this.progress = Math.floor(33 + (t - 0.3) / 0.35 * 33)
          this.loadingPhase = 1
        } else if (t < 0.95) {
          this.progress = Math.floor(66 + (t - 0.65) / 0.3 * 33)
          this.loadingPhase = 2
        } else {
          this.progress = 99
        }

        this.loadingText = this.loadingTexts[this.loadingPhase]

        if (this.progress >= 99 && currentStep < steps + 20) {
          this.progress = 99
        }

        if (currentStep >= steps + 20) {
          clearInterval(timer)
          this.progress = 100
          this.loadingText = '修图完成！'

          // 开始圆形扩散覆盖
          setTimeout(() => {
            this.revealActive = true
            this.processing = false
          }, 400)

          // 扩散完成
          setTimeout(() => {
            this.revealDone = true
            this.done = true
          }, 1800)
        }
      }, interval)
    },
    particleStyle(i) {
      const size = Math.random() * 6 + 2
      return {
        left: `${Math.random() * 100}%`,
        top: `${Math.random() * 100}%`,
        width: `${size}px`,
        height: `${size}px`,
        animationDelay: `${Math.random() * 6}s`,
        animationDuration: `${Math.random() * 4 + 4}s`,
      }
    },
    resetAll() {
      this.imageUrl = null
      this.prompt = ''
      this.processing = false
      this.done = false
      this.progress = 0
      this.dragging = false
      this.revealActive = false
      this.revealDone = false
    },
  },
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@400;500;700;900&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Noto Sans SC', -apple-system, BlinkMacSystemFont, sans-serif;
  min-height: 100vh;
  background: linear-gradient(135deg, #0f0c29 0%, #302b63 50%, #24243e 100%);
  color: #fff;
  overflow-x: hidden;
}

.app {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 40px 20px;
  position: relative;
}

.container {
  max-width: 640px;
  width: 100%;
  text-align: center;
  position: relative;
  z-index: 2;
}

.title {
  font-size: 2.2rem;
  font-weight: 900;
  background: linear-gradient(135deg, #f093fb, #f5576c, #ffd200);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 8px;
  letter-spacing: 2px;
}

.subtitle {
  font-size: 0.95rem;
  color: rgba(255, 255, 255, 0.6);
  margin-bottom: 36px;
  letter-spacing: 1px;
}

/* 图片舞台：两层叠加，手机比例 */
.image-stage {
  position: relative;
  width: 100%;
  max-width: 340px;
  margin: 0 auto 24px;
  aspect-ratio: 9 / 16;
  border-radius: 24px;
  overflow: hidden;
  background: rgba(255, 255, 255, 0.04);
  border: 2px dashed rgba(255, 255, 255, 0.25);
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
  cursor: pointer;
}

.image-stage:hover:not(.hide-interactive) {
  border-color: rgba(240, 147, 251, 0.6);
  box-shadow: 0 8px 30px rgba(240, 147, 251, 0.15);
}

.image-stage.dragging {
  border-color: #f093fb;
  background: rgba(240, 147, 251, 0.1);
}

.image-stage.has-image {
  border-style: solid;
  border-color: rgba(255, 255, 255, 0.1);
}

.image-stage.hide-interactive {
  cursor: default;
  border-color: rgba(255, 255, 255, 0.1);
}

.upload-placeholder {
  text-align: center;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.upload-icon {
  font-size: 2.5rem;
  display: block;
  margin-bottom: 12px;
}

.upload-placeholder p {
  color: rgba(255, 255, 255, 0.5);
  font-size: 0.95rem;
}

.preview-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

/* 结果覆盖层：从上往下覆盖 */
.result-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 5;
  overflow: hidden;
  /* 初始：从底部裁剪100%，完全隐藏 */
  clip-path: inset(0 0 100% 0);
  animation: slideReveal 1.2s cubic-bezier(0.33, 1, 0.68, 1) forwards;
}

.result-overlay.reveal-done {
  clip-path: inset(0 0 0 0);
  animation: none;
}

@keyframes slideReveal {
  0% {
    clip-path: inset(0 0 100% 0);
  }
  100% {
    clip-path: inset(0 0 0 0);
  }
}

.result-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

/* 覆盖扫描光线 */
.reveal-line {
  position: absolute;
  left: 0;
  width: 100%;
  height: 3px;
  background: linear-gradient(90deg, transparent, rgba(240, 147, 251, 0.9), rgba(245, 87, 108, 0.9), rgba(255, 210, 0, 0.9), transparent);
  box-shadow: 0 0 20px rgba(240, 147, 251, 0.8), 0 0 40px rgba(240, 147, 251, 0.4);
  top: 0;
  animation: lineSweep 1.2s cubic-bezier(0.33, 1, 0.68, 1) forwards;
  pointer-events: none;
  z-index: 10;
}

@keyframes lineSweep {
  0% {
    top: 0%;
  }
  100% {
    top: 100%;
  }
}

/* 输入区域 */
.input-section {
  margin-bottom: 20px;
}

.input-wrapper {
  display: flex;
  align-items: center;
  background: rgba(255, 255, 255, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 14px;
  padding: 4px 6px 4px 16px;
  transition: all 0.3s ease;
}

.input-wrapper:focus-within {
  border-color: rgba(240, 147, 251, 0.5);
  box-shadow: 0 0 20px rgba(240, 147, 251, 0.15);
  background: rgba(255, 255, 255, 0.1);
}

.input-icon {
  font-size: 1.1rem;
  margin-right: 8px;
}

.prompt-input {
  flex: 1;
  background: none;
  border: none;
  outline: none;
  color: #fff;
  font-size: 0.95rem;
  padding: 12px 0;
  font-family: inherit;
}

.prompt-input::placeholder {
  color: rgba(255, 255, 255, 0.35);
}

.prompt-input:disabled {
  opacity: 0.5;
}

/* 按钮 */
.magic-btn {
  width: 100%;
  padding: 14px 28px;
  font-size: 1.05rem;
  font-weight: 700;
  font-family: inherit;
  border: none;
  border-radius: 14px;
  cursor: pointer;
  background: linear-gradient(135deg, #f093fb, #f5576c);
  color: #fff;
  transition: all 0.3s ease;
  letter-spacing: 2px;
  position: relative;
  overflow: hidden;
}

.magic-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
  transition: left 0.5s;
}

.magic-btn:hover:not(:disabled)::before {
  left: 100%;
}

.magic-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 8px 30px rgba(245, 87, 108, 0.4);
}

.magic-btn:active:not(:disabled) {
  transform: translateY(0);
}

.magic-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.magic-btn.processing {
  background: linear-gradient(135deg, #667eea, #764ba2);
}

.btn-loading {
  display: inline-flex;
  align-items: center;
  gap: 6px;
}

/* 进度条 */
.progress-section {
  margin-top: 28px;
}

.progress-bar-wrapper {
  width: 100%;
  height: 28px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 14px;
  overflow: hidden;
  position: relative;
}

.progress-bar {
  height: 100%;
  background: linear-gradient(90deg, #f093fb, #f5576c, #ffd200);
  border-radius: 14px;
  transition: width 0.15s ease;
  position: relative;
}

.progress-bar::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(255, 255, 255, 0.3),
    transparent
  );
  animation: shimmer 1.5s infinite;
}

@keyframes shimmer {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(100%); }
}

.progress-text {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 0.8rem;
  font-weight: 700;
  color: #fff;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.5);
}

.loading-text {
  margin-top: 14px;
  font-size: 0.95rem;
  color: rgba(255, 255, 255, 0.7);
  animation: pulse 1.5s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

/* 结果区域 */
.result-area {
  margin-top: 24px;
  display: inline-flex;
  flex-direction: column;
  align-items: center;
  gap: 14px;
}

.result-message {
  display: inline-flex;
  align-items: center;
  gap: 12px;
}

.result-icon {
  font-size: 1.4rem;
}

.result-message p {
  font-size: 1.1rem;
  font-weight: 800;
  color: #ffd200;
  filter: drop-shadow(0 0 8px rgba(255, 210, 0, 0.5));
}

.retry-btn {
  margin-top: 12px;
  padding: 10px 28px;
  font-size: 1rem;
  font-weight: 700;
  font-family: inherit;
  border: 2px solid rgba(240, 147, 251, 0.5);
  border-radius: 12px;
  cursor: pointer;
  background: rgba(240, 147, 251, 0.1);
  color: #f093fb;
  transition: all 0.3s ease;
  letter-spacing: 1px;
}

.retry-btn:hover {
  background: rgba(240, 147, 251, 0.25);
  border-color: #f093fb;
  box-shadow: 0 0 20px rgba(240, 147, 251, 0.3);
  transform: translateY(-2px);
}

/* 过渡动画 */
.slide-fade-enter-active {
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.slide-fade-leave-active {
  transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
}

.slide-fade-enter-from {
  opacity: 0;
  transform: translateY(16px);
}

.slide-fade-leave-to {
  opacity: 0;
  transform: translateY(-12px) scale(0.97);
}

.msg-pop-enter-active {
  transition: all 0.5s cubic-bezier(0.16, 1, 0.3, 1);
}

.msg-pop-leave-active {
  transition: all 0.3s ease;
}

.msg-pop-enter-from {
  opacity: 0;
  transform: translateY(16px) scale(0.85);
}

.msg-pop-leave-to {
  opacity: 0;
}

/* 粒子背景 */
.particles {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 1;
}

.particle {
  position: absolute;
  background: rgba(240, 147, 251, 0.3);
  border-radius: 50%;
  animation: float linear infinite;
}

@keyframes float {
  0%, 100% {
    transform: translateY(0) translateX(0);
    opacity: 0;
  }
  10% {
    opacity: 1;
  }
  90% {
    opacity: 1;
  }
  50% {
    transform: translateY(-80px) translateX(30px);
    opacity: 0.6;
  }
}
</style>
