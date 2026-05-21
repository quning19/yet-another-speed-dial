# 二级菜单支持 - 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在点击第一层文件夹标签时，下方显示第二层子文件夹标签页，支持快速切换显示子文件夹的书签

**Architecture:** 在现有文件夹标签架构基础上，新增第二层标签容器。当点击第一层文件夹时，动态渲染该文件夹的直接子文件夹列表。第二层第一个标签显示父文件夹书签，其他标签显示各子文件夹书签。

**Tech Stack:** 原生 JavaScript + Chrome Bookmarks API，依赖现有 Sortable 库实现拖拽排序

---

## 文件结构

```
src/
├── index.html     # 新增 subfoldersContainer HTML 结构
├── css/
│   └── index.css # 新增 .subfolders 和 .subfolder-tab 样式
└── js/
    └── index.js  # 新增状态和函数：buildSubfolderTabs(), switchSubfolder()
```

---

## Task 1: HTML 结构

**Files:**
- Modify: `src/index.html:22-29`

- [ ] **Step 1: 在 index.html 的 foldersContainer 后插入第二层容器**

在 `<div class="folders" id="foldersContainer">...</div>` 后添加：

```html
<!-- 二级菜单容器 -->
<div class="subfolders" id="subfoldersContainer">
    <div id="subfolders" class="subfolders-content"></div>
    <a id="addSubFolderButton" title="Add Subfolder">
        <svg id="addSubFolderIcon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24"><path d="M12.75 2.25a.75.75 0 0 0-1.5 0v9h-9a.75.75 0 0 0 0 1.5h9v9a.75.75 0 0 0 1.5 0v-9h9a.75.75 0 0 0 0-1.5h-9v-9Z"></path></svg>
    </a>
</div>
```

- [ ] **Step 2: 验证 HTML 结构**

确认新增元素正确插入，无语法错误。

- [ ] **Step 3: 提交**

```bash
git add src/index.html
git commit -m "feat: add subfolders container HTML structure"
```

---

## Task 2: CSS 样式

**Files:**
- Modify: `src/css/index.css`

- [ ] **Step 1: 添加 .subfolders 样式**

在 CSS 文件末尾添加：

```css
.subfolders {
    display: flex;
    align-items: center;
    min-height: 40px;
    padding: 0 30px;
    background: rgba(0, 0, 0, 0.2);
}

.subfolders-content {
    display: flex;
    flex-wrap: wrap;
    align-items: baseline;
}

.subfolder-tab {
    opacity: 0.7;
    font-weight: bold;
    font-size: medium;
    margin: 8px 12px;
    cursor: default;
    transition: opacity 160ms ease, padding 160ms ease;
}

.subfolder-tab:hover {
    opacity: 1;
}

.subfolder-tab.active {
    opacity: 1;
    color: #75FB4C;
}

#addSubFolderIcon {
    height: 1rem;
    vertical-align: middle;
    opacity: 0.7;
    margin-left: 12px;
    cursor: pointer;
    fill: var(--color);
    transition: opacity 160ms ease;
}

#addSubFolderIcon:hover {
    opacity: 1;
}
```

- [ ] **Step 2: 验证样式**

启动扩展，确认第二层容器显示正常，样式与第一层文件夹标签一致。

- [ ] **Step 3: 提交**

```bash
git add src/css/index.css
git commit -m "feat: add subfolder tabs CSS styles"
```

---

## Task 3: JavaScript 状态和基础函数

**Files:**
- Modify: `src/js/index.js:137` (状态声明区)
- Modify: `src/js/index.js:463` (folderLink 函数附近)
- Modify: `src/js/index.js:419` (showFolder 函数)

- [ ] **Step 1: 新增状态变量**

在 `let currentFolder = null;` (line 137) 后添加：

```javascript
let currentSubFolder = null;  // 当前第二层选中的子文件夹ID，null表示显示父文件夹书签
let currentSubFolderParent = null;  // 当前第二层所属的第一层文件夹ID
```

- [ ] **Step 2: 新增 DOM 引用**

在 `const addFolderButton = document.getElementById('addFolderButton');` (line 24) 后添加：

