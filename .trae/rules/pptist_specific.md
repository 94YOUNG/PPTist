# PPTist 项目特定规则

## 项目特点

PPTist 是一个功能丰富的在线演示文稿编辑器，具有以下特点：
- 复杂的画布交互（拖拽、缩放、旋转）
- 多种元素类型（文本、图片、形状、图表、表格等）
- 富文本编辑功能
- 历史记录和撤销/重做
- 导入导出多种格式
- AI 辅助生成 PPT

## 关键技术点

### 1. 元素系统

#### 元素类型定义
```typescript
// 所有元素类型
type PPTElement = 
  | PPTTextElement      // 文本
  | PPTImageElement     // 图片
  | PPTShapeElement     // 形状
  | PPTLineElement      // 线条
  | PPTChartElement     // 图表
  | PPTTableElement     // 表格
  | PPTLatexElement     // LaTeX公式
  | PPTVideoElement     // 视频
  | PPTAudioElement     // 音频

// 元素通用属性
interface PPTBaseElement {
  id: string
  left: number
  top: number
  width: number
  height: number
  rotate: number
  lock?: boolean
  groupId?: string
  link?: PPTElementLink
  name?: string
}
```

#### 元素操作原则
1. **不可变数据**: 不要直接修改元素对象，通过 store 更新
2. **批量更新**: 多个属性更新时，使用一次 store.updateElement
3. **性能优先**: 高频操作使用本地状态，结束时更新 store
4. **类型安全**: 使用类型守卫确保类型正确

### 2. 画布系统

#### 坐标系统
```typescript
// 画布坐标：元素在画布中的位置
// 屏幕坐标：鼠标在屏幕上的位置
// 需要考虑缩放比例

// 屏幕坐标转画布坐标
const screenToCanvas = (screenX: number, screenY: number) => {
  const canvasX = (screenX - canvasOffset.x) / canvasScale
  const canvasY = (screenY - canvasOffset.y) / canvasScale
  return { x: canvasX, y: canvasY }
}
```

#### 视口管理
```typescript
// 视口大小：画布的基准宽度（默认1000px）
// 视口比例：画布的高宽比（默认0.5625，即16:9）
// 缩放比例：画布的实际显示缩放

// 计算实际画布大小
const actualWidth = viewportSize * canvasScale
const actualHeight = viewportSize * viewportRatio * canvasScale
```

### 3. 交互系统

#### 拖拽流程
```typescript
// 1. 鼠标按下：记录初始位置
const handleMouseDown = (e: MouseEvent, element: PPTElement) => {
  isDragging.value = true
  startX = e.clientX
  startY = e.clientY
  originLeft = element.left
  originTop = element.top
}

// 2. 鼠标移动：更新位置（使用节流）
const handleMouseMove = throttle((e: MouseEvent) => {
  if (!isDragging.value) return
  
  const deltaX = (e.clientX - startX) / canvasScale
  const deltaY = (e.clientY - startY) / canvasScale
  
  // 更新本地状态
  localElement.left = originLeft + deltaX
  localElement.top = originTop + deltaY
  
  // 显示对齐吸附线
  updateAlignmentLines()
}, 16)

// 3. 鼠标松开：更新 store，记录快照
const handleMouseUp = () => {
  isDragging.value = false
  
  // 更新 store
  slidesStore.updateElement({
    id: element.id,
    props: { left: localElement.left, top: localElement.top },
  })
  
  // 记录快照
  addHistorySnapshot()
}
```

#### 对齐吸附
```typescript
// 吸附阈值：5像素
const SNAP_THRESHOLD = 5

// 计算对齐吸附线
const calculateAlignmentLines = () => {
  const lines: AlignmentLineProps[] = []
  
  // 遍历其他元素，收集对齐线
  elementList.value.forEach(el => {
    if (el.id === currentElement.id) return
    
    // 水平对齐线：顶部、底部、中心
    lines.push({
      type: 'horizontal',
      axis: { x: el.left, y: el.top },
      length: el.width,
    })
    
    // 垂直对齐线：左侧、右侧、中心
    lines.push({
      type: 'vertical',
      axis: { x: el.left, y: el.top },
      length: el.height,
    })
  })
  
  return lines
}

// 检查是否吸附
const checkSnap = (value: number, lines: number[]) => {
  for (const line of lines) {
    if (Math.abs(value - line) < SNAP_THRESHOLD) {
      return line
    }
  }
  return null
}
```

