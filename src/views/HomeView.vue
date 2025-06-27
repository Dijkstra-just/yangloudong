<script setup lang="ts">
import { ref, onMounted } from 'vue'
import ThreeDViewer from '../components/ThreeDViewer.vue'

// 景点数据接口
interface Attraction {
  id: string
  name: string
  description: string
  history: string
  features: string[]
  images: string[]
  modelUrl?: string  // 3D模型URL
  x: number
  y: number
}

// 页面状态控制
const isInitialView = ref(true)
const showModal = ref(false)
const show3DModel = ref(false)  // 是否显示3D模型
const selectedAttraction = ref<Attraction | null>(null)
const currentImageIndex = ref(0)

// 景点数据
const attractions = ref<Attraction[]>([
  {
    id: 'yangloudong-party-service-center',
    name: '羊楼洞村党群服务中心',
    description: '羊楼洞村的党群服务中心，为村民提供各类公共服务，是村民活动的重要场所。',
    history: '现代建设的服务设施，体现了新时代乡村治理的成果。',
    features: ['党建服务', '村民活动', '公共服务'],
    images: ['/bwg1.jpg', '/bwg2.jpg'],
    modelUrl: 'https://lumalabs.ai/capture/your-model-1', // 替换为实际的模型URL
    x: 42,
    y: 17
  },
  {
    id: 'yangloudong-exhibition-center',
    name: '羊楼洞国际会展中心',
    description: '展示羊楼洞茶文化和历史的现代化展览中心，是了解古镇文化的重要窗口。',
    history: '为展示羊楼洞丰富的茶文化历史而建设的现代化场馆。',
    features: ['茶文化展示', '历史文物', '互动体验'],
    images: ['/bwg1.jpg', '/bwg2.jpg'],
    modelUrl: 'https://lumalabs.ai/capture/your-model-2',
    x: 35,
    y: 34
  },
  {
    id: 'mingqing-stone-street',
    name: '明清石板街',
    description: '羊楼洞最具特色的古街道，青石板铺就，承载着千年历史。街道两旁保存着明清时期的古建筑，是体验古镇风貌的最佳去处。',
    history: '始建于明清时期，长约2200米，是当时茶叶贸易的主要通道。青石板路面被历代运茶的"鸡公车"碾成寸余深槽，见证了古镇的繁华历史。',
    features: ['青石板路', '明清建筑', '茶马古道遗迹', '历史文化街区'],
    images: ['/sbj1.jpg', '/sbj2.jpeg','/sbj3.png'],
    modelUrl: 'https://lumalabs.ai/capture/20cc6bdd-638a-4c29-93d7-f9fae106384e',
    x: 60,
    y: 56
  },
  {
    id: 'china-brick-tea-museum',
    name: '中国青砖茶博物馆',
    description: '中国首个以青砖茶为主题的专业博物馆，全面展示青砖茶的历史、制作工艺和文化内涵。',
    history: '青砖茶有200多年的生产历史，羊楼洞是青砖茶的发源地。博物馆详细展示了从茶叶种植到成品销售的全过程。',
    features: ['青砖茶历史', '制茶工艺', '茶文化展示', '互动体验'],
    images: ['/bwg1.jpg', '/bwg2.jpg'],
    modelUrl: 'https://lumalabs.ai/capture/b2e36f28-529a-483f-8e20-62b253265627',
    x: 80,
    y: 50
  },
  {
    id: 'yangloudong-primary-school',
    name: '羊楼洞小学',
    description: '历史悠久的乡村小学，培育了一代代羊楼洞的子弟，是古镇教育文化的重要组成部分。',
    history: '学校历史可追溯至清末民初，见证了古镇教育事业的发展历程。',
    features: ['教育历史', '乡村文化', '人才培养'],
    images: ['/bwg1.jpg', '/bwg2.jpg'],
    modelUrl: 'https://lumalabs.ai/capture/your-model-5',
    x: 74,
    y: 57
  },
  {
    id: 'lei-family-courtyard',
    name: '雷家大院',
    description: '羊楼洞著名的古民居建筑群，展现了明清时期商人家族的生活风貌和建筑艺术。',
    history: '雷家是羊楼洞的名门望族，曾经营茶庄生意。大院建筑精美，布局合理，是研究明清民居的重要实物。',
    features: ['古民居建筑', '商贾文化', '传统工艺', '家族历史'],
    images: ['/ljdy1.jpg', '/ljdy2.jpg'],
    modelUrl: 'https://lumalabs.ai/capture/55d64516-88e6-4633-9c6c-f04ef01426b6',
    x: 78,
    y: 85
  },
  {
    id: 'guantong-temple',
    name: '圆通寺',
    description: '羊楼洞的古老寺庙，是当地佛教文化的重要载体，也是村民精神寄托之所。',
    history: '寺庙历史悠久，经历多次重建，承载着深厚的宗教文化内涵。',
    features: ['佛教文化', '古建筑', '宗教活动', '精神寄托'],
    images: ['/yts1.jpg', '/yts2.png'],
    modelUrl: 'https://lumalabs.ai/capture/831a4d86-e66d-4119-8646-d3f919a16e5c',
    x: 60,
    y: 83
  },
  {
    id: 'guanyin-spring',
    name: '观音泉',
    description: '羊楼洞著名的天然泉水，水质清澈甘甜，是历代制茶的优质水源，被誉为"茶泉"。',
    history: '泉水自古以来就是羊楼洞制茶的重要水源，泉水甘甜清冽，为青砖茶的品质提供了保障。民间传说此泉与观音菩萨有关。',
    features: ['天然泉水', '制茶水源', '民间传说', '生态景观'],
    images: ['/gyq1.png', '/gyq2.png'],
    modelUrl: 'https://lumalabs.ai/capture/your-model-8',
    x: 90,
    y: 78
  }
])

