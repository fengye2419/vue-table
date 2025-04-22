<template>
  <div class="vue-table-container">
    <div class="table-wrapper">
      <table class="vue-table">
        <thead>
          <tr>
            <th v-if="showRowNumbers" class="corner-cell"></th>
            <th
              v-for="(col, colIndex) in tableData.columns"
              :key="col.id"
              :class="{ 
                selected: selectedColumns.includes(colIndex),
                'header-editing': isEditingHeader(colIndex)
              }"
              @click="toggleColumnSelection(colIndex, $event)"
              @dblclick.stop="startHeaderEditing(colIndex)"
              @contextmenu.prevent="showHeaderContextMenu($event, colIndex)"
            >
              <template v-if="isEditingHeader(colIndex)">
                <a-input
                  v-model:value="editingHeaderValue"
                  @blur="saveHeader"
                  @pressEnter="saveHeader"
                  ref="headerInput"
                  size="small"
                />
              </template>
              <template v-else>
                {{ col.title }}
              </template>
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, rowIndex) in tableData.rows" :key="row.id" :class="{ 'row-even': rowIndex % 2 === 0 }">
            <td
              v-if="showRowNumbers"
              class="row-header"
              :class="{ selected: selectedRows.includes(rowIndex) }"
              @click="toggleRowSelection(rowIndex, $event)"
            >
              {{ rowIndex + 1 }}
            </td>
            <td
              v-for="cell in getVisibleCells(rowIndex)"
              :key="cell.key"
              :rowspan="cell.rowspan"
              :colspan="cell.colspan"
              :class="{
                'cell-selected': isCellSelected(cell.rowIndex, cell.colIndex),
                'cell-editing': isEditingCell(cell.rowIndex, cell.colIndex),
              }"
              @click="handleCellClick(cell.rowIndex, cell.colIndex, $event)"
              @contextmenu.prevent="showContextMenu($event, cell.rowIndex, cell.colIndex)"
            >
              <template v-if="isEditingCell(cell.rowIndex, cell.colIndex)">
                <a-input
                  v-model:value="editingCellValue"
                  @blur="saveCell"
                  @pressEnter="saveCell"
                  ref="cellInput"
                  size="small"
                />
              </template>
              <template v-else>
                {{ cell.value }}
              </template>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    
    <!-- 右键菜单 -->
    <a-dropdown :open="contextMenuVisible" :trigger="['contextmenu']">
      <template #overlay>
        <a-menu v-if="contextMenuType === 'cell'" class="context-menu">
          <a-menu-item key="edit" @click="handleContextMenuAction('edit')">
            <span>编辑单元格</span>
          </a-menu-item>
          <a-menu-divider />
          <a-menu-item key="insertRowAbove" @click="handleContextMenuAction('insertRowAbove')">
            <span>在上方插入行</span>
          </a-menu-item>
          <a-menu-item key="insertRowBelow" @click="handleContextMenuAction('insertRowBelow')">
            <span>在下方插入行</span>
          </a-menu-item>
          <a-menu-item key="insertColLeft" @click="handleContextMenuAction('insertColLeft')">
            <span>在左侧插入列</span>
          </a-menu-item>
          <a-menu-item key="insertColRight" @click="handleContextMenuAction('insertColRight')">
            <span>在右侧插入列</span>
          </a-menu-item>
          <a-menu-divider />
          <a-menu-item key="mergeCell" :disabled="!canMergeCells" @click="handleContextMenuAction('merge')">
            <span>合并单元格</span>
          </a-menu-item>
          <a-menu-item key="splitCell" :disabled="!canSplitCell" @click="handleContextMenuAction('split')">
            <span>拆分单元格</span>
          </a-menu-item>
          <a-menu-divider />
          <a-menu-item key="removeRow" :disabled="!hasSelectedRows" @click="handleContextMenuAction('removeRow')">
            <span>删除选中行</span>
          </a-menu-item>
          <a-menu-item key="removeColumn" :disabled="!hasSelectedColumns" @click="handleContextMenuAction('removeColumn')">
            <span>删除选中列</span>
          </a-menu-item>
        </a-menu>
        <a-menu v-else class="context-menu">
          <a-menu-item key="editHeader" @click="handleContextMenuAction('edit')">
            <span>编辑列标题</span>
          </a-menu-item>
          <a-menu-divider />
          <a-menu-item key="insertColLeft" @click="handleContextMenuAction('insertColLeft')">
            <span>在左侧插入列</span>
          </a-menu-item>
          <a-menu-item key="insertColRight" @click="handleContextMenuAction('insertColRight')">
            <span>在右侧插入列</span>
          </a-menu-item>
          <a-menu-divider />
          <a-menu-item key="removeColumn" :disabled="!hasSelectedColumns" @click="handleContextMenuAction('removeColumn')">
            <span>删除选中列</span>
          </a-menu-item>
        </a-menu>
      </template>
      <div class="context-menu-trigger" :style="contextMenuStyle"></div>
    </a-dropdown>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, nextTick } from "vue";
