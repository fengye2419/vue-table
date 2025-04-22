# Vue 表格组件

这是一个简单而功能强大的Vue表格组件，基于Vue 3和TypeScript实现，支持丰富的表格编辑和操作功能。

## 功能特点

- **右键菜单操作**：
  - 新增行/列
  - 删除行/列
  - 复制/粘贴单元格内容
  - 合并/取消合并单元格
- **支持直接编辑单元格内容**（双击单元格）
- **支持编辑表头**（双击表头）
- **灵活的选择操作**：
  - 点击单元格进行选择
  - 按住Shift可选择范围
  - Ctrl/Command+点击可多选

## 技术栈

- Vue 3
- TypeScript
- Ant Design Vue
- Vite

## 安装与运行

### 前置条件

- Node.js (>= 16.0.0)
- npm (>= 8.0.0)

### 安装依赖

```bash
npm install
```

### 开发环境运行

```bash
npm run dev
```

### 构建生产版本

```bash
npm run build
```

## 使用指南

### 基本操作

- **选择单元格**：
  - 单击选择一个单元格
  - 按住Shift可选择范围
  - 按住Ctrl/Command可多选单元格

- **右键菜单操作**：
  - 右键点击单元格，打开上下文菜单
  - 菜单中提供各种操作选项

- **编辑操作**：
  - 双击单元格可编辑内容
  - 双击表头可编辑列标题
  - 按Enter或点击其他区域完成编辑

- **单元格合并**：
  - 选中多个单元格
  - 右键菜单选择"合并单元格"

- **拆分单元格**：
  - 选中已合并的单元格
  - 右键菜单选择"拆分单元格"

### 组件结构

组件采用TypeScript强类型设计，主要包含以下数据结构：

```typescript
interface Column {
  id: string;
  title: string;
  field: string;
}

interface Row {
  id: string;
  [key: string]: any;
}

interface MergedCell {
  rowIndex: number;
  colIndex: number;
  rowspan: number;
  colspan: number;
}

interface TableData {
  columns: Column[];
  rows: Row[];
  mergedCells: MergedCell[];
}
```

## 项目结构

```
vue-table/
├── src/
│   ├── components/
│   │   └── VueTable.vue    # 表格组件
│   ├── App.vue             # 主应用
│   └── main.ts             # 入口文件
├── package.json
└── README.md
```

## 自定义和扩展

该组件可根据需求进一步扩展，例如：

- 添加数据导入/导出功能
- 实现表格数据的本地存储
- 添加行/列的拖拽排序功能
- 实现更丰富的单元格类型（如下拉选择、日期选择器等）
- 添加表格数据的过滤和排序功能
- 集成数据可视化功能

## 许可证

MIT
