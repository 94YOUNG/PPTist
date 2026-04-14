# 最佳实践

## 组件开发最佳实践

### 1. 组件设计原则

#### 单一职责原则
```vue
<!-- ✅ 推荐：一个组件只负责一件事 -->
<template>
  <div class="color-picker">
    <Saturation :color="color" @change="handleSaturationChange" />
    <Hue :hue="hue" @change="handleHueChange" />
    <Alpha :alpha="alpha" @change="handleAlphaChange" />
  </div>
</template>

<!-- ❌ 避免：一个组件做太多事情 -->
<template>
  <div class="editor">
    <!-- 包含了颜色选择、字体选择、对齐方式等太多功能 -->
  </div>
</template>
```

#### 组件通信
```vue
<!-- ✅ 推荐：使用 props 和 emits -->
<script setup lang="ts">
interface Props {
  element: PPTElement
}
const props = defineProps<Props>()

interface Emits {
  (e: 'update', element: PPTElement): void
  (e: 'delete', id: string): void
}
const emit = defineEmits<Emits>()

const handleUpdate = () => {
  emit('update', props.element)
}
</script>

<!-- ❌ 避免：直接修改 props -->
<script setup lang="ts">
const props = defineProps<{ element: PPTElement }>()

const handleClick = () => {
  props.element.left = 100 // ❌ 直接修改 props
}
</script>
```

### 2. 性能优化

#### 合理使用 v-if 和 v-show
```vue
<!-- ✅ 推荐：频繁切换使用 v-show -->
<div v-show="isVisible" class="tooltip">
  {{ content }}
</div>

<!-- ✅ 推荐：条件渲染使用 v-if -->
<Modal v-if="showModal" @close="showModal = false">
  <template #content>
    <!-- 复杂内容 -->
  </template>
</Modal>
```

#### 列表渲染优化
```vue
<!-- ✅ 推荐：使用唯一 key -->
<template>
  <div
    v-for="element in elements"
    :key="element.id"
    class="element"
  >
    {{ element.name }}
  </div>
</template>

<!-- ❌ 避免：使用索引作为 key -->
<template>
  <div
    v-for="(element, index) in elements"
    :key="index"
    class="element"
  >
    {{ element.name }}
  </div>
</template>
```

#### 计算属性缓存
```vue
<script setup lang="ts">
import { computed } from 'vue'

// ✅ 推荐：使用 computed 缓存计算结果
const elementStyle = computed(() => ({
  left: `${props.element.left}px`,
  top: `${props.element.top}px`,
  width: `${props.element.width}px`,
  height: `${props.element.height}px`,
  transform: `rotate(${props.element.rotate}deg)`,
}))

// ❌ 避免：在模板中直接计算
// <div :style="`left: ${element.left}px; top: ${element.top}px`">
</script>
```

### 3. 响应式数据处理

#### 避免不必要的响应式
```typescript
// ✅ 推荐：静态配置不需要响应式
const STATIC_CONFIG = {
  MAX_ELEMENTS: 100,
  MIN_SCALE: 0.1,
  MAX_SCALE: 5,
}

// ✅ 推荐：大数据使用 shallowRef
const largeData = shallowRef<BigData>({})

// ❌ 避免：所有数据都用 ref
const config = ref({
  MAX_ELEMENTS: 100,
  MIN_SCALE: 0.1,
})
```

#### 正确使用 toRefs
```typescript
// ✅ 推荐：解构 props 时使用 toRefs
import { toRefs } from 'vue'

const props = defineProps<{
  element: PPTElement
  scale: number
}>()

const { element, scale } = toRefs(props)

// 现在可以安全地使用 element.value 和 scale.value
```

## Hooks 开发最佳实践

### 1. Hook 设计模式

#### 返回值设计
```typescript
// ✅ 推荐：返回对象，包含状态和方法
export default function useElementDrag() {
  const isDragging = ref(false)
  const dragOffset = ref({ x: 0, y: 0 })
  
  const startDrag = (element: PPTElement) => {
    isDragging.value = true
    // ...
  }
  
  const endDrag = () => {
    isDragging.value = false
    // ...
  }
  
  return {
    isDragging: readonly(isDragging),
    dragOffset: readonly(dragOffset),
    startDrag,
    endDrag,
  }
}

// ❌ 避免：返回数组，不够语义化
export default function useElementDrag() {
  return [isDragging, startDrag, endDrag]
}
```

#### Hook 组合
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

### 2. 副作用管理