import { message } from "ant-design-vue";

// 定义表格数据结构
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

// 使用defineOptions设置组件名称
defineOptions({
  name: "VueTable",
});

// 初始化表格数据
const tableData = ref<TableData>({
  columns: [
    { id: "col1", title: "列1", field: "field1" },
    { id: "col2", title: "列2", field: "field2" },
    { id: "col3", title: "列3", field: "field3" },
  ],
  rows: [
    { id: "row1", field1: "数据1-1", field2: "数据1-2", field3: "数据1-3" },
    { id: "row2", field1: "数据2-1", field2: "数据2-2", field3: "数据2-3" },
    { id: "row3", field1: "数据3-1", field2: "数据3-2", field3: "数据3-3" },
  ],
  mergedCells: [],
});

// 定义选中状态
const selectedRows = ref<number[]>([]);
const selectedColumns = ref<number[]>([]);
const selectedCells = ref<{ rowIndex: number; colIndex: number }[]>([]);
const editingCell = ref<{ rowIndex: number; colIndex: number } | null>(null);
const editingCellValue = ref<string>("");

// 表头编辑状态
const editingHeader = ref<number | null>(null);
const editingHeaderValue = ref<string>("");

// 序号列显示状态
const showRowNumbers = ref(true);

// 计算属性
const hasSelectedRows = computed(() => selectedRows.value.length > 0);
const hasSelectedColumns = computed(() => selectedColumns.value.length > 0);
const hasSelectedCells = computed(() => selectedCells.value.length > 0);
const canMergeCells = computed(() => selectedCells.value.length > 1);
const canSplitCell = computed(() => {
  if (selectedCells.value.length !== 1) return false;
  const cell = selectedCells.value[0];
  return isCellMerged(cell.rowIndex, cell.colIndex);
});

// 切换序号列显示状态
const toggleRowNumbers = () => {
  showRowNumbers.value = !showRowNumbers.value;
  message.success(showRowNumbers.value ? '显示序号列' : '隐藏序号列');
};

// 表头编辑相关函数
const isEditingHeader = (colIndex: number) => {
  return editingHeader.value === colIndex;
};

const startHeaderEditing = (colIndex: number) => {
  editingHeader.value = colIndex;
  editingHeaderValue.value = tableData.value.columns[colIndex].title;
  
  // 聚焦输入框
  nextTick(() => {
    const input = document.querySelector(".header-editing input") as HTMLInputElement;
    if (input) {
      input.focus();
      input.select();
      // 确保输入框尺寸适应单元格
      const headerCell = document.querySelector(".header-editing") as HTMLElement;
      if (headerCell) {
        const width = headerCell.offsetWidth;
        const height = headerCell.offsetHeight;
        input.style.width = `${width}px`;
        input.style.height = `${height}px`;
      }
    }
  });
};

const saveHeader = () => {
  if (editingHeader.value === null) return;
  
  tableData.value.columns[editingHeader.value].title = editingHeaderValue.value;
  editingHeader.value = null;
};

// 获取某一行中可见的单元格
const getVisibleCells = (rowIndex: number) => {
  const cells: Array<{
    rowIndex: number;
    colIndex: number;
    key: string;
    value: string;
    rowspan: number;
    colspan: number;
  }> = [];

  tableData.value.columns.forEach((col, colIndex) => {
    if (shouldRenderCell(rowIndex, colIndex)) {
      cells.push({
        rowIndex,
        colIndex,
        key: `cell-${rowIndex}-${colIndex}`,
        value: getCellValue(rowIndex, colIndex),
        rowspan: getCellRowspan(rowIndex, colIndex),
        colspan: getCellColspan(rowIndex, colIndex),
      });
    }
  });

  return cells;
};

