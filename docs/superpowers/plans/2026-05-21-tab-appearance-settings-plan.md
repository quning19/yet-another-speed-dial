# 页签外观设置 — 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 支持一级和二级页签字号自定义，调整拖拽放大尺寸，给二级菜单加拖拽放大效果

**Architecture:** CSS 变量驱动字号和拖拽尺寸，JS 通过 `setProperty` 写入，设置面板用 `<select>` 控件

**Tech Stack:** 原生 JavaScript + CSS 变量，复用现有 `dialSize` 选项值体系

---

## 文件结构

```
src/
├── css/
│   └── index.css  # 新增 CSS 变量声明 + 修改 .folderTitle/.subfolder-tab 样式
├── index.html     # 设置面板新增两行字号选择器
└── js/
    └── index.js   # defaults + applySettings + saveSettings 新增字段
```

---

### Task 1: CSS — 新增变量 + 修改样式

**Files:**
- Modify: `src/css/index.css:241-264` (`.folderTitle` and `.folders-drag-active .folderTitle`)
- Modify: `src/css/index.css:964-990` (`.subfolder-tab` and `.subfolders-drag-active`)

- [ ] **Step 1: 在 `.folderTitle` 基类添加 `font-size`**

在 `src/css/index.css` 的 `.folderTitle` 块（line 241）中添加一行：

```css
.folderTitle {
    opacity: 0.85;
    font-weight: bold;
    font-size: var(--folder-tab-font-size, medium);
    transition: padding 160ms ease, margin 160ms ease, outline 160ms ease, background 160ms ease;
    padding: 4px 6px;
    margin: 8px 6px;
}
```

- [ ] **Step 2: 修改一级拖拽放大样式**

修改 `.folders-drag-active .folderTitle`（line 254），将 padding 改为 CSS 变量，新增 `font-size` 放大 1.5 倍：

```css
.folders-drag-active .folderTitle {
    padding: var(--folder-drop-padding, 40px);
    font-size: calc(var(--folder-tab-font-size, medium) * 1.5);
    margin: 0;
    border-radius: 12px;
}
```

- [ ] **Step 3: 在 `.subfolder-tab` 基类添加 `font-size`**

在 `.subfolder-tab` 块（line 964）中添加一行：

```css
.subfolder-tab {
    opacity: 0.7;
    font-weight: bold;
    font-size: var(--subfolder-tab-font-size, medium);
    margin: 8px 12px;
    cursor: default;
    transition: opacity 160ms ease, padding 160ms ease;
}
```

- [ ] **Step 4: 给二级菜单添加拖拽放大效果**

在 `.subfolders-drag-active .subfolder-tab.drag-hover` 之前（line 982 前）新增 `.subfolders-drag-active .subfolder-tab` 规则：

```css
.subfolders-drag-active .subfolder-tab {
    padding: var(--subfolder-drop-padding, 24px);
    font-size: calc(var(--subfolder-tab-font-size, medium) * 1.5);
}

.subfolders-drag-active .subfolder-tab.drag-hover {
    background: rgba(255, 255, 255, 0.1);
    transform: scale(1.1);
    border-radius: 4px;
}

.subfolders-drag-active .subfolder-tab:not(.drag-hover) {
    opacity: 0.5;
}
```

- [ ] **Step 5: 提交**

```bash
git add src/css/index.css
git commit -m "feat: add font-size CSS vars and drag-expand for subfolder tabs"
```

---

### Task 2: HTML — 设置面板新增字号选择器

**Files:**
- Modify: `src/index.html:353-372` (`dialSize` 行之后)

- [ ] **Step 1: 在 `dialSize` 行后插入两个字号选择器**

在 `dialSize` 的 `</div>` 之后、`maxCols` 行之前（约为 line 354）插入：

```html
            <div class='row'>
                <div class='column'>
                    <label data-locale="folderTabFontSize">Folder Tab Font Size</label>
                </div>
                <div class='column controls'>
                    <select class="settingsCtl" id="folderTabFontSize" name="folderTabFontSize">
                        <option value="xx-small">XX-Small</option>
                        <option value="x-small">X-Small</option>
                        <option value="small">Small</option>
                        <option value="medium" selected>Medium</option>
                        <option value="large">Large</option>
                        <option value="x-large">X-Large</option>
                        <option value="xx-large">XX-Large</option>
                    </select>
                </div>
            </div>
            <div class='row'>
                <div class='column'>
                    <label data-locale="subfolderTabFontSize">Subfolder Tab Font Size</label>
                </div>
                <div class='column controls'>
                    <select class="settingsCtl" id="subfolderTabFontSize" name="subfolderTabFontSize">
                        <option value="xx-small">XX-Small</option>
                        <option value="x-small">X-Small</option>
                        <option value="small">Small</option>
                        <option value="medium" selected>Medium</option>
                        <option value="large">Large</option>
                        <option value="x-large">X-Large</option>
                        <option value="xx-large">XX-Large</option>
                    </select>
                </div>
            </div>
```

