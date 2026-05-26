# ppt-master 下载与安装经验

## 技能仓库位置

ppt-master 作为 Claude Code 的自定义技能，安装路径为：

```
C:\Users\cchen\.claude\skills\ppt-master\
```

该目录在用户启用该技能后自动部署，内容包括 `scripts/`、`templates/`、`references/`、`workflows/` 等核心模块。

## 环境依赖

### Python 解释器

- 系统安装了 Python 3（通过 `python` 命令调用）
- **注意**：Windows 上 `python3` 命令可能指向 Microsoft Store 的 Python 安装存根（exit code 49），应使用 `python` 替代
- 所有脚本均以 `python <script_path>` 形式执行

### Python 包依赖

依赖定义在 `requirements.txt` 中，核心依赖包括：

| 包名 | 用途 |
|------|------|
| `python-pptx` | SVG 转 PPTX 导出 |
| `PyMuPDF` | PDF 转 Markdown |
| `Pillow` | 图片处理 |
| `mammoth`, `markdownify` | DOCX/HTML 转 Markdown |
| `openpyxl` | Excel 转 Markdown |
| `beautifulsoup4`, `requests` | 网页抓取与解析 |
| `cairosvg` 或 `svglib` | SVG 转 PNG（Office 兼容模式） |

安装命令：

```bash
pip install -r C:/Users/cchen/.claude/skills/ppt-master/requirements.txt
```

## 已知问题

1. **`python3` vs `python`**：Windows 上必须使用 `python`，而非 `python3`
2. **`gbk` 编解码错误**：部分脚本在 Windows 终端输出非 ASCII 字符时可能触发 `UnicodeEncodeError`，不影响实际功能（输出文件内容正常）
3. **EMF/WMF 矢量图**：浏览器实时预览无法渲染 EMF/WMF 格式，PPTX 输出为正确结果