// 进入完整视图
const enterFullView = () => {
  isInitialView.value = false
}

// 返回初始视图
const goBackToInitial = () => {
  isInitialView.value = true
  closeModal()
}

// 点击景点标记
const selectAttraction = (attraction: Attraction) => {
  selectedAttraction.value = attraction
  currentImageIndex.value = 0
  show3DModel.value = false
  showModal.value = true
}

// 关闭弹窗
const closeModal = () => {
  showModal.value = false
  selectedAttraction.value = null
  currentImageIndex.value = 0
  show3DModel.value = false
}

// 切换到3D模型视图
const view3DModel = () => {
  show3DModel.value = !show3DModel.value
}

// 图片轮播控制
const nextImage = () => {
  if (selectedAttraction.value && currentImageIndex.value < selectedAttraction.value.images.length - 1) {
    currentImageIndex.value++
  }
}

const prevImage = () => {
  if (currentImageIndex.value > 0) {
    currentImageIndex.value--
  }
}

const goToImage = (index: number) => {
  currentImageIndex.value = index
}

onMounted(() => {
  console.log('羊楼洞古镇页面加载完成')
})
</script>

<template>
  <div class="yangloudong-site">
    <!-- 网站头部 -->
          <header class="site-header">
        <div class="container">
          <div class="title-row">
            <h1 class="site-title">羊楼洞古镇</h1>
            <span class="site-subtitle">世界茶业第一古镇 · 中国砖茶之乡</span>
          </div>
          <button 
            v-if="!isInitialView" 
            class="back-btn"
            @click="goBackToInitial"
          >
            ← 返回首页
          </button>
        </div>
      </header>

    <!-- 主要内容区域 -->
    <main class="main-content" :class="{ 'full-view': !isInitialView }">
      
      <!-- 初始视图 -->
      <div v-if="isInitialView" class="initial-view">
        <div class="container">
          <div class="initial-content">
            <div class="initial-image-container">
              <img src="/羊楼洞示意.png" alt="羊楼洞古镇示意图" class="initial-overview-image">
            </div>
            <button class="next-btn" @click="enterFullView">
              <span>开始探索</span>
              <span class="arrow">→</span>
            </button>
          </div>
        </div>
      </div>

      <!-- 完整视图 -->
      <div v-else class="full-content">
        <!-- 古镇总体介绍 -->
        <section class="intro-section">
          <div class="container">
            <div class="intro-grid">
              <div class="intro-text">
                <h2>千年古镇 茶韵悠长</h2>
                <p class="intro-description">
                  羊楼洞古镇位于湘鄂交界要冲，是赤壁市"六大古镇"之一，为"松峰茶"和"洞茶"原产地，
                  是湖北老青茶的发源地和"世界茶业第一古镇"，素有"中国砖茶之乡"的美称。
                </p>
                <p class="intro-description">
                  古镇在清朝年间，就已是一个人口超过四万，拥有200家茶庄的国际贸易集散地。
                  在《大清皇舆全图》上，羊楼洞的标记与汉口、武昌是一样的规格。羊楼洞的兴衰史就是一部茶叶的传奇史。
                </p>
                <div class="features-list">
                  <div class="feature-item">
                    <span class="feature-icon">🏛️</span>
                    <span>明清古建筑群</span>
                  </div>
                  <div class="feature-item">
                    <span class="feature-icon">🍃</span>
                    <span>青砖茶发源地</span>
                  </div>
                  <div class="feature-item">
                    <span class="feature-icon">🐎</span>
                    <span>茶马古道重镇</span>
                  </div>
                  <div class="feature-item">
                    <span class="feature-icon">💧</span>
                    <span>观音泉天然好水</span>
                  </div>
                </div>
              </div>
              <div class="intro-image">
                <img src="/羊楼洞示意.png" alt="羊楼洞古镇示意图" class="overview-image">
                <p class="image-caption">羊楼洞古镇区位示意图</p>
              </div>
            </div>
          </div>
        </section>

        <!-- 互动地图区域 -->
        <section class="map-section">
          <div class="container">
            <h2 class="section-title">古镇景点导览</h2>
            <p class="section-subtitle">点击地图上的标记点，了解各个景点的详细信息</p>
            
            <div class="interactive-map">
              <img 
                src="/古镇旅游地图.jpeg" 
                alt="羊楼洞古镇旅游地图" 
                class="map-image"
              >
              
              <!-- 景点标记点 -->
              <div 
                v-for="attraction in attractions" 
                :key="attraction.id"
                class="attraction-marker"
                :style="{ left: attraction.x + '%', top: attraction.y + '%' }"
                @click="selectAttraction(attraction)"
                :title="attraction.name"
              >
                <div class="marker-dot"></div>
                <div class="marker-label">{{ attraction.name }}</div>
              </div>
            </div>
          </div>
        </section>

        <!-- 景点特色展示 -->
        <section class="highlights-section">
          <div class="container">
            <h2 class="section-title">古镇特色</h2>
            <div class="highlights-grid">
              <div class="highlight-card">
                <div class="highlight-icon">🧱</div>
                <h3>青砖茶文化</h3>
                <p>羊楼洞是青砖茶的发源地，已有200多年的生产历史。青砖茶以老青茶为原料，经蒸汽高温压制而成，远销内蒙古、新疆、西藏等地。</p>
              </div>
              <div class="highlight-card">
                <div class="highlight-icon">🏘️</div>
                <h3>明清古街</h3>
                <p>保存完好的明清石板街，街面全部以青石铺设，被历代运茶的"鸡公车"碾成寸余深槽，见证了古镇的繁华历史。</p>
              </div>
              <div class="highlight-card">
                <div class="highlight-icon">🌊</div>
                <h3>观音泉水</h3>
                <p>位于松峰山下的观音泉，水质清澈甘醇，是历代精制名茶的优质水源，为青砖茶的独特品质提供了保障。</p>
              </div>
              <div class="highlight-card">
                <div class="highlight-icon">🛤️</div>
                <h3>茶马古道</h3>
                <p>古镇是茶马古道上的重要驿站，连接着万里茶路，从这里运出的茶砖远销俄罗斯、蒙古等国，成就了国际贸易的传奇。</p>
              </div>
            </div>
          </div>
        </section>
      </div>
    </main>

    <!-- 景点详情弹窗 -->
    <div v-if="showModal" class="modal-overlay" @click="closeModal">
      <div class="modal-content" @click.stop :class="{ 'fullscreen-model': show3DModel }">
        <div class="modal-header">
          <h3>{{ selectedAttraction?.name }}</h3>
          <div class="header-controls">
            <!-- 3D模型切换按钮 -->
            <button 
              v-if="selectedAttraction?.modelUrl"
              class="model-toggle-btn"
              @click="view3DModel"
              :class="{ active: show3DModel }"
            >
              {{ show3DModel ? '查看详情' : '3D 模型' }}
            </button>
            <button class="close-btn" @click="closeModal">&times;</button>
          </div>
        </div>
        
        <div class="modal-body">
          <!-- 3D模型展示区域 (全屏模式) -->
          <div v-if="show3DModel && selectedAttraction?.modelUrl" class="model-viewer-fullscreen">
            <ThreeDViewer :source="selectedAttraction.modelUrl" />
          </div>

          <!-- 常规内容区域 (仅在非3D模式显示) -->
          <div v-else class="regular-content">
            <!-- 图片轮播区域 -->
            <div v-if="selectedAttraction?.images.length" class="image-carousel">
              <div class="carousel-container">
                <div class="carousel-images">
                  <img 
                    v-for="(image, index) in selectedAttraction.images" 
                    :key="index"
                    :src="image" 
                    :alt="`${selectedAttraction.name} - 图片 ${index + 1}`"
                    class="carousel-image"
                    :style="{ transform: `translateX(${(index - currentImageIndex) * 100}%)` }"
                  >
                </div>
                
                <!-- 左右切换按钮 -->
                <button 
                  v-if="currentImageIndex > 0"
                  class="carousel-btn prev-btn" 
                  @click="prevImage"
                >
                  ‹
                </button>
                <button 
                  v-if="currentImageIndex < selectedAttraction.images.length - 1"
                  class="carousel-btn next-btn" 
                  @click="nextImage"
                >
                  ›
                </button>
                
                <!-- 指示器 -->
                <div class="carousel-indicators">
                  <button
                    v-for="(image, index) in selectedAttraction.images"
                    :key="index"
                    class="indicator"
                    :class="{ active: index === currentImageIndex }"
                    @click="goToImage(index)"
                  ></button>
                </div>
              </div>
            </div>

            <!-- 景点信息 -->
            <div class="attraction-info">
              <div class="info-section">
                <h4>景点介绍</h4>
                <p>{{ selectedAttraction?.description }}</p>
              </div>
              <div class="info-section">
                <h4>历史沿革</h4>
                <p>{{ selectedAttraction?.history }}</p>
              </div>
              <div class="info-section">
                <h4>特色亮点</h4>
                <ul class="features-list">
                  <li v-for="feature in selectedAttraction?.features" :key="feature">
                    {{ feature }}
                  </li>
                </ul>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 网站底部 -->
    <footer class="site-footer">
      <div class="container">
        <p>&copy; 2024 羊楼洞古镇. 传承千年茶文化，展现古镇新风貌</p>
      </div>
    </footer>
  </div>