- [ ] **Step 2: 提交**

```bash
git add src/index.html
git commit -m "feat: add folder and subfolder tab font size selectors to settings"
```

---

### Task 3: JS — defaults + applySettings + saveSettings

**Files:**
- Modify: `src/js/index.js:158-176` (defaults)
- Modify: `src/js/index.js:2038-2042` (applySettings 中 `--folder-drop-padding` 赋值)
- Modify: `src/js/index.js:2144-2161` (saveSettings)
- Modify: `src/js/index.js` 中新增 DOM 引用和变量

- [ ] **Step 1: 新增 DOM 引用**

在 `src/js/index.js` 的 DOM 引用区域（`const dialSizeInput = ...` 附近），添加两个新引用。找到 `dialSizeInput`：

```javascript
const dialSizeInput = document.getElementById("dialSize");
```

在其后添加：

```javascript
const folderTabFontSizeInput = document.getElementById("folderTabFontSize");
const subfolderTabFontSizeInput = document.getElementById("subfolderTabFontSize");
```

- [ ] **Step 2: defaults 新增字段**

在 `defaults` 对象（line 158）中添加：

```javascript
folderTabFontSize: 'medium',
subfolderTabFontSize: 'medium',
```

- [ ] **Step 3: applySettings 新增 CSS 变量写入**

在 `applySettings` 中，找到 `--folder-drop-padding` 的设置位置（line 2042），在其后添加字号变量写入。同时将 `folderDropPadding` 的默认值从 60px 改为 40px。

修改各 case 中的 `folderDropPadding` 值，以及 default 分支（line 2036 和 2046）：
- `xx-large`: `'50px'` → `'40px'`
- `x-large`: `'46px'` → `'36px'`
- `large`: `'42px'` → `'32px'`
- `medium`: `'40px'` → `'28px'`
- `small`: `'34px'` → `'24px'`
- `x-small`: `'28px'` → `'20px'`
- `xx-small`: `'20px'` → `'20px'`
- default (line 2046): `'60px'` → `'40px'`

在 `--folder-drop-padding` 的 `setProperty` 行（line 2042）之后添加：

```javascript
document.documentElement.style.setProperty('--folder-tab-font-size', settings.folderTabFontSize);
document.documentElement.style.setProperty('--subfolder-tab-font-size', settings.subfolderTabFontSize);
document.documentElement.style.setProperty('--subfolder-drop-padding', '24px');
```

- [ ] **Step 4: applySettings 回填 select 控件**

在 `applySettings` 中，找到其他 select 的回填位置（`dialSizeInput.value = settings.dialSize` 附近，line 2036 区域），添加：

```javascript
folderTabFontSizeInput.value = settings.folderTabFontSize;
subfolderTabFontSizeInput.value = settings.subfolderTabFontSize;
```

- [ ] **Step 5: saveSettings 新增字段**

在 `saveSettings`（line 2144）中添加两行：

```javascript
settings.folderTabFontSize = folderTabFontSizeInput.value;
settings.subfolderTabFontSize = subfolderTabFontSizeInput.value;
```

- [ ] **Step 6: 提交**

```bash
git add src/js/index.js
git commit -m "feat: add tab font size settings to JS logic"
```

---

## 自检清单

1. **Spec coverage:**
   - [x] CSS 变量 `--folder-tab-font-size` / `--subfolder-tab-font-size` 声明和使用
   - [x] 一级拖拽 padding 改为 40px 变量
   - [x] 一级拖拽字号放大 1.5 倍
   - [x] 二级拖拽新增 padding + 字号放大效果
   - [x] 设置面板两个 select 控件
   - [x] JS defaults / saveSettings / applySettings 完整支持

2. **Placeholder scan:** 无 TBD/TODO

3. **Type consistency:** `folderTabFontSize` / `subfolderTabFontSize` 命名在 CSS 变量、HTML id、JS 中一致
