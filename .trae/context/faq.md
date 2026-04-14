# 常见问题解答

## 开发环境问题

### 1. 项目启动失败

#### 问题：npm install 失败
```bash
# 解决方案 1：清除缓存
npm cache clean --force
rm -rf node_modules package-lock.json
npm install

# 解决方案 2：使用国内镜像
npm config set registry https://registry.npmmirror.com
npm install

# 解决方案 3：使用 pnpm
npm install -g pnpm
pnpm install
```

#### 问题：TypeScript 类型错误
```bash
# 解决方案：重新生成类型声明
npm run type-check

# 或者重启 TypeScript 服务
# VS Code: Cmd+Shift+P -> TypeScript: Restart TS Server
```

### 2. 热更新不生效

#### 问题：修改代码后页面不更新
```typescript
// 解决方案：检查 vite.config.ts
export default defineConfig({
  server: {
    watch: {
      usePolling: true, // 在某些系统上需要启用轮询
    },
  },
})
```

## 组件开发问题

### 1. Props 类型错误

#### 问题：Props 类型不匹配
```vue
<!-- ❌ 错误写法 -->
<script setup lang="ts">
const props = defineProps({
  element: Object, // 类型不明确
})
</script>

<!-- ✅ 正确写法 -->
<script setup lang="ts">
interface Props {
  element: PPTElement
}
const props = defineProps<Props>()
</script>
```

### 2. 响应式丢失

#### 问题：解构后失去响应式
```typescript
// ❌ 错误写法
const { activeElementIdList } = useMainStore()
// activeElementIdList 不是响应式的

// ✅ 正确写法 1：使用 storeToRefs
import { storeToRefs } from 'pinia'
const { activeElementIdList } = storeToRefs(useMainStore())

// ✅ 正确写法 2：直接使用 store
const mainStore = useMainStore()
mainStore.activeElementIdList // 响应式的

// ✅ 正确写法 3：使用 computed
const activeElementIdList = computed(() => mainStore.activeElementIdList)
```

### 3. 组件通信问题

#### 问题：子组件无法触发父组件事件
```vue
<!-- ❌ 错误写法 -->
<!-- 子组件 -->
<script setup lang="ts">
const emit = defineEmits(['update'])

const handleClick = () => {
  emit('update') // 没有传递参数
}
</script>

<!-- 父组件 -->
<ChildComponent @update="handleUpdate" />

<!-- ✅ 正确写法 -->
<!-- 子组件 -->
<script setup lang="ts">
interface Emits {
  (e: 'update', value: PPTElement): void
}
const emit = defineEmits<Emits>()

const handleClick = () => {
  emit('update', props.element) // 传递参数
}
</script>

<!-- 父组件 -->
<ChildComponent @update="handleUpdate" />
```

## 状态管理问题

### 1. Store 状态不同步

#### 问题：多个组件使用同一个 store，状态不同步
```typescript
// ❌ 错误写法：在组件内部创建 store 实例
export default {
  setup() {
    const mainStore = useMainStore() // 每次调用都创建新实例
    return { mainStore }
  }
}

// ✅ 正确写法：在组件外部创建 store 实例
const mainStore = useMainStore()

export default {
  setup() {
    return { mainStore }
  }
}

// ✅ 最佳实践：使用 Pinia 的单例模式
// Pinia 默认就是单例，直接使用即可
const mainStore = useMainStore()
```

### 2. 持久化问题

#### 问题：页面刷新后状态丢失
```typescript
// 解决方案：使用 localStorage 或 IndexedDB 持久化

// 方案 1：手动持久化
const mainStore = useMainStore()

// 保存
watch(
  () => mainStore.$state,
  (state) => {
    localStorage.setItem('main-store', JSON.stringify(state))
  },
  { deep: true }
)

// 恢复
const savedState = localStorage.getItem('main-store')
if (savedState) {
  mainStore.$patch(JSON.parse(savedState))
}

// 方案 2：使用 pinia-plugin-persistedstate
import { createPinia } from 'pinia'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

const pinia = createPinia()
pinia.use(piniaPluginPersistedstate)
```

## 元素操作问题

### 1. 元素拖拽卡顿

