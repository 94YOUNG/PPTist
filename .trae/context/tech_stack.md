# 技术栈详解

## 核心框架

### Vue 3.5.17

- **Composition API**: 使用 `<script setup>` 语法
- **响应式系统**: ref, reactive, computed, watch
- **生命周期**: onMounted, onUnmounted 等
- **依赖注入**: provide/inject
- **Teleport**: 模态框等组件
- **Suspense**: 异步组件加载

### TypeScript 5.3

- **严格模式**: 启用所有严格类型检查
- **类型推导**: 充分利用类型推导
- **泛型**: 提高代码复用性
- **装饰器**: 未使用（Vue 3 使用 Composition API）
- **类型声明**: .d.ts 文件

### Pinia 3.0.2

- **Composition API**: 使用 setup 语法
- **模块化**: 按功能划分 store
- **持久化**: 结合 IndexedDB
- **DevTools**: 完整的调试支持

## 构建工具

### Vite 5.3.5

- **快速启动**: 原生 ESM 开发服务器
- **HMR**: 热模块替换
- **构建优化**: Rollup 打包
- **插件系统**: 丰富的插件生态

#### Vite 插件

```typescript
// vite.config.ts
plugins: [
  vue(), // Vue 支持
  AutoImport({
    // 自动导入
    imports: ['vue', 'vue-router'],
    dts: 'src/auto-imports.d.ts'
  }),
  Components({
    // 组件自动导入
    resolvers: [
      IconsResolver({
        // 图标自动导入
        prefix: 'Icon'
      })
    ],
    dts: 'components.d.ts'
  }),
  Icons({
    // 图标编译
    autoInstall: true
  })
]
```

## 样式方案

### SCSS

- **预处理器**: 变量、嵌套、mixin
- **模块化**: 按组件拆分样式文件
- **全局样式**: `src/assets/styles/global.scss`
- **CSS 变量**: 主题切换支持

### 样式工具

```scss
// 变量定义
$primary-color: #1890ff;
$border-radius: 4px;

// Mixin
@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

@mixin respond-to($breakpoint) {
  @if $breakpoint == 'mobile' {
    @media (max-width: 768px) {
      @content;
    }
  }
}
```

## UI 组件库

### 自定义组件

项目未使用第三方 UI 库，自行实现以下组件：

- Button, ButtonGroup
- Input, TextArea, NumberInput
- Select, SelectCustom, SelectGroup
- Checkbox, CheckboxButton
- RadioGroup, RadioButton
- Switch, Slider
- ColorPicker
- Modal, Drawer, Popover
- Message, Tooltip
- Tabs, Divider

### 图标方案

- **unplugin-icons**: 自动导入图标
- **@iconify**: 图标库
- **自定义 SVG**: 项目特定图标

```typescript
// 使用图标
import IconParkOutlineEdit from '~icons/icon-park-outline/edit'

<IconParkOutlineEdit />
```

## 富文本编辑器

### ProseMirror

- **文档模型**: Node, Mark, Schema
- **状态管理**: EditorState, EditorView
- **命令系统**: Commands, Keymap
- **插件系统**: Plugin, PluginKey

#### 核心模块

```typescript
// Schema 定义
const schema = new Schema({
  nodes: {
    doc: { ... },
    paragraph: { ... },
    text: { ... },
  },
  marks: {
    bold: { ... },
    italic: { ... },
    link: { ... },
  }
})

// 编辑器初始化
const view = new EditorView(element, {
  state: EditorState.create({
    schema,
    plugins: [
      keymap(baseKeymap),
      history(),
      placeholder('请输入内容'),
    ]
  })
})
```

## 数据可视化

### ECharts 6.0.0

- **图表类型**: 折线图、柱状图、饼图、散点图等
- **主题定制**: 自定义主题颜色
- **响应式**: 自动调整大小

```typescript
// 图表配置
const option = {
  color: themeColors,
  xAxis: { type: 'category' },
  yAxis: { type: 'value' },
  series: [
    {
      type: 'line',
      data: chartData
    }
  ]
}
```

## 数据存储

### IndexedDB (Dexie 4.0.11)

- **本地存储**: 幻灯片数据持久化
- **版本管理**: 数据库版本控制
- **索引优化**: 快速查询

```typescript
// 数据库定义
const db = new Dexie('PPTistDatabase')
db.version(1).stores({
  slides: '++id, data, createdAt',
  images: '++id, blob, createdAt'
})
```

## 导出功能

### pptxgenjs 3.12.0

- **PPTX 生成**: 创建 PowerPoint 文件
- **样式支持**: 文本、形状、图片、图表
- **动画支持**: 部分动画效果

```typescript
// 创建 PPTX
const pptx = new PptxGenJS()
const slide = pptx.addSlide()
slide.addText('Hello World', {
  x: 1,
  y: 1,
  fontSize: 24,
  color: '363636'
})
pptx.writeFile('presentation.pptx')
```

### html-to-image 1.11.13

- **截图**: 将 DOM 转换为图片
- **格式支持**: PNG, JPEG, SVG
- **质量优化**: 支持缩放和质量设置