#### 清理副作用
```typescript
// ✅ 推荐：在 onUnmounted 中清理
export default function useEventListener(
  target: EventTarget,
  event: string,
  callback: EventListener
) {
  onMounted(() => {
    target.addEventListener(event, callback)
  })
  
  onUnmounted(() => {
    target.removeEventListener(event, callback)
  })
}
```

#### watch 和 watchEffect
```typescript
// ✅ 推荐：明确依赖的 watch
watch(
  () => props.element.left,
  (newLeft, oldLeft) => {
    console.log(`Left changed from ${oldLeft} to ${newLeft}`)
  },
  { immediate: true }
)

// ✅ 推荐：自动追踪依赖的 watchEffect
watchEffect(() => {
  console.log(`Element position: ${props.element.left}, ${props.element.top}`)
})

// ❌ 避免：深度监听大对象
watch(
  () => props.element,
  (newElement) => {
    // 性能问题
  },
  { deep: true }
)
```

## Store 开发最佳实践

### 1. Store 组织

#### 模块化设计
```typescript
// ✅ 推荐：按功能划分 store
// store/main.ts - 全局状态
export const useMainStore = defineStore('main', () => {
  const activeElementIdList = ref<string[]>([])
  const canvasScale = ref(1)
  
  return {
    activeElementIdList,
    canvasScale,
  }
})

// store/slides.ts - 幻灯片数据
export const useSlidesStore = defineStore('slides', () => {
  const slides = ref<Slide[]>([])
  const slideIndex = ref(0)
  
  return {
    slides,
    slideIndex,
  }
})
```

#### Actions 设计
```typescript
// ✅ 推荐：Actions 处理复杂逻辑
export const useSlidesStore = defineStore('slides', () => {
  const slides = ref<Slide[]>([])
  
  // Action 处理复杂逻辑
  const addElement = (element: PPTElement) => {
    const currentSlide = slides.value[slideIndex.value]
    if (!currentSlide.elements) {
      currentSlide.elements = []
    }
    currentSlide.elements.push(element)
  }
  
  // Action 可以调用其他 action
  const addElements = (elements: PPTElement[]) => {
    elements.forEach(element => addElement(element))
  }
  
  return {
    slides,
    addElement,
    addElements,
  }
})
```

### 2. 状态持久化

#### 选择性持久化
```typescript
// ✅ 推荐：只持久化必要的数据
const mainStore = useMainStore()

// 使用 watch 监听变化并保存
watch(
  () => mainStore.$state,
  (state) => {
    // 只保存关键状态
    const { canvasScale, canvasPercentage } = state
    localStorage.setItem('main-store', JSON.stringify({
      canvasScale,
      canvasPercentage,
    }))
  },
  { deep: true }
)
```

## TypeScript 最佳实践

### 1. 类型定义

#### 优先使用 interface
```typescript
// ✅ 推荐：对象类型使用 interface
interface PPTElement {
  id: string
  left: number
  top: number
}

// ✅ 推荐：联合类型、工具类型使用 type
type ElementTypes = 'text' | 'image' | 'shape'
type PartialElement = Partial<PPTElement>
```

#### 类型守卫
```typescript
// ✅ 推荐：使用类型守卫
function isTextElement(element: PPTElement): element is PPTTextElement {
  return element.type === 'text'
}

// 使用
if (isTextElement(element)) {
  console.log(element.content) // 类型安全
}

// ❌ 避免：类型断言
const textElement = element as PPTTextElement // 不安全
console.log(textElement.content) // 可能出错
```

### 2. 泛型使用

#### 泛型约束
```typescript
// ✅ 推荐：使用泛型约束
function getElementById<T extends PPTElement>(
  id: string,
  elements: T[]
): T | undefined {
  return elements.find(el => el.id === id)
}

// 使用
const textElement = getElementById<PPTTextElement>('123', textElements)
```

## 样式开发最佳实践

### 1. SCSS 组织

#### 使用变量和 mixin
```scss
// ✅ 推荐：使用变量
$primary-color: #1890ff;
$border-radius: 4px;

.element {
  background: $primary-color;
  border-radius: $border-radius;
}

// ✅ 推荐：使用 mixin
@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

.container {
  @include flex-center;
}
```

#### BEM 命名规范
```scss
// ✅ 推荐：BEM 命名
.element {
  &__header {
    font-size: 16px;
  }
  
  &__content {
    padding: 20px;
  }
  
  &--selected {
    border: 2px solid $primary-color;
  }
}
```