// 行操作
const addRow = () => {
  const newRowId = `row${tableData.value.rows.length + 1}`;
  const newRow: Row = { id: newRowId };

  tableData.value.columns.forEach((col) => {
    newRow[col.field] = "";
  });

  tableData.value.rows.push(newRow);
  message.success("添加行成功");
};

const removeSelectedRows = () => {
  if (selectedRows.value.length === 0) return;

  // 从大到小排序，以便正确删除
  const sortedRows = [...selectedRows.value].sort((a, b) => b - a);

  // 检查是否有合并单元格跨越这些行
  const invalidMerges = tableData.value.mergedCells.filter((cell) => {
    return sortedRows.some(
      (rowIndex) =>
        rowIndex >= cell.rowIndex && rowIndex < cell.rowIndex + cell.rowspan
    );
  });

  if (invalidMerges.length > 0) {
    message.error("无法删除包含合并单元格的行，请先拆分相关单元格");
    return;
  }

  // 删除行
  sortedRows.forEach((rowIndex) => {
    tableData.value.rows.splice(rowIndex, 1);

    // 更新合并单元格信息
    tableData.value.mergedCells = tableData.value.mergedCells.map((cell) => {
      if (cell.rowIndex > rowIndex) {
        return { ...cell, rowIndex: cell.rowIndex - 1 };
      }
      return cell;
    });
  });

  // 清空选择
  selectedRows.value = [];
  selectedCells.value = [];
  message.success("删除行成功");
};

const toggleRowSelection = (rowIndex: number, event: MouseEvent) => {
  if (event.ctrlKey || event.metaKey) {
    const index = selectedRows.value.indexOf(rowIndex);
    if (index >= 0) {
      selectedRows.value.splice(index, 1);
    } else {
      selectedRows.value.push(rowIndex);
    }
  } else {
    if (selectedRows.value.length === 1 && selectedRows.value[0] === rowIndex) {
      selectedRows.value = [];
    } else {
      selectedRows.value = [rowIndex];
    }
  }
  
  // 不清除单元格选择，只清除列选择
  if (!event.shiftKey) {
    selectedColumns.value = [];
  }
};

// 列操作
const addColumn = () => {
  const newColId = `col${tableData.value.columns.length + 1}`;
  const newField = `field${tableData.value.columns.length + 1}`;

  tableData.value.columns.push({
    id: newColId,
    title: `列${tableData.value.columns.length + 1}`,
    field: newField,
  });

  // 为每一行添加新列的数据
  tableData.value.rows.forEach((row) => {
    row[newField] = "";
  });

  message.success("添加列成功");
};

const removeSelectedColumns = () => {
  if (selectedColumns.value.length === 0) return;

  // 从大到小排序，以便正确删除
  const sortedColumns = [...selectedColumns.value].sort((a, b) => b - a);

  // 检查是否有合并单元格跨越这些列
  const invalidMerges = tableData.value.mergedCells.filter((cell) => {
    return sortedColumns.some(
      (colIndex) =>
        colIndex >= cell.colIndex && colIndex < cell.colIndex + cell.colspan
    );
  });

  if (invalidMerges.length > 0) {
    message.error("无法删除包含合并单元格的列，请先拆分相关单元格");
    return;
  }

  // 删除列
  sortedColumns.forEach((colIndex) => {
    const field = tableData.value.columns[colIndex].field;
    tableData.value.columns.splice(colIndex, 1);

    // 删除每行中的对应字段
    tableData.value.rows.forEach((row) => {
      delete row[field];
    });

    // 更新合并单元格信息
    tableData.value.mergedCells = tableData.value.mergedCells.map((cell) => {
      if (cell.colIndex > colIndex) {
        return { ...cell, colIndex: cell.colIndex - 1 };
      }
      return cell;
    });
  });

  // 清空选择
  selectedColumns.value = [];
  selectedCells.value = [];
  message.success("删除列成功");
};

