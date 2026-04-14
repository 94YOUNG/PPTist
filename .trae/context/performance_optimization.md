# 性能优化指南

## 性能问题分析

### 1. 性能瓶颈识别

#### 使用 Chrome DevTools
```
1. Performance 面板：记录性能数据
   - 打开 DevTools (F12)
   - 切换到 Performance 标签
   - 点击录制按钮
   - 执行操作
   - 停止录制并分析

2. Memory 面板：分析内存使用
   - 查看内存占用
   - 分析内存泄漏
   - 对比堆快照

3. Rendering 面板：分析渲染性能
   - 启用 FPS meter
   - 查看 Paint flashing
   - 分析 Layout shifts
```

#### 关键性能指标
```typescript
// 使用 Performance API 监控
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(`${entry.name}: ${entry.duration}ms`)
  }
})

observer.observe({ entryTypes: ['measure', 'navigation'] })

// 测量关键操作
performance.mark('element-drag-start')
// ... 拖拽操作
performance.mark('element-drag-end')
performance.measure('element-drag', 'element-drag-start', 'element-drag-end')
```

## 渲染性能优化

### 1. 减少 Re-render

#### 使用 v-once
```vue
<!-- ✅ 推荐：静态内容使用 v-once -->
<template>
  <div v-once class="static-content">
    <h1>{{ title }}</h1>
    <p>这段内容不会改变</p>
  </div>
</template>
```

#### 使用 v-memo
```vue
<!-- ✅ 推荐：条件渲染使用 v-memo -->
<template>
  <div
    v-for="element in elements"
    :key="element.id"
    v-memo="[element.id, element.selected]"
  >
    <!-- 只有 id 或 selected 变化时才重新渲染 -->
  </div>
</template>
```

#### 合理使用 key
```vue
<!-- ✅ 推荐：使用唯一且稳定的 key -->
<template>
  <div
    v-for="element in elements"
    :key="element.id"
  >
    {{ element.name }}
  </div>
</template>

<!-- ❌ 避免：使用索引作为 key -->
<template>
  <div
    v-for="(element, index) in elements"
    :key="index"
  >
    {{ element.name }}
  </div>
</template>
```

### 2. 虚拟滚动

#### 实现虚拟列表
```vue
<template>
  <div
    class="virtual-list"
    @scroll="handleScroll"
  >
    <div
      class="virtual-list-content"
      :style="{ height: `${totalHeight}px` }"
    >
      <div
        v-for="item in visibleItems"
        :key="item.id"
        class="virtual-list-item"
        :style="{ transform: `translateY(${item.offset}px)` }"
      >
        <slot :item="item" />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
const itemHeight = 50
const bufferSize = 5

const scrollTop = ref(0)
const containerHeight = ref(600)

const visibleCount = computed(() => {
  return Math.ceil(containerHeight.value / itemHeight) + bufferSize * 2
})

const startIndex = computed(() => {
  return Math.max(0, Math.floor(scrollTop.value / itemHeight) - bufferSize)
})

const endIndex = computed(() => {
  return Math.min(items.value.length, startIndex.value + visibleCount.value)
})

const visibleItems = computed(() => {
  return items.value.slice(startIndex.value, endIndex.value).map((item, i) => ({
    ...item,
    offset: (startIndex.value + i) * itemHeight,
  }))
})

const totalHeight = computed(() => items.value.length * itemHeight)

const handleScroll = throttle((e: Event) => {
  scrollTop.value = (e.target as HTMLElement).scrollTop
}, 16)
</script>
```

### 3. 图片优化

#### 懒加载
```vue
<template>
  <img
    v-lazy="imageUrl"
    :alt="alt"
    @load="handleLoad"
  />
</template>

<script setup lang="ts">
const vLazy = {
  mounted(el: HTMLImageElement, binding: any) {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          el.src = binding.value
          observer.unobserve(el)
        }
      })
    })
    
    observer.observe(el)
  }
}
</script>
```

#### 图片压缩
```typescript
// ✅ 推荐：上传前压缩图片
async function compressImage(file: File): Promise<Blob> {
  const options = {
    maxSizeMB: 1,
    maxWidthOrHeight: 1920,
    useWebWorker: true,
  }
  
  return await imageCompression(file, options)
}
```

## 计算性能优化

### 1. 计算属性缓存

#### 使用 computed
```typescript
// ✅ 推荐：使用 computed 缓存计算结果
const elementStyle = computed(() => {
  return {
    left: `${props.element.left}px`,
    top: `${props.element.top}px`,
    width: `${props.element.width}px`,
    height: `${props.element.height}px`,
    transform: `rotate(${props.element.rotate}deg)`,
  }
})

// ❌ 避免：在模板中直接计算
// <div :style="`left: ${element.left}px; top: ${element.top}px`">
```

