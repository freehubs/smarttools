# SmartTools 开发指南

> 本文档面向 AI 大模型或开发者，提供创建新工具和修改现有工具所需的全部上下文。

## 项目概览

SmartTools 是一个纯静态小工具集合网站，每个工具是 **独立的单页 HTML**，托管在 GitHub Pages 上。

- 仓库地址：`https://github.com/freehubs/smarttools`
- 线上地址：`https://freehubs.github.io/smarttools/`
- 部署方式：push 到 `main` 分支 → GitHub Actions 自动部署
- 零构建依赖：无 Node.js、无打包工具，浏览器直接打开 HTML 即可开发

## 目录结构

```
smarttools/
├── index.html                     # 首页（工具卡片导航）
├── assets/style.css               # 共享样式（仅 index.html 和简单工具使用）
├── tools/
│   ├── json-formatter.html        # 简单工具（引用 assets/style.css）
│   ├── timezone-converter.html    # 简单工具（引用 assets/style.css）
│   └── sg-tax-calculator.html     # 复杂工具（自包含全部 CSS，★ 设计标杆）
├── .github/workflows/deploy.yml   # GitHub Pages 自动部署
└── README.md
```

## 设计标杆：sg-tax-calculator.html

**所有新工具必须以 `sg-tax-calculator.html` 为设计标杆。** 它代表了本项目追求的最终品质：功能完整、交互精致、信息密度高。

---

## 一、工程架构规则

### 1.1 每个工具 = 一个 HTML 文件

- 放置于 `tools/` 目录下，文件名用 **kebab-case**（如 `loan-calculator.html`）
- HTML + CSS + JS 全部内联在同一文件中，**不引用外部 JS 库**
- CSS 写在 `<style>` 标签中，JS 写在页面底部的 `<script>` 标签中
- 复杂工具（如 sg-tax-calculator）应自包含全部样式，不依赖 `assets/style.css`

### 1.2 添加新工具的步骤

1. 在 `tools/` 新建 HTML 文件
2. 在 `index.html` 的 `.tools-grid` 中添加一张卡片：

```html
<a href="tools/your-tool.html" class="tool-card">
  <div class="icon">🔧</div>
  <h3>工具名称</h3>
  <p>一句话描述工具功能。</p>
</a>
```

3. Push 到 main 即自动部署

---

## 二、视觉设计规范

### 2.1 CSS 变量体系

所有工具必须使用 CSS 变量管理颜色。以下是标准变量集（从 sg-tax-calculator 提取）：

```css
:root {
  --primary: #6366f1;        /* 主色调 Indigo */
  --primary-light: #818cf8;
  --primary-dark: #4f46e5;
  --surface: #ffffff;         /* 卡片/面板背景 */
  --surface-alt: #f8fafc;     /* 输入框/表头背景 */
  --border: #e2e8f0;
  --text: #1e293b;
  --text-secondary: #64748b;
  --text-tertiary: #94a3b8;
  --success: #10b981;
  --warning: #f59e0b;
  --danger: #ef4444;
  --radius: 12px;
  --shadow: 0 1px 3px rgba(0,0,0,.06), 0 4px 12px rgba(0,0,0,.04);
}
```

### 2.2 整体布局

- **背景**：渐变色 `linear-gradient(135deg, #eef2ff 0%, #e0e7ff 50%, #ede9fe 100%)`
- **容器**：`max-width: 960px; margin: 0 auto; padding: 32px 20px 64px;`
- **字体**：`-apple-system, BlinkMacSystemFont, 'Segoe UI', Inter, Roboto, sans-serif`
- **圆角**：统一 12px（卡片）、8-10px（输入框/按钮）

### 2.3 响应式断点

- `640px`：输入网格单列、摘要卡片 2 列
- 所有布局使用 CSS Grid，手机上自动降级为单列

---

## 三、UI 组件模式

以下是本项目中已建立的 UI 模式，新工具应复用这些模式而非发明新的。

### 3.1 顶部导航栏

每个工具页面顶部必须有导航栏，链接回首页：