const toggleColumnSelection = (colIndex: number, event: MouseEvent) => {
  if (event.ctrlKey || event.metaKey) {
    const index = selectedColumns.value.indexOf(colIndex);
    if (index >= 0) {
      selectedColumns.value.splice(index, 1);
    } else {
      selectedColumns.value.push(colIndex);
    }
  } else {
    if (
      selectedColumns.value.length === 1 &&
      selectedColumns.value[0] === colIndex
    ) {
      selectedColumns.value = [];
    } else {
      selectedColumns.value = [colIndex];
    }
  }
  
  // 不清除单元格选择，只清除行选择
  if (!event.shiftKey) {
    selectedRows.value = [];
  }
};

// 单元格操作
const handleCellClick = (
  rowIndex: number,
  colIndex: number,
  event: MouseEvent
) => {
  // 如果是双击，进入编辑模式
  if (event.detail === 2) {
    startEditing(rowIndex, colIndex);
    return;
  }

  // 单击处理选择
  if (event.ctrlKey || event.metaKey) {
    const cellIndex = selectedCells.value.findIndex(
      (cell) => cell.rowIndex === rowIndex && cell.colIndex === colIndex
    );

    if (cellIndex >= 0) {
      selectedCells.value.splice(cellIndex, 1);
    } else {
      selectedCells.value.push({ rowIndex, colIndex });
    }
  } else {
    const isSameCell =
      selectedCells.value.length === 1 &&
      selectedCells.value[0].rowIndex === rowIndex &&
      selectedCells.value[0].colIndex === colIndex;

    if (isSameCell) {
      selectedCells.value = [];
    } else {
      selectedCells.value = [{ rowIndex, colIndex }];
      
      // 当选中单个单元格时，同时选中对应的行和列
      if (!event.shiftKey) {
        selectedRows.value = [rowIndex];
        selectedColumns.value = [colIndex];
      }
    }
  }
};

const startEditing = (rowIndex: number, colIndex: number) => {
  const field = tableData.value.columns[colIndex].field;
  const value = tableData.value.rows[rowIndex][field];

  editingCell.value = { rowIndex, colIndex };
  editingCellValue.value = value || "";

  // 在下一个tick中聚焦输入框
  nextTick(() => {
    const input = document.querySelector(
      ".cell-editing input"
    ) as HTMLInputElement;
    if (input) {
      input.focus();
      input.select();
      // 确保输入框尺寸适应单元格
      const cell = document.querySelector(".cell-editing") as HTMLElement;
      if (cell) {
        const width = cell.offsetWidth;
        const height = cell.offsetHeight;
        input.style.width = `${width}px`;
        input.style.height = `${height}px`;
      }
    }
  });
};

const saveCell = () => {
  if (!editingCell.value) return;

  const { rowIndex, colIndex } = editingCell.value;
  const field = tableData.value.columns[colIndex].field;

  tableData.value.rows[rowIndex][field] = editingCellValue.value;
  editingCell.value = null;
};

const isEditingCell = (rowIndex: number, colIndex: number) => {
  return (
    editingCell.value &&
    editingCell.value.rowIndex === rowIndex &&
    editingCell.value.colIndex === colIndex
  );
};

const isCellSelected = (rowIndex: number, colIndex: number) => {
  return selectedCells.value.some(
    (cell) => cell.rowIndex === rowIndex && cell.colIndex === colIndex
  );
};

// 单元格合并与拆分
const mergeCells = () => {
  if (selectedCells.value.length < 2) {
    message.warning("请至少选择两个单元格进行合并");
    return;
  }

  // 验证选中的单元格是否可以形成矩形区域
  const rows = selectedCells.value.map((cell) => cell.rowIndex);
  const cols = selectedCells.value.map((cell) => cell.colIndex);
  const minRow = Math.min(...rows);
  const maxRow = Math.max(...rows);
  const minCol = Math.min(...cols);
  const maxCol = Math.max(...cols);

  const isRectangular =
    selectedCells.value.length ===
    (maxRow - minRow + 1) * (maxCol - minCol + 1);
  if (!isRectangular) {
    message.error("只能合并形成矩形的单元格区域");
    return;
  }

  // 检查选中区域内是否有已合并的单元格
  const hasMergedCells = tableData.value.mergedCells.some(
    (cell) =>
      cell.rowIndex >= minRow &&
      cell.rowIndex <= maxRow &&
      cell.colIndex >= minCol &&
      cell.colIndex <= maxCol
  );

  if (hasMergedCells) {
    message.error("选中区域内已有合并单元格，请先拆分");
    return;
  }

  // 合并单元格
  tableData.value.mergedCells.push({
    rowIndex: minRow,
    colIndex: minCol,
    rowspan: maxRow - minRow + 1,
    colspan: maxCol - minCol + 1,
  });

  selectedCells.value = [];
  message.success("单元格合并成功");
};