### 4. 富文本编辑

#### ProseMirror 集成
```typescript
// Schema 定义
const schema = new Schema({
  nodes: {
    doc: { content: 'block+' },
    paragraph: {
      content: 'inline*',
      group: 'block',
      parseDOM: [{ tag: 'p' }],
      toDOM: () => ['p', 0],
    },
    text: { group: 'inline' },
  },
  marks: {
    bold: {
      parseDOM: [{ tag: 'strong' }],
      toDOM: () => ['strong', 0],
    },
    italic: {
      parseDOM: [{ tag: 'em' }],
      toDOM: () => ['em', 0],
    },
  },
})

// 编辑器初始化
const view = new EditorView(element, {
  state: EditorState.create({
    schema,
    plugins: [
      keymap(baseKeymap),
      history(),
      placeholder('请输入内容'),
    ],
  }),
  dispatchTransaction: (transaction) => {
    const newState = view.state.apply(transaction)
    view.updateState(newState)
    
    // 同步到元素数据
    const content = newState.doc.toJSON()
    slidesStore.updateElement({
      id: element.id,
      props: { content },
    })
  },
})
```

### 5. 历史记录

#### 快照机制
```typescript
// 快照数据结构
interface SnapshotData {
  slides: Slide[]
  slideIndex: number
}

// 添加快照
const addHistorySnapshot = () => {
  const snapshot: SnapshotData = {
    slides: cloneDeep(slidesStore.slides),
    slideIndex: slidesStore.slideIndex,
  }
  
  // 如果当前不在最新位置，删除后面的快照
  if (snapshotIndex.value < snapshots.value.length - 1) {
    snapshots.value = snapshots.value.slice(0, snapshotIndex.value + 1)
  }
  
  // 添加新快照
  snapshots.value.push(snapshot)
  snapshotIndex.value = snapshots.value.length - 1
  
  // 限制快照数量
  if (snapshots.value.length > MAX_SNAPSHOTS) {
    snapshots.value.shift()
    snapshotIndex.value--
  }
}

// 撤销
const undo = () => {
  if (snapshotIndex.value > 0) {
    snapshotIndex.value--
    const snapshot = snapshots.value[snapshotIndex.value]
    slidesStore.setSlides(snapshot.slides)
    slidesStore.setSlideIndex(snapshot.slideIndex)
  }
}

// 重做
const redo = () => {
  if (snapshotIndex.value < snapshots.value.length - 1) {
    snapshotIndex.value++
    const snapshot = snapshots.value[snapshotIndex.value]
    slidesStore.setSlides(snapshot.slides)
    slidesStore.setSlideIndex(snapshot.slideIndex)
  }
}
```

### 6. 导入导出

#### 导出 PPTX
```typescript
import PptxGenJS from 'pptxgenjs'

const exportToPPTX = (slides: Slide[]) => {
  const pptx = new PptxGenJS()
  
  slides.forEach(slide => {
    const pptxSlide = pptx.addSlide()
    
    // 设置背景
    if (slide.background) {
      pptxSlide.background = slide.background
    }
    
    // 添加元素
    slide.elements.forEach(element => {
      switch (element.type) {
        case 'text':
          pptxSlide.addText(element.content, {
            x: element.left / 1000, // 转换为英寸
            y: element.top / 1000,
            w: element.width / 1000,
            h: element.height / 1000,
            fontSize: element.defaultFontSize,
            fontFace: element.defaultFontName,
            color: element.defaultColor.replace('#', ''),
          })
          break
        
        case 'image':
          pptxSlide.addImage({
            url: element.src,
            x: element.left / 1000,
            y: element.top / 1000,
            w: element.width / 1000,
            h: element.height / 1000,
          })
          break
        
        // ... 其他元素类型
      }
    })
  })
  
  // 保存文件
  pptx.writeFile({ fileName: 'presentation.pptx' })
}
```