```html
<div style="background:var(--surface);border-bottom:1px solid var(--border);
  padding:10px 20px;position:sticky;top:0;z-index:100;backdrop-filter:blur(10px);">
  <div style="max-width:960px;margin:0 auto;display:flex;align-items:center;gap:12px;">
    <a href="../index.html" style="font-size:1.15rem;font-weight:700;color:var(--text);
      text-decoration:none;">&#9881; SmartTools</a>
    <a href="../index.html" style="font-size:0.9rem;color:var(--text-secondary);
      text-decoration:none;">&larr; All Tools</a>
  </div>
</div>
```

### 3.2 页面标题区

```html
<header>
  <h1>🇸🇬 工具标题</h1>
  <p>工具简介 — 一行说明</p>
</header>
```

### 3.3 卡片（.card）

内容的主要容器，白色背景 + 阴影 + 圆角：

```css
.card {
  background: var(--surface);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  padding: 24px 28px;
  margin-bottom: 20px;
}
```

卡片标题带图标：

```html
<div class="card-title">
  <span class="icon" style="background:#eef2ff;color:var(--primary)">💰</span>
  标题文字
</div>
```

### 3.4 输入控件

标准的表单字段结构：

```html
<div class="field">
  <label>字段名称</label>
  <div class="input-wrapper">
    <span class="prefix">SGD</span>  <!-- 可选前缀 -->
    <input id="xxx" type="text" inputmode="numeric" placeholder="0" class="num-input">
  </div>
  <p class="hint">辅助说明文字</p>  <!-- 可选 -->
</div>
```

- 数字输入用 `type="text" inputmode="numeric"` + JS 格式化（千分位），不要用 `type="number"`
- 网格布局用 `.input-grid`（2列）或 `.input-grid.col3`（3列）

### 3.5 可折叠分组（Collapsible Section）

适用于参数较多需要分组的场景：

```html
<div class="relief-section">
  <div class="relief-header" onclick="toggleSection(this)">
    <span class="arrow">▶</span>
    <span class="sec-title">分组标题</span>
    <span class="sec-amount" id="sec_xxx_amt">$0</span>  <!-- 右侧实时汇总 -->
  </div>
  <div class="relief-body">
    <!-- 展开后的内容 -->
  </div>
</div>
```

JS 切换函数：

```js
function toggleSection(header) {
  const isOpen = header.classList.toggle('open');
  header.nextElementSibling.classList.toggle('open', isOpen);
}
```

### 3.6 开关（Toggle Switch）

```html
<div class="toggle-row">
  <label class="toggle">
    <input type="checkbox" id="xxx" checked>
    <span class="slider"></span>
  </label>
  开关标签文字
</div>
```

### 3.7 摘要卡片网格（Summary Grid）

计算结果顶部的 4 格摘要，用 JS 动态渲染：

```js
const items = [
  { label: '总收入', value: '$120,000', sub: '补充说明' },
  { label: '应缴税额', value: '$5,350', sub: '有效税率 4.46%', hl: true },  // hl=高亮
  // ...
];
```

- 高亮卡片使用 `stat-card.highlight`（紫色渐变背景 + 白色文字）

### 3.8 数据表格

```css
.bracket-table th { text-transform: uppercase; letter-spacing: .5px; font-size: 11px; }
.bracket-table tr.active-bracket td { background: #eef2ff; font-weight: 600; }
.bracket-table tr.total-row td { border-top: 2px solid var(--primary); font-weight: 700; }
```

- 数字列右对齐
- 活跃行高亮、非活跃行降低透明度 `opacity:.4`
- 总计行粗体 + 顶部加粗分隔线
- 可加进度条列（`.bar-bg` + `.bar-fill`）

### 3.9 环形图（Donut Chart）

纯 SVG 实现，无需外部库：

```js
function renderDonut(segments) {
  // segments: [{ label, pct, color }]
  // 使用 stroke-dasharray + stroke-dashoffset 实现弧形
  // 中央显示关键百分比
}
```

### 3.10 主按钮

```html
<button class="btn btn-primary" onclick="calculate()">计算 Calculate</button>
```

- 全宽、居中、大号
- `transform: scale(.97)` 按压反馈

### 3.11 结果区域动画

计算结果默认隐藏，触发后带 `fadeUp` 动画出现，并自动滚动到视窗内：

```css
.results { display: none; }
.results.show { display: block; animation: fadeUp .4s ease; }

@keyframes fadeUp {
  from { opacity: 0; transform: translateY(12px); }
  to { opacity: 1; transform: translateY(0); }
}
```