const splitCell = () => {
  if (selectedCells.value.length !== 1) {
    message.warning("请选择一个已合并的单元格进行拆分");
    return;
  }

  const selectedCell = selectedCells.value[0];
  const mergedCellIndex = tableData.value.mergedCells.findIndex(
    (cell) =>
      cell.rowIndex === selectedCell.rowIndex &&
      cell.colIndex === selectedCell.colIndex
  );

  if (mergedCellIndex === -1) {
    message.warning("选中的单元格未被合并");
    return;
  }

  // 拆分单元格
  tableData.value.mergedCells.splice(mergedCellIndex, 1);

  selectedCells.value = [];
  message.success("单元格拆分成功");
};

// 辅助函数
const getCellRowspan = (rowIndex: number, colIndex: number) => {
  const mergedCell = tableData.value.mergedCells.find(
    (cell) => cell.rowIndex === rowIndex && cell.colIndex === colIndex
  );
  return mergedCell ? mergedCell.rowspan : 1;
};

const getCellColspan = (rowIndex: number, colIndex: number) => {
  const mergedCell = tableData.value.mergedCells.find(
    (cell) => cell.rowIndex === rowIndex && cell.colIndex === colIndex
  );
  return mergedCell ? mergedCell.colspan : 1;
};

const shouldRenderCell = (rowIndex: number, colIndex: number) => {
  // 检查该单元格是否是被其他合并单元格覆盖
  const isHidden = tableData.value.mergedCells.some((cell) => {
    // 如果当前单元格是合并单元格的左上角，应该渲染
    if (cell.rowIndex === rowIndex && cell.colIndex === colIndex) {
      return false;
    }

    // 检查当前单元格是否在某个合并单元格范围内
    return (
      rowIndex >= cell.rowIndex &&
      rowIndex < cell.rowIndex + cell.rowspan &&
      colIndex >= cell.colIndex &&
      colIndex < cell.colIndex + cell.colspan
    );
  });

  return !isHidden;
};

const getCellValue = (rowIndex: number, colIndex: number) => {
  const field = tableData.value.columns[colIndex].field;
  return tableData.value.rows[rowIndex][field] || "";
};

const isCellMerged = (rowIndex: number, colIndex: number) => {
  return tableData.value.mergedCells.some(
    (cell) => cell.rowIndex === rowIndex && cell.colIndex === colIndex
  );
};

// 右键菜单相关状态
const contextMenuVisible = ref(false);
const contextMenuStyle = ref({
  position: 'fixed',
  top: '0px',
  left: '0px',
});
const contextMenuType = ref<'cell' | 'header'>('cell');
const contextMenuColIndex = ref<number>(-1);

// 显示单元格右键菜单
const showContextMenu = (event: MouseEvent, rowIndex: number, colIndex: number) => {
  // 先选中单元格
  if (!isCellSelected(rowIndex, colIndex)) {
    handleCellClick(rowIndex, colIndex, event);
  }
  
  // 设置菜单位置和类型
  contextMenuStyle.value = {
    position: 'fixed',
    top: `${event.clientY}px`,
    left: `${event.clientX}px`,
  };
  contextMenuType.value = 'cell';
  
  // 显示菜单
  contextMenuVisible.value = true;
  
  // 点击其他位置关闭菜单
  setTimeout(() => {
    window.addEventListener('click', closeContextMenu);
  }, 0);
};

// 显示表头右键菜单
const showHeaderContextMenu = (event: MouseEvent, colIndex: number) => {
  // 先选中列，但保留单元格选择
  const originalCells = [...selectedCells.value];
  toggleColumnSelection(colIndex, event);
  selectedCells.value = originalCells;
  
  // 设置菜单位置和类型
  contextMenuStyle.value = {
    position: 'fixed',
    top: `${event.clientY}px`,
    left: `${event.clientX}px`,
  };
  contextMenuType.value = 'header';
  contextMenuColIndex.value = colIndex;
  
  // 显示菜单
  contextMenuVisible.value = true;
  
  // 点击其他位置关闭菜单
  setTimeout(() => {
    window.addEventListener('click', closeContextMenu);
  }, 0);
  
  event.preventDefault();
};