#### 问题：拖拽元素时卡顿
```typescript
// ❌ 错误写法：每次移动都更新 store
const handleMouseMove = (e: MouseEvent) => {
  slidesStore.updateElement({
    id: element.id,
    props: { left: e.clientX, top: e.clientY },
  })
}

// ✅ 正确写法：使用节流 + 本地状态
const localElement = ref({ ...props.element })

const handleMouseMove = throttle((e: MouseEvent) => {
  localElement.value.left = e.clientX
  localElement.value.top = e.clientY
}, 16)

const handleMouseUp = () => {
  // 只在松开鼠标时更新 store
  slidesStore.updateElement({
    id: element.id,
    props: localElement.value,
  })
}
```

### 2. 元素旋转计算错误

#### 问题：旋转后的元素位置计算不正确
```typescript
// 解决方案：使用正确的旋转计算公式
import { getRectRotatedRange } from '@/utils/element'

// 计算旋转后的边界范围
const { xRange, yRange } = getRectRotatedRange({
  left: element.left,
  top: element.top,
  width: element.width,
  height: element.height,
  rotate: element.rotate,
})

// xRange: [minX, maxX]
// yRange: [minY, maxY]
```

### 3. 元素对齐吸附不生效

#### 问题：元素拖拽时没有对齐吸附线
```typescript
// 检查以下几点：

// 1. 确保对齐吸附线数据正确
const alignmentLines = computed(() => {
  const lines: AlignmentLineProps[] = []
  
  // 收集其他元素的对齐线
  elementList.value.forEach(el => {
    if (el.id === currentElement.id) return
    
    // 添加水平对齐线
    lines.push({
      type: 'horizontal',
      axis: { x: el.left, y: el.top },
      length: el.width,
    })
    
    // 添加垂直对齐线
    lines.push({
      type: 'vertical',
      axis: { x: el.left, y: el.top },
      length: el.height,
    })
  })
  
  return lines
})

// 2. 确保吸附阈值合理
const SNAP_THRESHOLD = 5 // 像素

// 3. 确保对齐线渲染
<AlignmentLine
  v-for="line in alignmentLines"
  :key="`${line.type}-${line.axis.x}-${line.axis.y}`"
  :type="line.type"
  :axis="line.axis"
  :length="line.length"
/>
```

## 富文本编辑问题

### 1. 文本内容不更新

#### 问题：修改文本内容后，元素不更新
```typescript
// ❌ 错误写法：直接修改 content
element.content = newContent

// ✅ 正确写法：通过 store 更新
slidesStore.updateElement({
  id: element.id,
  props: { content: newContent },
})

// ✅ 正确写法：使用 ProseMirror 的更新机制
editorView.updateState(
  EditorState.create({
    doc: ProseMirrorNode.fromJSON(schema, newContent),
    plugins: editorView.state.plugins,
  })
)
```

### 2. 文本样式丢失

#### 问题：粘贴文本时样式丢失
```typescript
// 解决方案：处理剪贴板内容
import { EditorView } from 'prosemirror-view'

const handlePaste = (view: EditorView, event: ClipboardEvent) => {
  const text = event.clipboardData?.getData('text/plain')
  if (text) {
    // 插入纯文本，保留当前样式
    const transaction = view.state.tr.insertText(text)
    view.dispatch(transaction)
    return true
  }
  return false
}

// 注册插件
new EditorView(element, {
  state,
  handlePaste,
})
```

## 导入导出问题

### 1. 导出 PPTX 失败

#### 问题：导出的 PPTX 文件无法打开
```typescript
// 检查以下几点：

// 1. 确保图片格式正确
const imageFormats = ['png', 'jpg', 'jpeg', 'gif', 'bmp']

// 2. 确保图片 URL 可访问
const loadImage = async (url: string): Promise<string> => {
  try {
    const response = await fetch(url)
    const blob = await response.blob()
    return await blobToBase64(blob)
  } catch (error) {
    console.error('Failed to load image:', url)
    return ''
  }
}

// 3. 确保文本内容编码正确
slide.addText(text, {
  x: left,
  y: top,
  w: width,
  h: height,
  fontSize: fontSize,
  fontFace: fontName,
  color: color.replace('#', ''), // 移除 # 号
})
```

### 2. 导入 PPTX 格式错误