### 2. 防抖和节流

#### 防抖 (Debounce)
```typescript
// ✅ 推荐：搜索输入使用防抖
import { debounce } from 'lodash-es'

const handleSearch = debounce((keyword: string) => {
  // 执行搜索
  searchElements(keyword)
}, 300)
```

#### 节流 (Throttle)
```typescript
// ✅ 推荐：高频事件使用节流
import { throttle } from 'lodash-es'

// 拖拽事件
const handleMouseMove = throttle((e: MouseEvent) => {
  updateElementPosition(e)
}, 16) // 60fps

// 滚动事件
const handleScroll = throttle(() => {
  updateVisibleElements()
}, 100)
```

### 3. Web Worker

#### 复杂计算放入 Worker
```typescript
// worker.ts
self.onmessage = (e) => {
  const { type, data } = e.data
  
  if (type === 'calculate-layout') {
    const result = calculateLayout(data)
    self.postMessage({ type: 'layout-result', result })
  }
}

// main.ts
const worker = new Worker('./worker.ts', { type: 'module' })

worker.onmessage = (e) => {
  const { type, result } = e.data
  
  if (type === 'layout-result') {
    // 使用计算结果
    applyLayout(result)
  }
}

// 发送计算任务
function calculateLayoutAsync(elements: PPTElement[]) {
  worker.postMessage({
    type: 'calculate-layout',
    data: elements,
  })
}
```

## 内存优化

### 1. 避免内存泄漏

#### 清理定时器
```typescript
// ✅ 推荐：及时清理定时器
export default function useAutoSave() {
  let timer: number | null = null
  
  const startAutoSave = () => {
    timer = setInterval(() => {
      savePresentation()
    }, 30000)
  }
  
  onUnmounted(() => {
    if (timer) {
      clearInterval(timer)
      timer = null
    }
  })
  
  return { startAutoSave }
}
```

#### 清理事件监听
```typescript
// ✅ 推荐：及时清理事件监听
export default function useKeyboard() {
  const handleKeyDown = (e: KeyboardEvent) => {
    // 处理键盘事件
  }
  
  onMounted(() => {
    window.addEventListener('keydown', handleKeyDown)
  })
  
  onUnmounted(() => {
    window.removeEventListener('keydown', handleKeyDown)
  })
}
```

#### 清理对象引用
```typescript
// ✅ 推荐：清理大对象引用
export default function useImageLoader() {
  const images = ref<Map<string, HTMLImageElement>>(new Map())
  
  const loadImage = (id: string, src: string) => {
    const img = new Image()
    img.src = src
    images.value.set(id, img)
  }
  
  const clearImage = (id: string) => {
    const img = images.value.get(id)
    if (img) {
      img.src = '' // 清除图片引用
      images.value.delete(id)
    }
  }
  
  onUnmounted(() => {
    images.value.forEach((img, id) => {
      clearImage(id)
    })
    images.value.clear()
  })
}
```

### 2. 减少内存占用

#### 使用 shallowRef
```typescript
// ✅ 推荐：大数据使用 shallowRef
const largeData = shallowRef<BigData>({})

// 只有整个对象替换才会触发更新
largeData.value = newData
```

#### 避免深拷贝
```typescript
// ✅ 推荐：使用 structuredClone
const clonedData = structuredClone(originalData)

// ✅ 推荐：使用 lodash 的 cloneDeep
import { cloneDeep } from 'lodash-es'
const clonedData = cloneDeep(originalData)

// ❌ 避免：JSON.parse(JSON.stringify())
const clonedData = JSON.parse(JSON.stringify(originalData)) // 性能差
```

## 网络性能优化

### 1. 请求优化

#### 请求合并
```typescript
// ✅ 推荐：合并多个请求
async function loadSlideData(slideIds: string[]) {
  const promises = slideIds.map(id => fetchSlideData(id))
  const results = await Promise.all(promises)
  return results
}
```

#### 请求缓存
```typescript
// ✅ 推荐：缓存请求结果
const cache = new Map<string, Promise<any>>()

async function fetchWithCache<T>(url: string): Promise<T> {
  if (cache.has(url)) {
    return cache.get(url)!
  }
  
  const promise = fetch(url).then(res => res.json())
  cache.set(url, promise)
  
  return promise
}
```

### 2. 资源加载

#### 预加载
```typescript
// ✅ 推荐：预加载关键资源
function preloadImage(url: string): Promise<HTMLImageElement> {
  return new Promise((resolve, reject) => {
    const img = new Image()
    img.onload = () => resolve(img)
    img.onerror = reject
    img.src = url
  })
}

// 预加载下一张幻灯片的图片
async function preloadNextSlide() {
  const nextSlide = slides.value[slideIndex.value + 1]
  if (nextSlide) {
    const images = nextSlide.elements
      .filter(el => el.type === 'image')
      .map(el => preloadImage(el.src))
    
    await Promise.all(images)
  }
}
```