// 关闭右键菜单
const closeContextMenu = () => {
  contextMenuVisible.value = false;
  window.removeEventListener('click', closeContextMenu);
};

// 处理右键菜单操作
const handleContextMenuAction = (action: string) => {
  if (contextMenuType.value === 'cell') {
    switch (action) {
      case 'edit':
        if (selectedCells.value.length === 1) {
          const { rowIndex, colIndex } = selectedCells.value[0];
          startEditing(rowIndex, colIndex);
        }
        break;
      case 'merge':
        mergeCells();
        break;
      case 'split':
        splitCell();
        break;
      case 'removeRow':
        removeSelectedRows();
        break;
      case 'removeColumn':
        removeSelectedColumns();
        break;
      case 'insertRowAbove':
        insertRowAbove();
        break;
      case 'insertRowBelow':
        insertRowBelow();
        break;
      case 'insertColLeft':
        insertColLeft();
        break;
      case 'insertColRight':
        insertColRight();
        break;
    }
  } else if (contextMenuType.value === 'header') {
    switch (action) {
      case 'edit':
        startHeaderEditing(contextMenuColIndex.value);
        break;
      case 'removeColumn':
        removeSelectedColumns();
        break;
      case 'insertColLeft':
        insertColLeft();
        break;
      case 'insertColRight':
        insertColRight();
        break;
    }
  }
  closeContextMenu();
};

const insertRowAbove = () => {
  if (selectedRows.value.length === 0) return;

  const rowIndex = selectedRows.value[0];
  const newRowId = `row_${Date.now()}_${Math.floor(Math.random() * 1000)}`;
  const newRow: Row = { id: newRowId };

  tableData.value.columns.forEach((col) => {
    newRow[col.field] = "";
  });

  tableData.value.rows.splice(rowIndex, 0, newRow);
  
  // 更新合并单元格信息
  tableData.value.mergedCells = tableData.value.mergedCells.map((cell) => {
    if (cell.rowIndex >= rowIndex) {
      return { ...cell, rowIndex: cell.rowIndex + 1 };
    }
    return cell;
  });

  message.success("在上方插入行成功");
};

const insertRowBelow = () => {
  if (selectedRows.value.length === 0) return;

  const rowIndex = selectedRows.value[selectedRows.value.length - 1];
  const newRowId = `row_${Date.now()}_${Math.floor(Math.random() * 1000)}`;
  const newRow: Row = { id: newRowId };

  tableData.value.columns.forEach((col) => {
    newRow[col.field] = "";
  });

  tableData.value.rows.splice(rowIndex + 1, 0, newRow);
  
  // 更新合并单元格信息
  tableData.value.mergedCells = tableData.value.mergedCells.map((cell) => {
    if (cell.rowIndex > rowIndex) {
      return { ...cell, rowIndex: cell.rowIndex + 1 };
    }
    return cell;
  });

  message.success("在下方插入行成功");
};

const insertColLeft = () => {
  if (selectedColumns.value.length === 0) return;

  const colIndex = selectedColumns.value[0];
  const newColId = `col_${Date.now()}_${Math.floor(Math.random() * 1000)}`;
  const newField = `field_${Date.now()}_${Math.floor(Math.random() * 1000)}`;

  tableData.value.columns.splice(colIndex, 0, {
    id: newColId,
    title: `新列`,
    field: newField,
  });

  // 为每一行添加新列的数据
  tableData.value.rows.forEach((row) => {
    row[newField] = "";
  });
  
  // 更新合并单元格信息
  tableData.value.mergedCells = tableData.value.mergedCells.map((cell) => {
    if (cell.colIndex >= colIndex) {
      return { ...cell, colIndex: cell.colIndex + 1 };
    }
    return cell;
  });

  message.success("在左侧插入列成功");
};