```javascript
const subfoldersContainer = document.getElementById('subfoldersContainer');
const subfoldersContent = document.getElementById('subfolders');
const addSubFolderButton = document.getElementById('addSubFolderButton');
```

- [ ] **Step 3: 新增 buildSubfolderTabs 函数**

在 `showFolder` 函数后添加：

```javascript
async function buildSubfolderTabs(parentFolderId) {
    // 记录当前第二层所属的第一层文件夹
    currentSubFolderParent = parentFolderId;

    // 获取父文件夹的子文件夹
    const children = await chrome.bookmarks.getChildren(parentFolderId);
    const subfolders = children.filter(child => !child.url && child.parentId === parentFolderId);

    // 按 index 排序
    subfolders.sort((a, b) => (a.index || 0) - (b.index || 0));

    // 清空第二层容器
    subfoldersContent.innerHTML = '';

    // 获取父文件夹标题（用于"书签"标签）
    let parentTitle = homeFolderTitle;
    if (parentFolderId !== speedDialId) {
        const parentNode = await chrome.bookmarks.get(parentFolderId);
        if (parentNode && parentNode.length > 0) {
            parentTitle = parentNode[0].title;
        }
    }

    // 创建第一个"书签"标签（高亮）
    const bookmarkTab = document.createElement('a');
    bookmarkTab.classList.add('subfolder-tab', 'active');
    bookmarkTab.setAttribute('subfolderId', parentFolderId);
    bookmarkTab.setAttribute('is-bookmark-tab', 'true');
    bookmarkTab.textContent = parentTitle + ' ' + chrome.i18n.getMessage('bookmarks') || '书签';
    bookmarkTab.onclick = function () {
        switchSubfolder(parentFolderId, true);
    };
    subfoldersContent.appendChild(bookmarkTab);

    // 创建子文件夹标签
    for (let subfolder of subfolders) {
        const tab = document.createElement('a');
        tab.classList.add('subfolder-tab');
        tab.setAttribute('subfolderId', subfolder.id);
        tab.textContent = subfolder.title;
        tab.onclick = function () {
            switchSubfolder(subfolder.id, false);
        };
        subfoldersContent.appendChild(tab);
    }

    // 添加 [+ ] 新增子文件夹按钮
    addSubFolderButton.onclick = function () {
        createSubFolder(parentFolderId);
    };

    // 重置 currentSubFolder
    currentSubFolder = null;
}
```

- [ ] **Step 4: 新增 switchSubfolder 函数**

在 `buildSubfolderTabs` 后添加：

```javascript
async function switchSubfolder(subfolderId, isBookmarkTab) {
    // 高亮对应标签
    const tabs = document.getElementsByClassName('subfolder-tab');
    for (let tab of tabs) {
        if (tab.getAttribute('subfolderId') === subfolderId) {
            tab.classList.add('active');
        } else {
            tab.classList.remove('active');
        }
    }

    // 如果点击的是"书签"标签
    if (isBookmarkTab) {
        currentSubFolder = null;
        // 显示父文件夹的书签
        const children = await chrome.bookmarks.getChildren(subfolderId);
        await printBookmarks(children, subfolderId);
    } else {
        // 显示子文件夹的书签
        currentSubFolder = subfolderId;
        const children = await chrome.bookmarks.getChildren(subfolderId);
        await printBookmarks(children, subfolderId);
    }
}
```

- [ ] **Step 5: 新增 createSubFolder 函数**

在 `createFolder` 函数附近添加：

```javascript
function createSubFolder(parentId) {
    hideSettings();
    createFolderModalName.value = '';
    createFolderModalName.focus();
    createFolderModal.style.transform = "translateX(0%)";
    createFolderModal.style.opacity = "1";
    createFolderModalContent.style.transform = "scale(1)";
    createFolderModalContent.style.opacity = "1";
    // 临时存储父文件夹ID
    createFolderModal.dataset.parentId = parentId;
}
```

- [ ] **Step 6: 修改 createFolder 函数**

找到 `function createFolder()` 或 `saveFolder` 函数，修改为支持传入 parentId：

