# 页签外观设置 — 设计文档

## 背景

一级和二级页签的字号目前是硬编码 `font-size: medium`，拖拽放大时一级页签 padding 固定 60px 偏大。用户希望自定义字号，并将拖拽放大尺寸调整合理。

## 设计

### CSS 变量

新增 4 个 CSS 变量，默认值保持现有行为：

```css
--folder-tab-font-size: medium;         /* 一级页签字号 */
--subfolder-tab-font-size: medium;      /* 二级页签字号 */
--folder-drop-padding: 40px;            /* 一级拖拽放大（原 60px） */
--subfolder-drop-padding: 24px;         /* 二级拖拽放大（新增） */
```

### 样式修改

**一级页签基类：**
```css
.folderTitle {
    font-size: var(--folder-tab-font-size);
}
```

**一级拖拽放大（`.folders-drag-active .folderTitle`）：**
```css
.folders-drag-active .folderTitle {
    padding: var(--folder-drop-padding);       /* 原 60px → 40px */
    font-size: calc(var(--folder-tab-font-size) * 1.5);
    margin: 0;
    border-radius: 12px;
}
```

**二级页签基类：**
```css
.subfolder-tab {
    font-size: var(--subfolder-tab-font-size);
}
```

**二级拖拽放大（`.subfolders-drag-active .subfolder-tab`）新增：**
```css
.subfolders-drag-active .subfolder-tab {
    padding: var(--subfolder-drop-padding);
    font-size: calc(var(--subfolder-tab-font-size) * 1.5);
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

### 设置面板

在 `dialSize` 行后新增两个字号选择器，选项与 dialSize 一致：

| ID | Label | 默认值 |
|----|-------|--------|
| `folderTabFontSize` | 一级页签字号 | medium |
| `subfolderTabFontSize` | 二级页签字号 | medium |

选项值：`xx-small` / `x-small` / `small` / `medium` / `large` / `x-large` / `xx-large`

### JS 改动

**defaults 新增：**
```javascript
folderTabFontSize: 'medium',
subfolderTabFontSize: 'medium',
```

**saveSettings 新增：** 读取 select 值写入 `settings.folderTabFontSize` / `settings.subfolderTabFontSize`

**applySettings 新增：** 将值写入 CSS 变量 `--folder-tab-font-size` / `--subfolder-tab-font-size`

### 涉及文件

- `src/css/index.css` — 新增 CSS 变量 + 修改样式
- `src/index.html` — 设置面板新增两行
- `src/js/index.js` — defaults / saveSettings / applySettings 新增字段

### 边界情况

- 导入旧版设置文件（无新字段）时，`Object.assign(defaults, ...)` 会填入默认值 medium
- 导出包含新字段，导入后正常生效