const insertColRight = () => {
  if (selectedColumns.value.length === 0) return;

  const colIndex = selectedColumns.value[selectedColumns.value.length - 1];
  const newColId = `col_${Date.now()}_${Math.floor(Math.random() * 1000)}`;
  const newField = `field_${Date.now()}_${Math.floor(Math.random() * 1000)}`;

  tableData.value.columns.splice(colIndex + 1, 0, {
    id: newColId,
    title: `新列`,
    field: newField,
  });

  // 为每一行添加新列的数据
  tableData.value.rows.forEach((row) => {
    row[newField] = "";
  });
  
  // 更新合并单元格信息
  tableData.value.mergedCells = tableData.value.mergedCells.map((cell) => {
    if (cell.colIndex > colIndex) {
      return { ...cell, colIndex: cell.colIndex + 1 };
    }
    return cell;
  });

  message.success("在右侧插入列成功");
};
</script>

<style scoped>
.vue-table-container {
  margin: 20px;
  display: flex;
  flex-direction: column;
  gap: 15px;
  position: relative;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  background-color: #fff;
  padding: 16px;
}

.table-wrapper {
  overflow-x: auto;
  position: relative;
  border-radius: 6px;
  border: 1px solid #e8eaec;
}

.vue-table {
  border-collapse: collapse;
  width: 100%;
  border: none;
  font-size: 14px;
}

.vue-table th,
.vue-table td {
  border: 1px solid #e8eaec;
  padding: 10px 12px;
  text-align: left;
  min-width: 120px;
  height: 46px;
  position: relative;
  vertical-align: middle;
  white-space: pre-wrap;
  word-break: break-word;
  transition: all 0.3s ease;
}

.vue-table th {
  background-color: #f5f7fa;
  font-weight: 600;
  cursor: pointer;
  user-select: none;
  color: #515a6e;
  border-bottom: 2px solid #dcdfe6;
}

.vue-table th:hover {
  background-color: #e6f7ff;
  color: #1890ff;
}

.vue-table tbody tr {
  transition: background-color 0.3s ease;
}

.vue-table tbody tr:hover {
  background-color: #f0f9ff;
}

.vue-table tbody tr.row-even {
  background-color: #fafafa;
}

.vue-table tbody tr.row-even:hover {
  background-color: #f0f9ff;
}

.corner-cell {
  background-color: #f5f7fa;
  min-width: 50px !important;
  cursor: default;
}

.corner-cell:hover {
  background-color: #f5f7fa;
}

.row-header {
  background-color: #f5f7fa;
  text-align: center;
  min-width: 50px !important;
  font-weight: 600;
  cursor: pointer;
  user-select: none;
  color: #515a6e;
}

.row-header:hover {
  background-color: #e6f7ff;
  color: #1890ff;
}

.cell-selected {
  background-color: rgba(24, 144, 255, 0.1);
  outline: 2px solid #1890ff;
  z-index: 1;
  box-shadow: 0 0 0 2px rgba(24, 144, 255, 0.2);
}

.selected {
  background-color: rgba(24, 144, 255, 0.2);
}

.cell-editing,
.header-editing {
  padding: 0 !important;
  position: relative;
  overflow: visible;
  z-index: 10;
}

.cell-editing .ant-input,
.header-editing .ant-input {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  min-height: 46px;
  border: none;
  padding: 10px 12px;
  box-sizing: border-box;
  outline: 2px solid #1890ff;
  border-radius: 0;
  font-size: inherit;
  font-family: inherit;
  z-index: 10;
  background-color: white;
  box-shadow: 0 0 10px rgba(24, 144, 255, 0.3);
}

/* 右键菜单样式 */
.context-menu-trigger {
  position: fixed;
  width: 1px;
  height: 1px;
  z-index: 1000;
}

.context-menu {
  padding: 4px 0;
  border-radius: 4px;
  box-shadow: 0 3px 6px -4px rgba(0, 0, 0, 0.12), 0 6px 16px 0 rgba(0, 0, 0, 0.08), 0 9px 28px 8px rgba(0, 0, 0, 0.05);
}

/* 美化滚动条 */
.table-wrapper::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

.table-wrapper::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 4px;
}

.table-wrapper::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 4px;
}

.table-wrapper::-webkit-scrollbar-thumb:hover {
  background: #a8a8a8;
}

/* 细节优化 */
.vue-table th:first-child,
.vue-table td:first-child {
  border-left: none;
}

.vue-table th:last-child,
.vue-table td:last-child {
  border-right: none;
}

.vue-table tbody tr:last-child td {
  border-bottom: none;
}
</style> 