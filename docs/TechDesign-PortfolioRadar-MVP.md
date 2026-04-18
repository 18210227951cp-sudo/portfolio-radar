# 第三步：技术设计文档 — PortfolioRadar MVP

## 3.1 技术栈选型（零环境依赖原则）

- 语言：纯 HTML5 + CSS3 + Vanilla JavaScript
- 图表库：Chart.js（通过 CDN 引入，无需 npm）
- 数据存储：localStorage（无需后端、无需数据库）
- 样式框架：Tailwind CSS（CDN 版）
- 字体：Google Fonts（CDN）
- 部署：GitHub Pages（静态托管）
- 约束：无需 Node.js、Python、任何后端服务器

## 3.2 文件结构（单文件方案，最简化）

### 运行时（MVP）
- index.html（唯一运行文件，包含 HTML + Tailwind + Chart.js + 业务 JS）

### 文档（非运行必需）
- docs/research-PortfolioRadar.md
- docs/PRD-PortfolioRadar-MVP.md
- docs/TechDesign-PortfolioRadar-MVP.md

## 3.3 数据模型设计

### 3.3.1 持仓数据结构（localStorage）

- Storage Key：portfolioRadar.v1
- 数据结构（JSON）：
  - holdings: Holding[]
  - meta: { version: 1, updatedAt: number }

Holding 字段（建议）：
- id: string（UUID 或时间戳+随机数）
- name: string
- code: string（可选）
- costPrice: number
- currentPrice: number
- quantity: number
- createdAt: number
- updatedAt: number

### 3.3.2 计算字段（运行时派生，不落库）
- costValue = costPrice * quantity
- marketValue = currentPrice * quantity
- pnl = marketValue - costValue
- pnlPct = costValue === 0 ? 0 : pnl / costValue

组合汇总：
- totalCost = Σ costValue
- totalMarket = Σ marketValue
- totalPnl = totalMarket - totalCost
- totalPnlPct = totalCost === 0 ? 0 : totalPnl / totalCost

## 3.4 页面结构与组件划分（单文件内模块化）

### 3.4.1 UI 区块（从上到下）
- 顶部汇总卡片（总成本/总市值/总盈亏/总收益率）
- 操作区（添加持仓按钮、排序下拉/按钮、清空数据等）
- 持仓表格（每行显示名称/代码/成本价/现价/数量/市值/盈亏/收益率/操作）
- 图表区（仓位饼图）
- 弹窗/抽屉（新增/编辑持仓表单）

### 3.4.2 JS 模块（函数式封装）

- state：
  - getState() / setState(next)
  - state.holdings（源数据）
- storage：
  - loadFromStorage()
  - saveToStorage(holdings)
  - clearStorage()
- compute：
  - computeHoldingDerived(holding)
  - computePortfolioSummary(holdings)
- format：
  - formatMoney(value)
  - formatPct(value)
- render：
  - renderSummary(summary)
  - renderTable(holdingsDerived)
  - renderChart(holdingsDerived)
- actions：
  - addHolding(payload)
  - updateHolding(id, payload)
  - deleteHolding(id)
  - sortHoldings(mode)
- ui：
  - openModal(mode, holding?)
  - closeModal()
  - bindEvents()

## 3.5 关键交互与数据流

### 3.5.1 初始化流程
1. DOMContentLoaded
2. loadFromStorage() → 得到 holdings
3. setState({ holdings })
4. compute summary/derived
5. renderSummary + renderTable + renderChart
6. bindEvents（表单提交、编辑/删除、排序、清空）

### 3.5.2 新增/编辑流程
1. 打开表单（新增/编辑模式）
2. 输入校验（必填、数字、非负/正数规则）
3. addHolding/updateHolding → 更新 state.holdings
4. saveToStorage
5. 重新计算 + 重渲染（表格、汇总、图表）

### 3.5.3 删除/清空流程
- 删除：
  - deleteHolding(id) → saveToStorage → re-render
- 清空：
  - 二次确认 → clearStorage + setState({ holdings: [] }) → re-render

## 3.6 图表实现（Chart.js）

- 图表类型：pie（饼图）
- 数据：
  - labels：持仓名称（必要时拼接代码）
  - data：marketValue
- 更新策略：
  - 初始化时创建 chart 实例
  - 每次数据变更后调用 chart.data = ... + chart.update()
- 空状态：
  - totalMarket 为 0 时展示占位数据或隐藏图表并显示提示文案（实现二选一）

## 3.7 样式与可用性规范

- 主题：深色背景，金融工具风格
- 颜色编码：
  - 盈利：#22c55e
  - 亏损：#ef4444
- 数字格式：
  - 金额：千分位 + 2 位小数（或按需要配置）
  - 百分比：2 位小数，显示 % 符号
- 响应式：
  - 表格在小屏可横向滚动，确保信息可读

## 3.8 性能与可靠性

- 性能：
  - MVP 数据量通常 < 200 行，直接全量重渲染可接受
  - Chart 更新使用实例复用，避免重复创建
- 可靠性：
  - 存储数据增加 version 字段，便于未来迁移
  - 计算字段不落库，避免历史口径变更造成脏数据

