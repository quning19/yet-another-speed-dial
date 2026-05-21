# 二级菜单支持 - 设计文档

## 背景

Yet Another Speed Dial 目前的文件夹导航只有一层。用户希望在点击第一层文件夹标签时，能在下方显示一行子文件夹标签页，方便快速切换显示子文件夹的书签。

## 交互设计

### 结构

```
┌─────────────────────────────────────────────────────────┐
│ 第一层: [文件夹A] [文件夹B] [文件夹C] [+新增文件夹]       │
│                ↓ 点击文件夹A后                            │
│ 第二层: [A书签] [子文件夹1] [子文件夹2] [+新增子文件夹] │
└─────────────────────────────────────────────────────────┘

书签显示区（平铺显示当前选中内容）
```

### 层级关系

- **第一层**：现有顶层文件夹标签，每个对应一个书签容器
- **第二层**：仅显示当前选中第一层文件夹的子结构
  - 第一个固定标签：标签文字为父文件夹名称，显示该文件夹内的书签
  - 后续标签：按顺序显示各个子文件夹
  - 最后一个 [+ ]：新增子文件夹

### 交互规则

1. **点击第一层文件夹A**
   - 第二层刷新为A的直接子文件夹列表
   - 第二层的"书签"标签默认高亮
   - 主显示区域刷新为文件夹A下的书签（不含子目录）
2. **点击第二层"书签"标签** → 高亮该标签，主显示区显示A的书签
3. **点击第二层"子文件夹1"** → 高亮"子文件夹1"标签，主显示区改为显示子文件夹1的书签
4. **切换第一层文件夹B** → 第二层刷新为B的子结构，B的"书签"标签高亮，主显示区刷新为B的书签
5. **点击第二层 [+ 新增] 按钮** → 弹出创建子文件夹模态框
6. **最大深度限制**：只支持2层（不能再往子文件夹的子文件夹递归）

### 状态管理

```javascript
// 新增状态
let currentSubFolder = null;  // 当前第二层选中的子文件夹ID，null表示显示第一层文件夹本身的书签
```

### 视觉表现

- **第二层标签**：使用 `.subfolder-tab` 样式类
- **高亮状态**：`.active` 类
- **第二层容器**：新增 HTML 元素，插入到 `#foldersContainer` 下方

## 技术实现

### HTML 结构

在 `#foldersContainer` 后新增：

```html
<div class="subfolders" id="subfoldersContainer">
    <div id="subfolders"></div>
    <a id="addSubFolderButton" title="Add Subfolder">+</a>
</div>
```

### CSS 样式

```css
.subfolders {
    display: flex;
    /* 与第一层对齐 */
}

.subfolder-tab {
    /* 与 folderTitle 类似样式 */
}

.subfolder-tab.active {
    /* 高亮状态 */
}
```

### 核心函数变更

#### 新增函数

```javascript
function buildSubfolderTabs(parentFolderId) {
    // 获取parentFolderId的直接子文件夹
    // 渲染：第一个"书签"标签 + 各子文件夹标签 + [+新增]按钮
    // 注册点击事件
}

function switchSubfolder(subfolderId) {
    // 高亮对应标签
    // 调用现有 printBookmarks 显示对应内容
}

function handleSubfolderClick(subfolderId) {
    // currentSubFolder = subfolderId
    // 更新主显示区
}
```

#### 修改函数

```javascript
// showFolder() - 点击第一层文件夹时调用 buildSubfolderTabs
function showFolder(id) {
    // 现有逻辑...
    if (id !== speedDialId) {
        buildSubfolderTabs(id);
    }
}

// printBookmarks() - 传入 currentSubFolder 参数决定显示谁的书签
```

### 数据流

1. 用户点击第一层文件夹 → `showFolder(id)` → `buildSubfolderTabs(id)`
2. 第二层标签点击 → `handleSubfolderClick(subfolderId)` → 更新 `currentSubFolder` → 调用 `printBookmarks(children, subfolderId)`
3. 第二层"书签"标签点击 → `currentSubFolder = null` → 调用 `printBookmarks(children, firstLevelFolderId)`

## 边界情况

1. **无子文件夹的文件夹**：第二层只显示一个"书签"标签（高亮）
2. **切换第一层时**：第二层刷新，原高亮状态重置
3. **新增子文件夹**：在第二层末尾添加新标签

## 未来扩展功能（暂不实现）

以下功能计划后续添加：

1. **显示访问次数**：在页签或书签上显示访问次数统计
2. **显示书签数量**：在子文件夹页签上显示其中书签的数量
3. **手动调整页签顺序**：支持拖拽调整第一层/第二层页签顺序
4. **自动调整页签顺序**：根据访问频率自动排序
5. **优化页签显示效果**：改进页签的视觉表现
6. **优化导入导出功能**：增强数据迁移功能

---

## 参考实现

参考项目 `C:\work\tools\ff-speed-dial\` 的 `breadcrumbs` 导航有类似层级展示逻辑，但本项目采用水平标签页而非面包屑路径。