```js
document.getElementById('results').classList.add('show');
document.getElementById('results').scrollIntoView({ behavior: 'smooth', block: 'start' });
```

---

## 四、JavaScript 编码规范

### 4.1 标准辅助函数

每个工具都应包含这些常用函数：

```js
function parseNum(id) {
  return parseFloat(document.getElementById(id).value.replace(/[^0-9.]/g, '')) || 0;
}
function selVal(id) { return document.getElementById(id).value; }
function selNum(id) { return parseFloat(selVal(id)) || 0; }
function fmt(n) { return n.toLocaleString('en-US', { maximumFractionDigits: 0 }); }
function fmtDec(n) { return n.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 }); }
```

### 4.2 数字输入自动格式化

所有金额输入框加 `.num-input` class，并统一注册千分位格式化：

```js
document.querySelectorAll('.num-input').forEach(input => {
  input.addEventListener('input', function () {
    const pos = this.selectionStart;
    const raw = this.value.replace(/[^0-9]/g, '');
    const before = this.value.length;
    this.value = raw ? parseInt(raw, 10).toLocaleString('en-US') : '';
    const diff = this.value.length - before;
    this.setSelectionRange(pos + diff, pos + diff);
  });
});
```

### 4.3 架构模式

采用 **compute → render 分离** 模式：

1. **compute 函数**：纯计算，返回数据对象（如 `computeReliefs()`）
2. **render 函数**：接收数据对象，生成 HTML 并更新 DOM（如 `renderBracketTable()`）
3. **主函数**：串联 compute 和 render（如 `calculate()`）

### 4.4 实时更新

输入值变化时实时更新汇总显示（不需要点按钮），使用 `onchange` 或 `oninput` 事件：

```html
<select id="xxx" onchange="updateTotals()">
```

键盘 Enter 触发主计算：

```js
document.addEventListener('keydown', e => { if (e.key === 'Enter') calculate(); });
```

---

## 五、内容与语言规范

### 5.1 双语标签

所有面向用户的标签采用 **中文 + 英文** 格式：

```
收入信息 Income
年薪 Annual Salary
税收减免明细 Tax Reliefs
计算税额 Calculate Tax
```

### 5.2 页面 lang 属性

双语工具用 `<html lang="zh-CN">`，纯英文工具用 `<html lang="en">`。

### 5.3 提示文字（Hint）

在输入框下方用 `.hint`（11px 灰色）提供限额、规则等辅助说明：

```html
<p class="hint">公民/PR 上限 $15,300 · 外籍上限 $35,700</p>
```

用中间点 `·` 分隔多条提示。

### 5.4 Footer

```html
<footer>
  基于 XX 数据 · 本工具仅供参考<br>
  SmartTools — <a href="../index.html">Home</a>
</footer>
```

---

## 六、功能深度要求

新工具不是写一个最简版本就够了，而是要追求 **"比用户预期多走一步"** 的完整度：

| 维度 | 要求 | 示例 |
|------|------|------|
| 输入完整性 | 覆盖该领域所有主要参数 | 税计算器包含 14 项减免，不是只输入收入 |
| 智能默认值 | 能自动推算的字段自动计算 | CPF 根据收入自动估算，可手动覆盖 |
| 结果丰富度 | 提供摘要 + 明细 + 可视化 | 摘要卡片 + 阶梯表 + 环形图 |
| 边界处理 | 处理上限/下限/特殊场景 | 减免 $80K 上限、Non-Resident 切换 |
| 信息透明 | 显示计算过程而非只显示结果 | 每个税率档位的应税额和税额都列出 |
| 交互细节 | 实时反馈、动画、格式化 | 千分位、结果渐入、滚动到结果区 |

---

## 七、质量检查清单

在提交新工具前，确认以下所有项：

- [ ] 文件在 `tools/` 目录下，kebab-case 命名
- [ ] 顶部有 SmartTools 导航栏和返回首页链接
- [ ] 底部有 footer
- [ ] `index.html` 已添加对应工具卡片
- [ ] 手机端（640px 以下）布局正常
- [ ] CSS 变量体系与标杆一致
- [ ] 数字输入有千分位格式化
- [ ] 计算结果带 fadeUp 动画
- [ ] 无外部 JS/CSS 依赖
- [ ] 浏览器直接打开 HTML 可正常运行
