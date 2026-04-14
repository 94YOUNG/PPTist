# PPTist 架构说明

## 整体架构

```
┌─────────────────────────────────────────────────────────┐
│                    PPTist 应用架构                        │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   Editor     │  │   Screen     │  │   Mobile     │  │
│  │   编辑器      │  │   演示模式    │  │   移动端      │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │              Views Layer (视图层)                  │   │
│  │  - Canvas (画布)                                  │   │
│  │  - Toolbar (工具栏)                               │   │
│  │  - Thumbnails (缩略图)                            │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │           Components Layer (组件层)               │   │
│  │  - UI Components (Button, Input, Modal...)       │   │
│  │  - Element Components (Text, Image, Shape...)    │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │            Hooks Layer (业务逻辑层)               │   │
│  │  - useCreateElement (创建元素)                    │   │
│  │  - useDragElement (拖拽元素)                      │   │
│  │  - useHistorySnapshot (历史记录)                  │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │            Store Layer (状态管理层)               │   │
│  │  - MainStore (全局状态)                           │   │
│  │  - SlidesStore (幻灯片数据)                       │   │
│  │  - SnapshotStore (快照数据)                       │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │            Utils Layer (工具层)                   │   │
│  │  - element.ts (元素计算)                          │   │
│  │  - canvas.ts (画布计算)                           │   │
│  │  - prosemirror (富文本)                           │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │           Services Layer (服务层)                 │   │
│  │  - API Services (接口调用)                        │   │
│  │  - Database (IndexedDB)                           │   │
│  │  - Export/Import (导入导出)                       │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

## 核心模块

### 1. 状态管理 (Store)

#### MainStore - 全局状态
```typescript
{
  activeElementIdList: string[]      // 当前选中的元素ID列表
  handleElementId: string            // 正在操作的元素ID
  canvasScale: number                // 画布缩放比例
  canvasPercentage: number           // 画布可视区域百分比
  creatingElement: CreatingElement   // 正在创建的元素
  toolbarState: ToolbarStates        // 工具栏状态
  richTextAttrs: TextAttrs           // 富文本属性
  textFormatPainter: TextFormatPainter | null  // 文字格式刷
  shapeFormatPainter: ShapeFormatPainter | null // 形状格式刷
}
```

#### SlidesStore - 幻灯片数据
```typescript
{
  slides: Slide[]                    // 幻灯片列表
  slideIndex: number                 // 当前幻灯片索引
  theme: SlideTheme                  // 主题配置
  viewportRatio: number              // 视口比例
  viewportSize: number               // 视口大小
}
```

#### SnapshotStore - 历史快照
```typescript
{
  snapshotData: SnapshotData[]       // 快照数据
  snapshotIndex: number              // 当前快照索引
}
```

### 2. 元素系统

#### 元素类型定义
```typescript
// 基础元素属性
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

// 具体元素类型
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
```

#### 元素操作流程
```
创建元素 → useCreateElement
    ↓
添加到画布 → SlidesStore.addElement
    ↓
选中元素 → MainStore.setActiveElementIdList
    ↓
操作元素 → useDragElement / useScaleElement / useRotateElement
    ↓
更新元素 → SlidesStore.updateElement
    ↓
记录快照 → SnapshotStore.addSnapshot
```

### 3. 画布系统

#### 画布组件结构
```
Canvas/
├── ViewportBackground.vue    # 画布背景
├── EditableElement.vue       # 可编辑元素容器
├── AlignmentLine.vue         # 对齐吸附线
├── GridLines.vue            # 网格线
├── Ruler.vue                # 标尺
├── MouseSelection.vue       # 框选组件
├── ElementCreateSelection.vue # 元素创建选区
└── Operate/                 # 操作组件
    ├── ResizeHandler.vue    # 缩放手柄
    ├── RotateHandler.vue    # 旋转手柄
    └── BorderLine.vue       # 边框线
```

#### 画布交互流程
```
用户点击元素
    ↓
useSelectElement.handleSelectElement
    ↓
更新 activeElementIdList
    ↓
渲染选中状态（边框、手柄）
    ↓
用户拖拽 → useDragElement
用户缩放 → useScaleElement
用户旋转 → useRotateElement
    ↓
实时更新元素位置/大小/角度
    ↓
显示对齐吸附线
    ↓