原函数可能在 line 491 附近，检查 `createFolder` 是否使用 `speedDialId` 作为 parentId，改为支持从 `createFolderModal.dataset.parentId` 读取。

```javascript
function createFolder() {
    let name = createFolderModalName.value.trim();
    let parentId = createFolderModal.dataset.parentId || speedDialId;

    if (name.length) {
        chrome.bookmarks.create({
            title: name,
            parentId: parentId
        }).then(node => {
            hideModals();
            // 刷新第二层
            if (parentId === currentSubFolderParent || parentId === currentFolder) {
                buildSubfolderTabs(parentId === currentSubFolderParent ? parentId : currentFolder);
            }
        });
    } else {
        hideModals();
    }
}
```

- [ ] **Step 7: 修改 showFolder 函数**

在 `showFolder(id)` 函数 (line 419) 中，点击文件夹时调用 `buildSubfolderTabs`：

```javascript
function showFolder(id) {
    hideSettings();
    let folders = document.getElementsByClassName('container');
    for (let folder of folders) {
        if (folder.id === id) {
            folder.style.display = "flex"
            folder.style.opacity = "0";
            layoutFolder = true;
            setTimeout(function () {
                folder.style.opacity = "1";
                animate()
            }, 20);
        } else {
            folder.style.display = "none";
        }
    }
    // style the active tab
    let folderTitles = document.getElementsByClassName('folderTitle');
    for (let title of folderTitles) {
        if (title.attributes.folderid.value === id) {
            title.classList.add('activeFolder');
        } else {
            title.classList.remove('activeFolder');
        }
    }

    // 构建第二层标签
    buildSubfolderTabs(id);
}
```

- [ ] **Step 8: 验证实现**

1. 启动扩展
2. 点击第一层文件夹，查看第二层是否显示
3. 点击第二层"书签"标签，查看主显示区
4. 点击子文件夹标签，查看主显示区是否切换
5. 切换第一层，查看第二层是否刷新

- [ ] **Step 9: 提交**

```bash
git add src/js/index.js
git commit -m "feat: implement subfolder tabs functionality"
```

---

## Task 4: 边界情况处理

**Files:**
- Modify: `src/js/index.js`

- [ ] **Step 1: 处理无子文件夹的情况**

在 `buildSubfolderTabs` 中，确保无子文件夹时只显示一个"书签"标签（已有逻辑，应该正常）。

- [ ] **Step 2: 处理 speedDialId 情况**

确保点击"主页"时不显示第二层（或显示但禁用）。在 `showFolder` 中判断：

```javascript
// 只在非 speedDialId 时构建第二层
if (id !== speedDialId) {
    buildSubfolderTabs(id);
} else {
    // 清空第二层
    subfoldersContent.innerHTML = '';
    currentSubFolder = null;
    currentSubFolderParent = null;
}
```

- [ ] **Step 3: 验证边界情况**

1. 点击主页（speedDialId），确认无第二层
2. 文件夹无子文件夹时，确认第二层只显示一个标签

- [ ] **Step 4: 提交**

```bash
git add src/js/index.js
git commit -m "feat: handle edge cases for subfolder tabs"
```

---

## 自检清单

1. **Spec coverage:** 检查设计文档中的每条交互规则是否都有实现
   - [x] 点击第一层文件夹A → 第二层刷新 + 默认选中"书签"标签 + 主显示区刷新
   - [x] 点击第二层"书签"标签 → 高亮该标签，主显示区显示A的书签
   - [x] 点击第二层"子文件夹1" → 高亮"子文件夹1"标签，主显示区改为显示子文件夹1的书签
   - [x] 切换第一层文件夹B → 第二层刷新为B的子结构，B的"书签"标签高亮，主显示区刷新为B的书签
   - [x] 点击 [+ 新增] 按钮 → 弹出创建子文件夹模态框
   - [x] 最大深度限制：只支持2层

2. **Placeholder scan:** 无 TBD/TODO/placeholder

3. **Type consistency:** 函数签名一致，无命名冲突

---

## 执行方式选择

**Plan complete and saved to `docs/superpowers/plans/2026-05-20-subfolder-tabs-plan.md`. Two execution options:**

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?**