</template>

<style scoped>
.yangloudong-site {
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
}

.container {
  margin: 0 auto;
  padding: 0 20px;
}

/* 头部样式 */
.site-header {
  background: linear-gradient(135deg, #1e3c72 0%, #2a5298 50%, #1e3c72 100%);
  color: white;
  text-align: center;
  position: relative;
  box-shadow: 0 4px 20px rgba(0,0,0,0.15);
  border-bottom: 3px solid #f39c12;
}

.site-header::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: url("data:image/svg+xml,%3Csvg width='40' height='40' viewBox='0 0 40 40' xmlns='http://www.w3.org/2000/svg'%3E%3Cg fill='%23ffffff' fill-opacity='0.05' fill-rule='evenodd'%3E%3Cpath d='M20 20c0 11.046-8.954 20-20 20v-40c11.046 0 20 8.954 20 20zM0 20c0-11.046 8.954-20 20-20v40c-11.046 0-20-8.954-20-20z'/%3E%3C/g%3E%3C/svg%3E") repeat;
  pointer-events: none;
}

.container {
  position: relative;
  z-index: 2;
}

.title-row {
  display: flex;
  align-items: baseline;
  justify-content: center;
  gap: 25px;
  flex-wrap: wrap;
  margin-bottom: 10px;
}