#### 按需加载
```typescript
// ✅ 推荐：动态导入组件
const HeavyComponent = defineAsyncComponent(() =>
  import('./HeavyComponent.vue')
)

// ✅ 推荐：动态导入模块
async function loadChartLibrary() {
  const { default: echarts } = await import('echarts')
  return echarts
}
```

## 代码分割

### 1. 路由懒加载
```typescript
// ✅ 推荐：路由懒加载
const routes = [
  {
    path: '/editor',
    component: () => import('@/views/Editor/index.vue'),
  },
  {
    path: '/screen',
    component: () => import('@/views/Screen/index.vue'),
  },
]
```

### 2. 组件懒加载
```vue
<template>
  <div>
    <Suspense>
      <template #default>
        <HeavyComponent />
      </template>
      <template #fallback>
        <Loading />
      </template>
    </Suspense>
  </div>
</template>

<script setup lang="ts">
const HeavyComponent = defineAsyncComponent(() =>
  import('./HeavyComponent.vue')
)
</script>
```

## 性能监控

### 1. 性能指标收集

#### Web Vitals
```typescript
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals'

getCLS(console.log)
getFID(console.log)
getFCP(console.log)
getLCP(console.log)
getTTFB(console.log)
```

#### 自定义性能监控
```typescript
class PerformanceMonitor {
  private metrics = new Map<string, number[]>()
  
  startMeasure(name: string) {
    performance.mark(`${name}-start`)
  }
  
  endMeasure(name: string) {
    performance.mark(`${name}-end`)
    performance.measure(name, `${name}-start`, `${name}-end`)
    
    const measure = performance.getEntriesByName(name)[0]
    const duration = measure.duration
    
    if (!this.metrics.has(name)) {
      this.metrics.set(name, [])
    }
    this.metrics.get(name)!.push(duration)
    
    // 清理
    performance.clearMarks(`${name}-start`)
    performance.clearMarks(`${name}-end`)
    performance.clearMeasures(name)
    
    return duration
  }
  
  getAverageTime(name: string): number {
    const times = this.metrics.get(name)
    if (!times || times.length === 0) return 0
    
    return times.reduce((a, b) => a + b, 0) / times.length
  }
}

const monitor = new PerformanceMonitor()

// 使用
monitor.startMeasure('element-drag')
// ... 拖拽操作
const duration = monitor.endMeasure('element-drag')
console.log(`Drag took ${duration}ms`)
```

### 2. 性能报告

#### 上报性能数据
```typescript
async function reportPerformance(data: any) {
  if (navigator.sendBeacon) {
    const blob = new Blob([JSON.stringify(data)], { type: 'application/json' })
    navigator.sendBeacon('/api/performance', blob)
  } else {
    await fetch('/api/performance', {
      method: 'POST',
      body: JSON.stringify(data),
    })
  }
}

// 页面卸载时上报
window.addEventListener('beforeunload', () => {
  const performanceData = {
    url: window.location.href,
    timestamp: Date.now(),
    metrics: {
      // 收集的性能指标
    },
  }
  
  reportPerformance(performanceData)
})
```

## 优化检查清单

### 渲染性能
- [ ] 使用虚拟滚动处理大列表
- [ ] 合理使用 v-if 和 v-show
- [ ] 使用 computed 缓存计算结果
- [ ] 避免在模板中使用复杂表达式
- [ ] 使用 v-once 和 v-memo 优化静态内容

### 计算性能
- [ ] 高频事件使用防抖和节流
- [ ] 复杂计算使用 Web Worker
- [ ] 避免在循环中进行复杂计算
- [ ] 使用高效的算法和数据结构

### 内存优化
- [ ] 及时清理定时器和事件监听
- [ ] 清理大对象引用
- [ ] 使用 shallowRef 处理大数据
- [ ] 避免内存泄漏

### 网络优化
- [ ] 合并请求
- [ ] 使用缓存
- [ ] 预加载关键资源
- [ ] 按需加载非关键资源

### 代码优化
- [ ] 代码分割和懒加载
- [ ] 减少包体积
- [ ] Tree shaking
- [ ] 压缩代码

### 监控
- [ ] 收集 Web Vitals
- [ ] 监控关键操作性能
- [ ] 上报性能数据
- [ ] 设置性能预算

## 总结

性能优化是一个持续的过程，需要：
1. **测量**: 使用工具测量性能
2. **分析**: 找出性能瓶颈
3. **优化**: 应用优化策略
4. **验证**: 验证优化效果
5. **监控**: 持续监控性能

记住：过早优化是万恶之源。先保证功能正确，再进行性能优化。