### 2. 响应式设计

#### 使用 mixin
```scss
// ✅ 推荐：响应式 mixin
@mixin respond-to($breakpoint) {
  @if $breakpoint == 'mobile' {
    @media (max-width: 768px) { @content; }
  } @else if $breakpoint == 'tablet' {
    @media (max-width: 1024px) { @content; }
  }
}

.element {
  width: 100%;
  
  @include respond-to('mobile') {
    width: 50%;
  }
}
```

## 性能优化最佳实践

### 1. 渲染优化

#### 虚拟滚动
```vue
<!-- ✅ 推荐：大列表使用虚拟滚动 -->
<template>
  <VirtualList
    :items="elements"
    :item-size="50"
    :buffer="10"
  >
    <template #default="{ item }">
      <ElementItem :element="item" />
    </template>
  </VirtualList>
</template>
```

#### 懒加载
```vue
<!-- ✅ 推荐：图片懒加载 -->
<template>
  <img
    v-lazy="imageUrl"
    :alt="alt"
  />
</template>

<!-- ✅ 推荐：组件懒加载 -->
<script setup lang="ts">
const HeavyComponent = defineAsyncComponent(() =>
  import('./HeavyComponent.vue')
)
</script>
```

### 2. 事件处理优化

#### 防抖和节流
```typescript
// ✅ 推荐：使用防抖和节流
import { debounce, throttle } from 'lodash-es'

// 防抖：搜索输入
const handleSearch = debounce((keyword: string) => {
  // 执行搜索
}, 300)

// 节流：滚动事件
const handleScroll = throttle(() => {
  // 处理滚动
}, 100)
```

### 3. 内存优化

#### 及时清理
```typescript
// ✅ 推荐：及时清理不需要的数据
export default function useImageLoader() {
  const imageUrls = ref<string[]>([])
  
  const loadImage = (url: string) => {
    const img = new Image()
    img.src = url
    imageUrls.value.push(url)
  }
  
  // 组件卸载时清理
  onUnmounted(() => {
    imageUrls.value.forEach(url => {
      URL.revokeObjectURL(url)
    })
    imageUrls.value = []
  })
}
```

## 错误处理最佳实践

### 1. 异步错误处理

#### 统一错误处理
```typescript
// ✅ 推荐：统一的错误处理
async function savePresentation() {
  try {
    await saveToServer(data)
    Message.success('保存成功')
  } catch (error) {
    console.error('Save failed:', error)
    Message.error('保存失败，请重试')
  }
}

// ✅ 推荐：错误边界
<template>
  <ErrorBoundary>
    <MyComponent />
  </ErrorBoundary>
</template>
```

### 2. 用户输入验证

#### 即时验证
```typescript
// ✅ 推荐：即时验证用户输入
const validateInput = (value: string) => {
  if (!value.trim()) {
    return '内容不能为空'
  }
  if (value.length > 100) {
    return '内容不能超过100个字符'
  }
  return null
}

const errorMessage = computed(() => {
  return validateInput(inputValue.value)
})
```

## 测试最佳实践

### 1. 单元测试

#### 测试工具函数
```typescript
// ✅ 推荐：测试纯函数
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
  
  it('should handle 90 degree rotation', () => {
    const element = {
      left: 100,
      top: 100,
      width: 200,
      height: 100,
      rotate: 90,
    }
    
    const result = getRectRotatedRange(element)
    
    // 验证旋转后的范围
    expect(result.xRange[0]).toBeLessThan(100)
    expect(result.xRange[1]).toBeGreaterThan(300)
  })
})
```

### 2. 组件测试

#### 测试用户交互
```typescript
// ✅ 推荐：测试用户交互
describe('ColorPicker', () => {
  it('should emit change event when color is selected', async () => {
    const wrapper = mount(ColorPicker, {
      props: {
        color: '#1890ff',
      },
    })
    
    await wrapper.find('.saturation').trigger('click', {
      clientX: 100,
      clientY: 100,
    })
    
    expect(wrapper.emitted('change')).toBeTruthy()
  })
})
```

## 总结

遵循这些最佳实践可以：
1. **提高代码质量**: 代码更清晰、更易维护
2. **提升性能**: 减少不必要的渲染和计算
3. **增强可维护性**: 代码结构清晰，易于理解
4. **减少错误**: 类型安全、错误处理完善
5. **提高开发效率**: 遵循约定，减少决策成本

记住：最佳实践不是教条，而是经验的总结。在实际开发中要根据具体情况灵活运用。