#### 导入 PPTX
```typescript
import pptxtojson from 'pptxtojson'

const importFromPPTX = async (file: File) => {
  const arrayBuffer = await file.arrayBuffer()
  const slides = await pptxtojson(arrayBuffer)
  
  // 转换为项目格式
  const convertedSlides = slides.map(slide => ({
    id: nanoid(10),
    background: slide.background,
    elements: slide.elements.map(element => {
      const baseElement = {
        id: nanoid(10),
        left: element.left,
        top: element.top,
        width: element.width,
        height: element.height,
        rotate: element.rotate || 0,
      }
      
      switch (element.type) {
        case 'text':
          return {
            ...baseElement,
            type: 'text',
            content: element.content,
            defaultFontName: element.fontName || 'Microsoft YaHei',
            defaultFontSize: element.fontSize || 18,
            defaultColor: element.color || '#000000',
          }
        
        case 'image':
          return {
            ...baseElement,
            type: 'image',
            src: element.src,
          }
        
        // ... 其他元素类型
      }
    }),
  }))
  
  return convertedSlides
}
```

## 性能优化要点

### 1. 渲染优化
- 使用虚拟滚动处理大量元素
- 使用 transform 代替 left/top
- 使用 will-change 提示浏览器
- 避免频繁的 DOM 操作

### 2. 计算优化
- 使用节流处理高频事件
- 缓存计算结果
- 使用 Web Worker 处理复杂计算

### 3. 内存优化
- 及时清理不需要的数据
- 限制历史记录数量
- 使用 shallowRef 处理大数据

## 代码风格要求

### 1. 命名约定
```typescript
// 元素相关
const element = ref<PPTElement>()
const elementList = ref<PPTElement[]>([])
const activeElementIdList = ref<string[]>([])

// 方法命名
const createElement = () => {}
const updateElement = () => {}
const deleteElement = () => {}

// 事件处理
const handleMouseDown = () => {}
const handleMouseMove = () => {}
const handleMouseUp = () => {}
```

### 2. 注释要求
```typescript
/**
 * 计算元素在画布中的矩形范围旋转后的新位置范围
 * @param element 元素的位置大小和旋转角度信息
 * @returns 返回旋转后的x和y轴范围
 */
export function getRectRotatedRange(element: RotatedElementData) {
  // ...
}

// 复杂逻辑需要注释
// 计算旋转后的边界范围
// 使用三角函数计算四个角的新位置
const radius = Math.sqrt(Math.pow(width, 2) + Math.pow(height, 2)) / 2
```

### 3. 错误处理
```typescript
// 用户操作需要友好提示
async function savePresentation() {
  try {
    await saveToServer(data)
    Message.success('保存成功')
  } catch (error) {
    console.error('Save failed:', error)
    Message.error('保存失败，请重试')
  }
}

// 边界条件检查
const updateElement = (id: string, props: Partial<PPTElement>) => {
  const element = getElementById(id)
  if (!element) {
    console.warn(`Element not found: ${id}`)
    return
  }
  
  // 更新元素
}
```

## 测试要点

### 1. 单元测试
- 测试工具函数（如 getRectRotatedRange）
- 测试 hooks（如 useDragElement）
- 测试 store actions

### 2. 集成测试
- 测试元素创建流程
- 测试拖拽交互
- 测试历史记录功能

### 3. E2E 测试
- 测试完整的编辑流程
- 测试导入导出功能
- 测试演示放映功能

## 注意事项

1. **性能敏感**: 画布操作频繁，注意性能优化
2. **类型安全**: 严格遵循 TypeScript 类型定义
3. **响应式陷阱**: 避免在循环中创建响应式数据
4. **内存管理**: 大文件和图片注意内存释放
5. **浏览器兼容**: 支持现代浏览器，不考虑 IE
6. **用户体验**: 操作需要流畅，响应需要及时
7. **错误处理**: 所有用户操作都需要错误处理
8. **数据验证**: 导入数据需要验证格式

## 开发建议

1. **先理解后修改**: 仔细阅读现有代码，理解设计意图
2. **小步提交**: 每次提交只做一件事
3. **测试驱动**: 重要功能先写测试
4. **性能优先**: 考虑性能影响
5. **用户视角**: 从用户角度思考功能
6. **文档更新**: 重要修改需要更新文档
7. **代码审查**: 提交 PR 前自我审查
8. **持续重构**: 发现问题及时重构
