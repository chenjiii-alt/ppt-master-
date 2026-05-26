# ppt-master 使用指南

本文档基于 ppt-master 技能生成 MCM/ICM 真菌碳平衡答辩 PPT 的完整流程编写，旨在提供一套可复用的操作指引。

---

## 一、总览

ppt-master 是一个基于 Claude Code 的 PPT 生成管道，输入源文档（PDF、DOCX、Markdown、网页等），输出可编辑的 PPTX 文件。

**核心流程**：

```
源文档 → 内容转换 → 项目初始化 → 方案设计（Strategist）
   → SVG 生成（Executor） → 后处理 → PPTX 导出
```

---

## 二、前置准备

### 2.1 环境要求

- Claude Code 已安装 ppt-master 技能
- Python 3.x（Windows 上使用 `python` 命令，非 `python3`）
- 安装 Python 依赖：

  ```bash
  pip install -r C:/Users/cchen/.claude/skills/ppt-master/requirements.txt
  ```

### 2.2 技能变量

关键路径变量（SKILL_DIR）：

```
C:/Users/cchen/.claude/skills/ppt-master/
```

---

## 三、操作步骤

### Step 1：源内容处理

将用户提供的源材料转换为 Markdown 格式。

| 源格式 | 命令 |
|--------|------|
| PDF | `python ${SKILL_DIR}/scripts/source_to_md/pdf_to_md.py <文件路径>` |
| DOCX | `python ${SKILL_DIR}/scripts/source_to_md/doc_to_md.py <文件路径>` |
| XLSX | `python ${SKILL_DIR}/scripts/source_to_md/excel_to_md.py <文件路径>` |
| 网页链接 | `python ${SKILL_DIR}/scripts/source_to_md/web_to_md.py <URL>` |

**输出**：Markdown 文件及提取的图片资源。

### Step 2：项目初始化

创建项目目录结构，导入源材料。

```bash
# 创建项目（ppt169 为 16:9 格式）
python ${SKILL_DIR}/scripts/project_manager.py init <项目名称> --format ppt169

# 导入源文件（--move 将文件移至项目目录）
python ${SKILL_DIR}/scripts/project_manager.py import-sources <项目路径> <源文件...> --move
```

**输出**：项目目录结构，包含 `sources/`、`notes/`、`svg_output/` 等子目录。

**支持的格式**：`ppt169`（16:9，默认）、`ppt43`（4:3）、`xhs`（小红书）、`story` 等。

### Step 3：模板选项（可选）

默认采用自由设计，无需模板。如需使用模板：

- Brand 模板：提供色彩、字体、Logo 等品牌标识
- Layout 模板：提供页面结构、版式布局
- Deck 模板：完整的品牌 + 布局 + 页面方案

模板可融合使用（如 Brand + Layout）。

### Step 4：方案设计（Strategist）

> ⛔ 本步骤需要用户确认后方可继续。

1. 阅读策略师指南：`read_file ${SKILL_DIR}/references/strategist.md`
2. 阅读设计规范参考模板：`read_file <项目路径>/templates/design_spec_reference.md`
3. 向用户呈现 **八项确认**：

   | 编号 | 确认项 |
   |------|--------|
   | ① | 画布格式 |
   | ② | 页数范围 |
   | ③ | 目标受众 |
   | ④ | 风格目标 |
   | ⑤ | 配色方案 |
   | ⑥ | 图标使用策略 |
   | ⑦ | 字体方案 |
   | ⑧ | 图片使用策略 |

4. 用户确认后，编写 `design_spec.md`（完整设计规范，含每页内容大纲）
5. 生成 `spec_lock.md`（执行契约，含颜色、字体、图标清单、页面节奏等最终锁定值）

### Step 5：图片分析（可选）

如果源文档包含图片，运行图片分析工具获取描述：

```bash
python ${SKILL_DIR}/scripts/analyze_images.py <项目路径>
```

### Step 6：SVG 生成（Executor）

> ⚠️ 本步骤由主 Agent 逐页手写 SVG，禁止脚本批量生成。

1. 每次生成前重新读取 `spec_lock.md`，获取当前页的 `page_rhythm`、`page_layouts`、`page_charts` 配置
2. 按照 `design_spec.md` 的内容大纲逐页生成 SVG
3. 每页保存为 `<项目路径>/svg_output/<页码>_<页面名>.svg`
4. 所有图标、颜色、字体必须严格取自 `spec_lock.md`

**SVG 画布**：`viewBox="0 0 1280 720"`（PPT 16:9）

**关键约束**：
- 禁止使用 `rgba()`、`<style>`、`<foreignObject>`、`<animate*>`、`<script>`
- 禁止使用 HTML 命名实体（如 `&nbsp;`），必须使用原始 Unicode
- 图标库统一使用 `tabler-outline`，`stroke-width="2"`
- 每个 `<g>` 的透明度须在子元素上单独设置

### Step 7：后处理与导出

依次执行以下三个脚本：

```bash
# 7.1 拆分演讲者备注
python ${SKILL_DIR}/scripts/total_md_split.py <项目路径>

# 7.2 SVG 后处理（嵌入图标、对齐图片、展平文本、转换圆角矩形为路径）
python ${SKILL_DIR}/scripts/finalize_svg.py <项目路径>

# 7.3 导出 PPTX
python ${SKILL_DIR}/scripts/svg_to_pptx.py <项目路径>
```

**输出**：`<项目路径>/exports/<项目名>_<时间戳>.pptx`

---

## 四、关键文件说明

| 文件 | 用途 |
|------|------|
| `sources/` | 源文档及转换产物 |
| `notes/total.md` | 完整演讲者备注（每页以 `---` 分隔） |
| `svg_output/*.svg` | 每页 SVG 源文件 |
| `spec_lock.md` | 设计执行契约（颜色、字体、图标、页面节奏） |
| `templates/design_spec.md` | 完整设计规范与内容大纲 |
| `backup/` | SVG 后处理前的自动备份 |
| `exports/*.pptx` | 最终导出产物 |

---

## 五、常见问题

### 5.1 `python3` 命令报错

**现象**：`python3` 返回 exit code 49。
**原因**：Windows 上 `python3` 指向 Microsoft Store 安装存根。
**解决**：全部使用 `python` 命令替代。

### 5.2 图标未在 `spec_lock.md` 中声明

**现象**：质量检查提示图标不在 inventory 中（warning）。
**解决**：将使用的图标补充到 `spec_lock.md` 的 `## icons.inventory` 列表中，或提示作为 warning 忽略（不阻塞流程）。

### 5.3 SVG 中的图片路径引用

**现象**：图片在 SVGs 中显示为空白。
**解决**：`finalize_svg.py` 会自动将图片嵌入 SVG，但生成阶段引用路径应与 `spec_lock.md` 中 `## images` 的路径一致。

### 5.4 终端输出中文乱码

**现象**：脚本运行时终端显示乱码（如 `�����`）。
**说明**：Windows 终端编码问题，实际生成的文件内容为正确 UTF-8 编码，不影响功能。

---

## 六、质量保证

- `svg_quality_checker.py`：检查 SVG 文件合规性（viewBox、颜色、图标、字体）
- 每次后处理前自动备份原始 SVG 到 `backup/` 目录
- PPTX 导出为原生 DrawingML 形状（在 PowerPoint 中可直接编辑）