```typescript
// 生成图片
const dataUrl = await toPng(element, {
  quality: 1,
  pixelRatio: 2
})
```

## 工具库

### Lodash 4.17.21

- **工具函数**: debounce, throttle, cloneDeep
- **集合操作**: map, filter, find
- **对象操作**: get, set, merge

```typescript
import { debounce, cloneDeep } from 'lodash-es'

const handleResize = debounce(() => {
  // 处理 resize
}, 300)
```

### nanoid 5.0.7

- **唯一 ID**: 生成唯一标识符
- **自定义长度**: 可配置 ID 长度
- **自定义字符**: 可配置字符集

```typescript
import { nanoid } from 'nanoid'

const id = nanoid(10) // 生成 10 位 ID
```

### tinycolor2 1.6.0

- **颜色处理**: 解析、转换、操作颜色
- **颜色计算**: 混合、变亮、变暗
- **格式转换**: RGB, HSL, HEX

```typescript
import tinycolor from 'tinycolor2'

const color = tinycolor('#1890ff')
const lighter = color.lighten(20).toString()
const rgba = color.setAlpha(0.5).toRgbString()
```

## 网络请求

### Axios 1.7.9

- **HTTP 客户端**: Promise based
- **拦截器**: 请求/响应拦截
- **错误处理**: 统一错误处理

```typescript
// axios 配置
const instance = axios.create({
  baseURL: '/api',
  timeout: 10000
})

instance.interceptors.response.use(
  (response) => response.data,
  (error) => {
    Message.error('请求失败')
    return Promise.reject(error)
  }
)
```

## 加密

### crypto-js 4.2.0

- **加密算法**: AES, SHA, MD5
- **数据加密**: 敏感数据加密存储

```typescript
import CryptoJS from 'crypto-js'

const encrypted = CryptoJS.AES.encrypt(data, secretKey).toString()
```

## 数学公式

### hfmath 0.0.2

- **LaTeX 渲染**: 将 LaTeX 转换为 SVG
- **公式支持**: 常用数学公式

```typescript
import { Hfmath } from 'hfmath'

const latex = 'E = mc^2'
const svg = new Hfmath(latex).svg()
```

## 代码质量

### ESLint 8.49.0

- **代码检查**: JavaScript/TypeScript/Vue
- **规则配置**: .eslintrc.cjs
- **自动修复**: --fix 参数

```javascript
// .eslintrc.cjs
module.exports = {
  extends: ['plugin:vue/vue3-essential', 'eslint:recommended', '@vue/eslint-config-typescript', 'prettier'],
  rules: {
    'vue/multi-word-component-names': 'off'
  }
}
```

### Prettier 3.8.2

- **代码格式化**: 统一代码风格
- **配置文件**: .prettierrc.json
- **编辑器集成**: 保存时自动格式化

```json
// .prettierrc.json
{
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5"
}
```

### Stylelint 15.11.0

- **样式检查**: SCSS/CSS
- **规则配置**: .stylelintrc.json
- **自动修复**: --fix 参数

### Husky 8.0.3 + Lint-staged 15.5.2

- **Git Hooks**: 提交前检查
- **暂存区检查**: 只检查修改的文件

```json
// package.json
{
  "lint-staged": {
    "*.{js,ts,vue}": ["eslint --fix", "prettier --write"],
    "*.{scss,css}": ["stylelint --fix", "prettier --write"]
  }
}
```

## 开发工具

### Vue DevTools

- **组件树**: 查看组件层级
- **状态检查**: 查看 Pinia store
- **性能分析**: 组件渲染性能

### TypeScript Language Service

- **类型检查**: 实时类型错误提示
- **智能提示**: 代码补全
- **重构支持**: 重命名、提取等

## 浏览器 API

### Clipboard API

- **剪贴板操作**: 复制、粘贴
- **异步 API**: Promise based

### Fullscreen API

- **全屏模式**: 演示模式
- **跨浏览器**: 兼容性处理

### File API

- **文件读取**: FileReader
- **文件保存**: FileSaver

### Canvas API

- **图像处理**: 截图、滤镜
- **性能优化**: 离屏渲染

## 性能监控

### Performance API

- **性能指标**: FCP, LCP, CLS
- **资源加载**: 资源时间线

### Lighthouse

- **性能评分**: 综合性能评估
- **优化建议**: 具体优化建议

## 总结

PPTist 采用了现代化的前端技术栈：

- **框架**: Vue 3 + TypeScript
- **构建**: Vite
- **状态**: Pinia
- **样式**: SCSS
- **编辑器**: ProseMirror
- **可视化**: ECharts
- **存储**: IndexedDB
- **导出**: pptxgenjs + html-to-image
- **质量**: ESLint + Prettier + Stylelint

技术选型原则：

1. **现代化**: 使用最新的稳定版本
2. **类型安全**: TypeScript 全覆盖
3. **性能优先**: 轻量级、高性能
4. **开发体验**: 完善的工具链
5. **可维护性**: 清晰的架构设计