.site-title {
  font-size: 3.2rem;
  margin: 0;
  text-shadow: 2px 2px 8px rgba(0,0,0,0.4);
  font-weight: 900;
  background: linear-gradient(45deg, #fff, #f39c12, #fff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  position: relative;
}

.site-title::after {
  content: '';
  position: absolute;
  bottom: -5px;
  left: 50%;
  transform: translateX(-50%);
  width: 80px;
  height: 3px;
  background: linear-gradient(90deg, transparent, #f39c12, transparent);
  border-radius: 2px;
}

.site-subtitle {
  font-size: 1.1rem;
  opacity: 0.95;
  white-space: nowrap;
  color: #ecf0f1;
  font-weight: 500;
  letter-spacing: 1px;
  margin-top: 5px;
}

.back-btn {
  position: absolute;
  left: 40px;
  top: 50%;
  transform: translateY(-50%);
  background: linear-gradient(135deg, #e74c3c, #c0392b);
  border: 2px solid rgba(255,255,255,0.3);
  color: white;
  padding: 12px 24px;
  border-radius: 30px;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 600;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(231,76,60,0.3);
}

.back-btn:hover {
  background: linear-gradient(135deg, #c0392b, #e74c3c);
  transform: translateY(-50%) translateY(-2px);
  box-shadow: 0 6px 20px rgba(231,76,60,0.4);
}

/* 主要内容样式 */
.main-content {
  transition: all 0.8s ease-in-out;
}

/* 初始视图样式 */
.initial-view {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 80vh;
  padding: 20px 0;
}

.initial-content {
  text-align: center;
}

.initial-image-container {
  position: relative;
  display: inline-block;
}

.initial-overview-image {
  width: 100%;
  border-radius: 20px;
  box-shadow: 0 20px 60px rgba(0,0,0,0.3);
  transition: all 0.8s ease;
}

.next-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border: none;
  color: white;
  padding: 20px 40px;
  font-size: 1.3rem;
  border-radius: 50px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 15px;
  margin: 8px auto 0;
  box-shadow: 0 10px 30px rgba(102, 126, 234, 0.4);
  transition: all 0.3s ease;
}

.next-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 15px 40px rgba(102, 126, 234, 0.6);
}

.next-btn .arrow {
  font-size: 1.5rem;
  transition: transform 0.3s ease;
}

.next-btn:hover .arrow {
  transform: translateX(5px);
}

/* 完整视图样式 */
.full-content {
  padding: 60px 0;
  animation: slideInFromLeft 0.8s ease-out;
}

@keyframes slideInFromLeft {
  from {
    opacity: 0;
    transform: translateX(-100px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

/* 介绍区域 */
.intro-section {
  margin-bottom: 80px;
}

.intro-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 60px;
  align-items: center;
}

.intro-text h2 {
  font-size: 2.5rem;
  color: #2c3e50;
  margin-bottom: 30px;
}

.intro-description {
  font-size: 1.1rem;
  line-height: 1.8;
  color: #555;
  margin-bottom: 20px;
}

.features-list {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
  margin-top: 30px;
}

.feature-item {
  display: flex;
  align-items: center;
  padding: 15px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.feature-icon {
  font-size: 1.5rem;
  margin-right: 10px;
}

.intro-image {
  text-align: center;
}

.overview-image {
  width: 100%;
  max-width: 500px;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.2);
}

.image-caption {
  margin-top: 15px;
  color: #666;
  font-style: italic;
}

/* 地图区域 */
.map-section {
  margin-bottom: 80px;
}

.section-title {
  font-size: 2.5rem;
  color: #2c3e50;
  text-align: center;
  margin-bottom: 15px;
}

.section-subtitle {
  text-align: center;
  color: #666;
  font-size: 1.1rem;
  margin-bottom: 40px;
}

.interactive-map {
  position: relative;
  display: inline-block;
  width: 100%;
  margin: 0 0;
  background: white;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.2);
  overflow: hidden;
}

.map-image {
  width: 100%;
  height: auto;
  display: block;
}

.attraction-marker {
  position: absolute;
  transform: translate(-50%, -50%);
  cursor: pointer;
  z-index: 10;
}

.marker-dot {
  width: 20px;
  height: 20px;
  background: #e74c3c;
  border: 3px solid white;
  border-radius: 50%;
  box-shadow: 0 3px 10px rgba(0,0,0,0.3);
  transition: all 0.3s ease;
}

.marker-label {
  position: absolute;
  top: -40px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(0,0,0,0.8);
  color: white;
  padding: 5px 10px;
  border-radius: 4px;
  font-size: 0.9rem;
  white-space: nowrap;
  opacity: 0;
  transition: all 0.3s ease;
  pointer-events: none;
}

.attraction-marker:hover .marker-dot {
  transform: scale(1.3);
  background: #c0392b;
}

.attraction-marker:hover .marker-label {
  opacity: 1;
  top: -35px;
}

/* 特色展示区域 */
.highlights-section {
  background: white;
  border-radius: 20px;
  padding: 60px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.1);
}

.highlights-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 30px;
  margin-top: 40px;
}

.highlight-card {
  text-align: center;
  padding: 30px;
  border-radius: 12px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  transition: transform 0.3s ease;
}

.highlight-card:hover {
  transform: translateY(-5px);
}

.highlight-icon {
  font-size: 3rem;
  margin-bottom: 20px;
}

.highlight-card h3 {
  font-size: 1.5rem;
  margin-bottom: 15px;
}

.highlight-card p {
  line-height: 1.6;
  opacity: 0.9;
}

/* 弹窗样式 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 20px;
  backdrop-filter: blur(5px);
  animation: fadeIn 0.3s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.modal-content {
  background: white;
  border-radius: 16px;
  max-width: 950px;
  width: 100%;
  max-height: 90vh;
  overflow: hidden;
  box-shadow: 0 25px 80px rgba(0,0,0,0.4);
  animation: slideUp 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  transform-origin: center;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(50px) scale(0.95);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 30px 35px;
  border-bottom: 1px solid #e8e8e8;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  position: relative;
}

.modal-header::before {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: linear-gradient(90deg, #f39c12, #e67e22, #f39c12);
}

.modal-header h3 {
  margin: 0;
  font-size: 1.8rem;
  font-weight: 700;
  text-shadow: 1px 1px 3px rgba(0,0,0,0.3);
}

.header-controls {
  display: flex;
  align-items: center;
  gap: 18px;
}

.model-toggle-btn {
  background: rgba(255,255,255,0.15);
  border: 2px solid rgba(255,255,255,0.25);
  color: white;
  padding: 10px 20px;
  border-radius: 25px;
  cursor: pointer;
  font-size: 0.95rem;
  font-weight: 600;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.model-toggle-btn:hover,
.model-toggle-btn.active {
  background: rgba(255,255,255,0.25);
  border-color: rgba(255,255,255,0.4);
  transform: translateY(-1px);
  box-shadow: 0 4px 15px rgba(0,0,0,0.2);
}

.close-btn {
  background: rgba(255,255,255,0.1);
  border: 2px solid rgba(255,255,255,0.2);
  font-size: 1.8rem;
  color: white;
  cursor: pointer;
  padding: 8px;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
}

.close-btn:hover {
  background: rgba(231,76,60,0.8);
  border-color: rgba(231,76,60,0.8);
  transform: scale(1.1);
  box-shadow: 0 4px 15px rgba(231,76,60,0.4);
}

.modal-body {
  padding: 0;
  overflow-y: auto;
  max-height: calc(90vh - 100px);
}

/* 3D模型展示样式 */
.model-viewer-fullscreen {
  width: 100%;
  height: 70vh;
  min-height: 500px;
  position: relative;
}

/* 全屏3D模型模式的弹窗样式 */
.modal-content.fullscreen-model {
  max-width: 95vw;
  max-height: 95vh;
  width: 95vw;
  height: 95vh;
}

.modal-content.fullscreen-model .modal-body {
  padding: 0;
  height: calc(100% - 80px);
}

.modal-content.fullscreen-model .model-viewer-fullscreen {
  height: 100%;
  min-height: unset;
}

/* 图片轮播样式 */
.image-carousel {
  position: relative;
  margin-bottom: 30px;
  background: #f8f9fa;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 20px rgba(0,0,0,0.1);
}

.carousel-container {
  position: relative;
  height: 400px;
  overflow: hidden;
  background: linear-gradient(45deg, #f0f2f5, #ffffff);
}

.carousel-images {
  position: relative;
  width: 100%;
  height: 100%;
}

.carousel-image {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: center;
  transition: transform 0.6s cubic-bezier(0.4, 0, 0.2, 1);
  filter: brightness(1.05) contrast(1.1) saturate(1.1);
  border-radius: 8px;
}

.carousel-image:hover {
  filter: brightness(1.1) contrast(1.15) saturate(1.15);
}

.carousel-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background: linear-gradient(135deg, rgba(0,0,0,0.7), rgba(0,0,0,0.9));
  border: 2px solid rgba(255,255,255,0.2);
  color: white;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  cursor: pointer;
  font-size: 1.8rem;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 10;
  backdrop-filter: blur(10px);
  box-shadow: 0 4px 15px rgba(0,0,0,0.3);
}

.carousel-btn:hover {
  background: linear-gradient(135deg, rgba(231,76,60,0.9), rgba(192,57,43,0.9));
  transform: translateY(-50%) scale(1.1);
  box-shadow: 0 6px 20px rgba(231,76,60,0.4);
  border-color: rgba(255,255,255,0.4);
}

.prev-btn {
  left: 20px;
}

.next-btn {
  right: 20px;
}

.carousel-indicators {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 12px;
  background: rgba(0,0,0,0.6);
  padding: 10px 20px;
  border-radius: 25px;
  backdrop-filter: blur(10px);
}

.indicator {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  border: 2px solid rgba(255,255,255,0.5);
  background: transparent;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
}

.indicator:hover {
  border-color: rgba(255,255,255,0.8);
  transform: scale(1.1);
}

.indicator.active {
  background: linear-gradient(135deg, #f39c12, #e67e22);
  border-color: #fff;
  transform: scale(1.3);
  box-shadow: 0 0 10px rgba(243,156,18,0.6);
}

.indicator.active::after {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 6px;
  height: 6px;
  background: white;
  border-radius: 50%;
}

/* 景点信息 */
.attraction-info {
  padding: 35px;
  background: linear-gradient(135deg, #f8f9fa 0%, #ffffff 100%);
}

.attraction-info .info-section {
  margin-bottom: 30px;
  padding: 20px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 2px 15px rgba(0,0,0,0.08);
  border-left: 4px solid #667eea;
  transition: all 0.3s ease;
}

.attraction-info .info-section:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 20px rgba(0,0,0,0.12);
}

.attraction-info h4 {
  color: #2c3e50;
  font-size: 1.3rem;
  font-weight: 700;
  margin-bottom: 15px;
  border-left: none;
  padding-left: 0;
  display: flex;
  align-items: center;
  gap: 10px;
}

.attraction-info h4::before {
  content: '📍';
  font-size: 1.1rem;
}

.attraction-info .info-section:nth-child(2) h4::before {
  content: '📜';
}

.attraction-info .info-section:nth-child(3) h4::before {
  content: '✨';
}

.attraction-info p {
  line-height: 1.8;
  color: #555;
  font-size: 1rem;
}

.features-list {
  margin: 0;
  padding: 0;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 12px;
}

.features-list li {
  list-style: none;
  padding: 12px 16px;
  border-bottom: none;
  position: relative;
  padding-left: 35px;
  background: rgba(102, 126, 234, 0.05);
  border-radius: 8px;
  border-left: 3px solid #667eea;
  transition: all 0.3s ease;
}

.features-list li:hover {
  background: rgba(102, 126, 234, 0.1);
  transform: translateX(5px);
}

.features-list li:before {
  content: "✓";
  color: #27ae60;
  font-weight: bold;
  position: absolute;
  left: 12px;
  font-size: 1.1rem;
}

/* 底部样式 */
.site-footer {
  background: #2c3e50;
  color: white;
  text-align: center;
  padding: 40px 0;
  /* margin-top: 80px; */
  box-shadow: 0 -4px 20px rgba(0,0,0,0.1);
}

/* 响应式设计 - 平板端 */
@media (max-width: 1024px) {
  .intro-grid {
    gap: 40px;
  }
  
  .highlights-section {
    padding: 40px 30px;
  }
  
  .full-content {
    padding: 40px 0;
  }
}

/* 移动端适配 - 大屏手机 */
@media (max-width: 768px) {
  .site-header {
    padding: 20px 0;
  }

  .site-title {
    font-size: 2.2rem;
  }

  .site-subtitle {
    font-size: 0.9rem;
    margin-top: 8px;
  }

  .title-row {
    flex-direction: column;
    gap: 8px;
    margin-bottom: 0;
  }

  .back-btn {
    position: static;
    transform: none;
    margin: 15px auto 0;
    display: block;
    font-size: 0.9rem;
    padding: 10px 20px;
  }

  /* 初始视图优化 */
  .initial-overview-image {
    max-width: 90vw;
    height: auto;
  }

  .next-btn {
    padding: 15px 30px;
    font-size: 1.1rem;
    margin-top: 30px;
  }

  /* 完整视图布局调整 */
  .intro-grid {
    grid-template-columns: 1fr;
    gap: 30px;
    text-align: center;
  }

  .intro-grid .intro-image {
    order: -1;
  }

  .intro-text h2 {
    font-size: 2rem;
    margin-bottom: 20px;
  }

  .intro-description {
    font-size: 1rem;
    margin-bottom: 15px;
  }

  .features-list {
    grid-template-columns: 1fr;
    gap: 10px;
    margin-top: 20px;
  }

  .feature-item {
    padding: 12px 15px;
    text-align: left;
  }

  .feature-icon {
    font-size: 1.3rem;
    margin-right: 8px;
  }

  /* 地图区域优化 */
  .section-title {
    font-size: 2rem;
    margin-bottom: 10px;
  }

  .section-subtitle {
    font-size: 1rem;
    margin-bottom: 25px;
    padding: 0 15px;
  }

  .interactive-map {
    margin: 0 -10px;
    border-radius: 8px;
  }

  .marker-dot {
    width: 18px;
    height: 18px;
    border-width: 2px;
  }

  .marker-label {
    font-size: 0.8rem;
    padding: 4px 8px;
    top: -35px;
  }

  /* 特色展示区域 */
  .highlights-section {
    padding: 40px 20px;
    margin: 0 10px;
  }

  .highlights-grid {
    grid-template-columns: 1fr;
    gap: 20px;
    margin-top: 25px;
  }

  .highlight-card {
    padding: 25px 20px;
  }

  .highlight-icon {
    font-size: 2.5rem;
    margin-bottom: 15px;
  }

  .highlight-card h3 {
    font-size: 1.3rem;
    margin-bottom: 12px;
  }

  .highlight-card p {
    font-size: 0.95rem;
    line-height: 1.5;
  }

  /* 弹窗优化 */
  .modal-overlay {
    padding: 10px;
    align-items: flex-start;
    overflow-y: auto;
  }

  .modal-content {
    max-width: 100vw;
    margin: 20px 0;
    max-height: calc(100vh - 40px);
  }

  .modal-header {
    padding: 20px;
    flex-wrap: wrap;
    gap: 10px;
  }

  .modal-header h3 {
    font-size: 1.4rem;
    flex: 1;
    min-width: 200px;
  }

  .header-controls {
    gap: 10px;
    flex-shrink: 0;
  }

  .model-toggle-btn {
    padding: 8px 15px;
    font-size: 0.85rem;
    border-radius: 20px;
  }

  .close-btn {
    width: 35px;
    height: 35px;
    font-size: 1.5rem;
  }

  /* 轮播图优化 */
  .carousel-container {
    height: 250px;
  }

  .carousel-btn {
    width: 40px;
    height: 40px;
    font-size: 1.4rem;
  }

  .prev-btn {
    left: 10px;
  }

  .next-btn {
    right: 10px;
  }

  .carousel-indicators {
    bottom: 15px;
    padding: 8px 15px;
  }

  .indicator {
    width: 10px;
    height: 10px;
  }

  /* 景点信息优化 */
  .attraction-info {
    padding: 20px 15px;
  }

  .attraction-info .info-section {
    padding: 15px;
    margin-bottom: 20px;
  }

  .attraction-info h4 {
    font-size: 1.2rem;
    margin-bottom: 10px;
  }

  .attraction-info p {
    font-size: 0.95rem;
    line-height: 1.6;
  }

  .features-list {
    grid-template-columns: 1fr;
    gap: 8px;
  }

  .features-list li {
    padding: 10px 14px;
    padding-left: 30px;
    font-size: 0.9rem;
  }

  .features-list li:before {
    left: 10px;
    font-size: 1rem;
  }

  /* 3D模型移动端优化 */
  .model-viewer-fullscreen {
    height: 60vh;
    min-height: 300px;
  }

  .modal-content.fullscreen-model {
    margin: 0;
    max-height: 100vh;
    border-radius: 0;
  }
}

/* 小屏手机适配 */
@media (max-width: 480px) {
  .container {
    padding: 0 15px;
  }

  .site-header {
    padding: 15px 0;
  }

  .site-title {
    font-size: 1.8rem;
  }

  .site-subtitle {
    font-size: 0.8rem;
  }

  .back-btn {
    font-size: 0.85rem;
    padding: 8px 16px;
  }

  /* 初始视图 */
  .initial-view {
    min-height: 70vh;
    padding: 15px 0;
  }

  .initial-overview-image {
    max-width: 95vw;
  }

  .next-btn {
    padding: 12px 25px;
    font-size: 1rem;
    margin-top: 25px;
  }

  .next-btn .arrow {
    font-size: 1.3rem;
  }

  /* 文字内容优化 */
  .intro-text h2 {
    font-size: 1.7rem;
    margin-bottom: 15px;
  }

  .intro-description {
    font-size: 0.9rem;
    line-height: 1.6;
  }

  .section-title {
    font-size: 1.7rem;
  }

  .section-subtitle {
    font-size: 0.9rem;
    padding: 0 10px;
  }

  /* 地图标记优化 */
  .marker-dot {
    width: 16px;
    height: 16px;
    border-width: 2px;
  }

  .marker-label {
    font-size: 0.75rem;
    padding: 3px 6px;
    top: -30px;
  }

  /* 特色卡片 */
  .highlights-section {
    padding: 30px 15px;
    margin: 0 5px;
  }

  .highlight-card {
    padding: 20px 15px;
  }

  .highlight-icon {
    font-size: 2.2rem;
  }

  .highlight-card h3 {
    font-size: 1.2rem;
  }

  .highlight-card p {
    font-size: 0.9rem;
  }

  /* 弹窗进一步优化 */
  .modal-overlay {
    padding: 5px;
  }

  .modal-content {
    margin: 10px 0;
    border-radius: 12px;
  }

  .modal-header {
    padding: 15px;
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
  }

  .modal-header h3 {
    font-size: 1.2rem;
    width: 100%;
  }

  .header-controls {
    align-self: flex-end;
    gap: 8px;
  }

  .model-toggle-btn {
    padding: 6px 12px;
    font-size: 0.8rem;
  }

  .close-btn {
    width: 30px;
    height: 30px;
    font-size: 1.3rem;
  }

  /* 轮播图小屏优化 */
  .carousel-container {
    height: 200px;
  }

  .carousel-btn {
    width: 35px;
    height: 35px;
    font-size: 1.2rem;
  }

  .prev-btn, .next-btn {
    top: 45%;
  }

  .prev-btn {
    left: 8px;
  }

  .next-btn {
    right: 8px;
  }

  .carousel-indicators {
    bottom: 10px;
    padding: 6px 12px;
    gap: 8px;
  }

  .indicator {
    width: 8px;
    height: 8px;
    border-width: 1px;
  }

  .indicator.active::after {
    width: 4px;
    height: 4px;
  }

  /* 景点信息小屏优化 */
  .attraction-info {
    padding: 15px 10px;
  }

  .attraction-info .info-section {
    padding: 12px;
    margin-bottom: 15px;
    border-left-width: 3px;
  }

  .attraction-info h4 {
    font-size: 1.1rem;
    margin-bottom: 8px;
  }

  .attraction-info h4::before {
    font-size: 1rem;
  }

  .attraction-info p {
    font-size: 0.9rem;
    line-height: 1.5;
  }

  .features-list li {
    padding: 8px 12px;
    padding-left: 28px;
    font-size: 0.85rem;
  }

  .features-list li:before {
    left: 8px;
    font-size: 0.9rem;
  }

  /* 3D模型小屏优化 */
  .model-viewer-fullscreen {
    height: 50vh;
    min-height: 250px;
  }

  .modal-content.fullscreen-model {
    width: 100vw;
    height: 100vh;
    max-width: 100vw;
    max-height: 100vh;
  }

  .modal-content.fullscreen-model .modal-header {
    padding: 10px 15px;
    flex-direction: row;
    align-items: center;
  }

  .modal-content.fullscreen-model .modal-header h3 {
    font-size: 1.1rem;
  }

  /* 底部样式 */
  .site-footer {
    padding: 20px 0;
    margin-top: 40px;
  }

  .site-footer p {
    font-size: 0.9rem;
    line-height: 1.4;
  }
}

/* 超小屏设备适配 */
@media (max-width: 360px) {
  .site-title {
    font-size: 1.6rem;
  }

  .site-subtitle {
    font-size: 0.75rem;
  }

  .next-btn {
    padding: 10px 20px;
    font-size: 0.95rem;
  }

  .intro-text h2 {
    font-size: 1.5rem;
  }

  .section-title {
    font-size: 1.5rem;
  }

  .carousel-container {
    height: 180px;
  }

  .model-viewer-fullscreen {
    height: 45vh;
    min-height: 200px;
  }

  .modal-header h3 {
    font-size: 1.1rem;
  }

  .highlights-section {
    padding: 25px 10px;
  }

  .highlight-card {
    padding: 15px 12px;
  }
}

/* 横屏模式优化 */
@media (max-height: 500px) and (orientation: landscape) {
  .initial-view {
    min-height: 60vh;
  }

  .carousel-container {
    height: 200px;
  }

  .model-viewer-fullscreen {
    height: 70vh;
    min-height: 200px;
  }

  .modal-content {
    max-height: 95vh;
  }
}
</style> 