松开鼠标 → 记录快照
```

### 4. 富文本编辑器

#### ProseMirror 集成
```
prosemirror/
├── schema/              # 文档模式
│   ├── nodes.ts        # 节点定义
│   └── marks.ts        # 标记定义
├── commands/            # 编辑命令
│   ├── setTextAlign.ts # 设置对齐
│   ├── toggleList.ts   # 切换列表
│   └── replaceText.ts  # 替换文本
└── plugins/             # 插件
    ├── keymap.ts       # 快捷键
    ├── inputrules.ts   # 输入规则
    └── placeholder.ts  # 占位符
```

#### 文本编辑流程
```
用户输入
    ↓
ProseMirror 处理输入
    ↓
应用 schema 验证
    ↓
执行 commands
    ↓
更新文档状态
    ↓
同步到 PPTTextElement.content
    ↓
触发重新渲染
```

### 5. 历史记录系统

#### 快照机制
```typescript
interface SnapshotData {
  slides: Slide[]          // 幻灯片数据
  slideIndex: number       // 当前索引
}

// 撤销/重做流程
用户操作 → addHistorySnapshot()
    ↓
保存当前状态到 snapshotData
    ↓
用户撤销 → undo()
    ↓
恢复上一个快照
    ↓
更新 SlidesStore
```

### 6. 导入导出系统

#### 导出流程
```
用户点击导出
    ↓
选择导出格式
    ↓
├─ JSON → 直接序列化数据
├─ PPTX → pptxgenjs 生成
├─ PDF → html-to-image + PDF库
└─ 图片 → html-to-image
    ↓
触发浏览器下载
```

#### 导入流程
```
用户选择文件
    ↓
读取文件内容
    ↓
├─ JSON → 直接解析
├─ PPTX → pptxtojson 转换
└─ 图片 → 创建图片元素
    ↓
验证数据格式
    ↓
更新 SlidesStore
```

### 7. AI PPT 生成

#### AI 生成流程
```
用户输入主题
    ↓
调用 AI 接口
    ↓
获取大纲结构
    ↓
生成幻灯片内容
    ↓
应用模板样式
    ↓
创建幻灯片
```

## 数据流

### 单向数据流
```
User Action (用户操作)
    ↓
Event Handler (事件处理)
    ↓
Hook Function (业务逻辑)
    ↓
Store Action (状态更新)
    ↓
State Update (状态变化)
    ↓
Component Re-render (组件重渲染)
    ↓
UI Update (界面更新)
```

### 示例：创建文本元素
```
用户点击"插入文本"
    ↓
Toolbar 触发 insertText
    ↓
useCreateElement.createTextElement
    ↓
创建文本元素对象
    ↓
SlidesStore.addElement
    ↓
MainStore.setActiveElementIdList
    ↓
Canvas 渲染新元素
    ↓
显示选中状态
    ↓
useHistorySnapshot.addHistorySnapshot
```

## 性能优化策略

### 1. 虚拟滚动
- 缩略图列表使用虚拟滚动
- 大量元素时只渲染可视区域

### 2. 防抖节流
- 拖拽、缩放等高频操作使用节流
- 搜索、自动保存使用防抖

### 3. 懒加载
- 图片懒加载
- 字体懒加载
- 组件懒加载

### 4. 缓存优化
- computed 缓存计算结果
- 避免不必要的响应式

### 5. 渲染优化
- 使用 key 优化列表渲染
- 合理使用 v-show vs v-if
- 减少组件嵌套层级

## 扩展性设计

### 1. 插件系统
- 元素类型可扩展
- 工具栏可扩展
- 导出格式可扩展

### 2. 主题系统
- CSS 变量实现主题切换
- 主题配置可自定义

### 3. 配置驱动
- 形状库配置化
- 图表类型配置化
- 动画效果配置化

## 安全考虑

### 1. XSS 防护
- 富文本内容过滤
- 用户输入转义

### 2. 数据验证
- 导入数据验证
- API 参数验证

### 3. 权限控制
- 锁定元素不可编辑
- 只读模式

## 浏览器兼容性

### 支持的浏览器
- Chrome >= 90
- Firefox >= 88
- Safari >= 14
- Edge >= 90

### 不支持
- IE 11 及以下
- 旧版移动浏览器
