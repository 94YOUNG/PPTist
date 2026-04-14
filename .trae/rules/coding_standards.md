# 编码规范

## TypeScript 规范

### 类型定义
```typescript
// ✅ 推荐：使用 interface 定义对象类型
interface PPTElement {
  id: string
  left: number
  top: number
  width: number
  height: number
  rotate: number
}

// ✅ 推荐：使用 type 定义联合类型、交叉类型
type ElementTypes = 'text' | 'image' | 'shape' | 'line'
type PPTTextElement = PPTBaseElement & {
  type: 'text'
  content: string
}

// ✅ 推荐：使用 const enum 定义常量
export const enum ElementTypes {
  TEXT = 'text',
  IMAGE = 'image',
  SHAPE = 'shape',
}

// ❌ 避免：使用 any
const data: any = {} // 不推荐

// ✅ 推荐：使用 unknown 或具体类型
const data: unknown = {}
const data: PPTElement = {} // 推荐
```

### 泛型使用
```typescript
// ✅ 推荐：泛型约束
function getElementById<T extends PPTElement>(
  id: string,
  elements: T[]
): T | undefined {
  return elements.find(el => el.id === id)
}

// ✅ 推荐：泛型默认值
interface StoreOptions<T = any> {
  state: () => T
  actions: Record<string, (this: T, ...args: any[]) => any>
}
```

### 类型守卫
```typescript
// ✅ 推荐：使用类型守卫
function isTextElement(element: PPTElement): element is PPTTextElement {
  return element.type === 'text'
}

// 使用
if (isTextElement(element)) {
  console.log(element.content) // 类型安全
}
```

## Vue 组件规范

### 组件结构
```vue
<!-- ✅ 推荐：使用 script setup -->
<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import type { PPTElement } from '@/types/slides'

// Props
interface Props {
  element: PPTElement
  scale: number
}
const props = defineProps<Props>()

// Emits
interface Emits {
  (e: 'update', value: PPTElement): void
  (e: 'delete', id: string): void
}
const emit = defineEmits<Emits>()

// 响应式状态
const isEditing = ref(false)
const elementStyle = computed(() => ({
  left: `${props.element.left}px`,
  top: `${props.element.top}px`,
}))

// 方法
const handleClick = () => {
  emit('update', props.element)
}

// 生命周期
onMounted(() => {
  console.log('Component mounted')
})
</script>

<template>
  <div 
    class="element-wrapper"
    :style="elementStyle"
    @click="handleClick"
  >
    <slot />
  </div>
</template>

<style scoped lang="scss">
.element-wrapper {
  position: absolute;
  cursor: move;
}
</style>
```

### Props 验证
```typescript
// ✅ 推荐：使用 TypeScript 定义 props
interface Props {
  element: PPTElement
  scale?: number
  disabled?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  scale: 1,
  disabled: false,
})
```

## Hooks 规范

### Hook 设计
```typescript
// ✅ 推荐：Hook 返回对象
export default function useElementDrag() {
  const mainStore = useMainStore()
  const slidesStore = useSlidesStore()
  
  // 内部状态
  const isDragging = ref(false)
  const dragOffset = ref({ x: 0, y: 0 })
  
  // 内部方法
  const handleMouseDown = (e: MouseEvent, element: PPTElement) => {
    isDragging.value = true
    // ...
  }
  
  const handleMouseMove = (e: MouseEvent) => {
    if (!isDragging.value) return
    // ...
  }
  
  const handleMouseUp = () => {
    isDragging.value = false
    // ...
  }
  
  // 公开方法
  const startDrag = (element: PPTElement) => {
    // ...
  }
  
  // 返回公开的 API
  return {
    isDragging: readonly(isDragging),
    startDrag,
    handleMouseDown,
    handleMouseMove,
    handleMouseUp,
  }
}
```

### Hook 组合
```typescript
// ✅ 推荐：组合多个 hooks
export default function useElement() {
  const drag = useElementDrag()
  const resize = useElementResize()
  const rotate = useElementRotate()
  
  const selectedElement = computed(() => {
    // ...
  })
  
  return {
    selectedElement,
    ...drag,
    ...resize,
    ...rotate,
  }
}
```

## Store 规范

### Store 定义
```typescript
// ✅ 推荐：使用 Composition API 风格
export const useMainStore = defineStore('main', () => {
  // State
  const activeElementIdList = ref<string[]>([])
  const canvasScale = ref(1)
  
  // Getters
  const activeElement = computed(() => {
    const slidesStore = useSlidesStore()
    return slidesStore.currentSlide?.elements.find(
      el => activeElementIdList.value.includes(el.id)
    )
  })
  
  // Actions
  const setActiveElement = (id: string) => {
    activeElementIdList.value = [id]
  }
  
  const clearSelection = () => {
    activeElementIdList.value = []
  }
  
  return {
    // State
    activeElementIdList,
    canvasScale,
    // Getters
    activeElement,
    // Actions
    setActiveElement,
    clearSelection,
  }
})
```