#### 问题：导入的 PPTX 内容解析错误
```typescript
// 解决方案：使用 pptxtojson 库
import pptxtojson from 'pptxtojson'

const importPPTX = async (file: File) => {
  try {
    const arrayBuffer = await file.arrayBuffer()
    const slides = await pptxtojson(arrayBuffer)
    
    // 验证数据格式
    if (!Array.isArray(slides)) {
      throw new Error('Invalid PPTX format')
    }
    
    // 转换为项目格式
    const convertedSlides = slides.map(slide => ({
      id: nanoid(10),
      elements: slide.elements.map(element => ({
        id: nanoid(10),
        ...element,
      })),
    }))
    
    return convertedSlides
  } catch (error) {
    console.error('Failed to import PPTX:', error)
    throw error
  }
}
```

## 性能问题

### 1. 页面加载慢

#### 问题：首次加载时间过长
```typescript
// 解决方案：

// 1. 代码分割
const Editor = () => import('@/views/Editor/index.vue')
const Screen = () => import('@/views/Screen/index.vue')

// 2. 资源预加载
<link rel="preload" href="/fonts/Inter.woff2" as="font" type="font/woff2" crossorigin>

// 3. 图片懒加载
<img v-lazy="imageUrl" />

// 4. 第三方库按需加载
import { debounce } from 'lodash-es' // 只导入需要的函数
```

### 2. 内存占用过高

#### 问题：长时间使用后内存占用过高
```typescript
// 解决方案：

// 1. 及时清理不需要的数据
onUnmounted(() => {
  // 清理定时器
  clearInterval(timer)
  
  // 清理事件监听
  window.removeEventListener('resize', handleResize)
  
  // 清理大对象
  largeData.value = null
})

// 2. 使用 shallowRef 处理大数据
const largeData = shallowRef<BigData>({})

// 3. 限制历史记录数量
const MAX_SNAPSHOTS = 50
if (snapshots.length > MAX_SNAPSHOTS) {
  snapshots.shift() // 删除最旧的快照
}
```

### 3. 拖拽卡顿

#### 问题：拖拽大量元素时卡顿
```typescript
// 解决方案：

// 1. 使用节流
const handleMouseMove = throttle((e: MouseEvent) => {
  updateElementPosition(e)
}, 16) // 60fps

// 2. 使用 transform 代替 left/top
// ❌ 错误写法
element.style.left = `${x}px`
element.style.top = `${y}px`

// ✅ 正确写法
element.style.transform = `translate(${x}px, ${y}px)`

// 3. 使用 will-change 提示浏览器
.element {
  will-change: transform;
}
```

## 样式问题

### 1. 样式不生效

#### 问题：修改样式后不生效
```vue
<!-- 检查以下几点： -->

<!-- 1. 确保使用了 scoped -->
<style scoped lang="scss">
.element {
  color: red;
}
</style>

<!-- 2. 确保选择器正确 -->
<style scoped lang="scss">
.parent {
  .child { /* 嵌套选择器 */
    color: red;
  }
}
</style>

<!-- 3. 使用深度选择器穿透 scoped -->
<style scoped lang="scss">
.parent :deep(.child) {
  color: red;
}
</style>
```

### 2. 响应式布局问题

#### 问题：移动端布局错乱
```scss
// 解决方案：使用响应式 mixin
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
    font-size: 14px;
  }
}
```

## 调试技巧

### 1. Vue DevTools

```typescript
// 在 DevTools 中查看组件状态
// 1. 安装 Vue DevTools 浏览器扩展
// 2. 打开 DevTools，切换到 Vue 标签
// 3. 选择组件查看 props、data、computed 等
```

### 2. Pinia DevTools

```typescript
// 在 DevTools 中查看 store 状态
// 1. Pinia 自动集成到 Vue DevTools
// 2. 切换到 Pinia 标签
// 3. 查看所有 store 的状态和 actions
```

### 3. Console 调试

```typescript
// 在组件中打印调试信息
import { getCurrentInstance } from 'vue'

export default {
  setup() {
    const instance = getCurrentInstance()
    console.log('Component instance:', instance)
    
    // 打印响应式数据
    const state = ref({ count: 0 })
    watchEffect(() => {
      console.log('State changed:', state.value)
    })
  }
}
```

## 总结

遇到问题时，按以下步骤排查：
1. **查看控制台错误**: 检查是否有错误信息
2. **检查网络请求**: 确认 API 调用是否正常
3. **使用 DevTools**: 查看组件状态和 store 数据
4. **阅读文档**: 查阅 Vue、Pinia、TypeScript 文档
5. **搜索问题**: 在 GitHub Issues 或 Stack Overflow 搜索
6. **提问**: 提供详细的错误信息和复现步骤

记住：大多数问题都有解决方案，关键是找到正确的排查方法。