## 样式规范

### SCSS 组织
```scss
// ✅ 推荐：使用变量和 mixin
@import '@/assets/styles/variable.scss';
@import '@/assets/styles/mixin.scss';

.element-wrapper {
  position: absolute;
  cursor: move;
  
  // 状态修饰符
  &.selected {
    outline: 2px solid $primary-color;
  }
  
  &.locked {
    opacity: 0.5;
    cursor: not-allowed;
  }
  
  // 子元素
  .element-content {
    @include flex-center;
    width: 100%;
    height: 100%;
  }
  
  // 响应式
  @include respond-to('mobile') {
    transform: scale(0.8);
  }
}
```

### CSS 变量
```scss
// ✅ 推荐：使用 CSS 变量实现主题
:root {
  --primary-color: #1890ff;
  --border-radius: 4px;
  --transition-duration: 0.3s;
}

.element {
  background: var(--primary-color);
  border-radius: var(--border-radius);
  transition: all var(--transition-duration);
}
```

## 工具函数规范

### 函数设计
```typescript
// ✅ 推荐：纯函数，有明确类型
export function getRectRotatedRange(
  element: RotatedElementData
): { xRange: [number, number]; yRange: [number, number] } {
  const { left, top, width, height, rotate = 0 } = element
  
  // 计算逻辑...
  
  return {
    xRange: [minX, maxX],
    yRange: [minY, maxY],
  }
}

// ✅ 推荐：带错误处理
export function parseJSON<T>(json: string): T | null {
  try {
    return JSON.parse(json) as T
  } catch (error) {
    console.error('Failed to parse JSON:', error)
    return null
  }
}
```

### 工具函数分类
```typescript
// ✅ 推荐：按功能分类导出
// utils/element.ts
export const getRectRotatedRange = () => { /* ... */ }
export const getElementRange = () => { /* ... */ }
export const getLineElementPath = () => { /* ... */ }

// utils/canvas.ts
export const calculateCanvasSize = () => { /* ... */ }
export const getCanvasPoint = () => { /* ... */ }
```

## 性能优化规范

### 响应式优化
```typescript
// ✅ 推荐：大数据使用 shallowRef
const largeData = shallowRef<BigData>({})

// ✅ 推荐：避免不必要的响应式
const staticConfig = { /* ... */ } // 不使用 ref

// ✅ 推荐：使用 computed 缓存
const expensiveValue = computed(() => {
  // 复杂计算
  return heavyCalculation(props.data)
})
```

### 事件处理优化
```typescript
// ✅ 推荐：使用防抖/节流
import { debounce, throttle } from 'lodash-es'

const handleResize = debounce(() => {
  // 处理 resize
}, 300)

const handleScroll = throttle(() => {
  // 处理 scroll
}, 100)
```

### 列表渲染优化
```vue
<!-- ✅ 推荐：使用 key 和虚拟列表 -->
<template>
  <div 
    v-for="element in elements" 
    :key="element.id"
  >
    <!-- ... -->
  </div>
</template>
```

## 错误处理规范

### 异步错误处理
```typescript
// ✅ 推荐：完整的错误处理
async function loadImage(src: string): Promise<HTMLImageElement> {
  try {
    const img = new Image()
    img.src = src
    await new Promise((resolve, reject) => {
      img.onload = resolve
      img.onerror = reject
    })
    return img
  } catch (error) {
    console.error('Failed to load image:', src, error)
    throw new Error(`图片加载失败: ${src}`)
  }
}
```

### 用户友好提示
```typescript
// ✅ 推荐：使用统一的提示组件
import { Message } from '@/components/Message'

async function savePresentation() {
  try {
    await saveToServer(data)
    Message.success('保存成功')
  } catch (error) {
    Message.error('保存失败，请重试')
    console.error(error)
  }
}
```

## 注释规范

### 函数注释
```typescript
/**
 * 计算元素在画布中的矩形范围旋转后的新位置范围
 * @param element 元素的位置大小和旋转角度信息
 * @returns 返回旋转后的x和y轴范围
 */
export function getRectRotatedRange(element: RotatedElementData) {
  // ...
}
```

### 复杂逻辑注释
```typescript
// ✅ 推荐：解释复杂逻辑
// 计算旋转后的边界范围
// 使用三角函数计算四个角的新位置
const radius = Math.sqrt(Math.pow(width, 2) + Math.pow(height, 2)) / 2
const auxiliaryAngle = Math.atan(height / width) * 180 / Math.PI
```

## 测试规范

### 单元测试
```typescript
// ✅ 推荐：测试工具函数
describe('getRectRotatedRange', () => {
  it('should return correct range for 0 degree rotation', () => {
    const element = {
      left: 100,
      top: 100,
      width: 200,
      height: 100,
      rotate: 0,
    }
    
    const result = getRectRotatedRange(element)
    
    expect(result.xRange).toEqual([100, 300])
    expect(result.yRange).toEqual([100, 200])
  })
})
```
