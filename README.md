# C++17 详解（中文翻译版 · LaTeX 工程）

把 Bartłomiej Filipek 的 *C++17 in Detail*（Leanpub）全书译为中文，
并使用 Manning 风格 LaTeX 模板排版。本目录即编译工程根目录；原书英文 PDF 在上一层目录。

- 原书：*C++17 in Detail*
- 作者：Bartłomiej Filipek（cppstories.com）
- 中文名：**C++17 详解**
- 成品：`latex_source/main.pdf`

---

## 目录结构

```
latex_source/                  # 本目录即工程根
├── main.tex                   # 编译入口：\input{book/ccs.tex} + \subfile{book/index.tex}
├── build.bat                  # ⭐ 一键编译脚本（Windows 双击即可，本项目用它编译）
├── fonts/                     # 等宽字体（JetBrains Mono NL 系列，无连字）
├── images/                    # 14 张正文插图（fig_pNNN_1.png，正文以 images/fig_*.png 引用）
├── main.pdf                   # 编译成品（约 287 页）
└── book/
    ├── ccs.tex                # 导言区：自定义命令/环境、minted v3 缓存配置
    ├── index.tex              # 章节装配（增删/调序章节只改这里）
    ├── images/                # 封面 cover.png + 提示框图标 icon_info.png / icon_warning.png + 插图副本
    ├── manning-pygments-style.tex  # 代码高亮配色
    └── content/
        ├── front/             # 00-title(封面) / 01-about-author / 02-tech-reviewer /
        │                       #   03-reviewers / 04-rev-history / 05-preface / 06-about-book
        ├── main/              # 00-quick-start + 第 1–18 章 + part1/2/3 分卷页
        ├── back/              # 空（本书无独立后记/索引章节）
        └── appendix/          # 01(编译器支持表) / 02(资源与参考文献)
```

> 上一层目录另有原书英文 PDF（`*.pdf`）与 `work/` 工作区（提取脚本、章节 txt、校验脚本等，非编译所需）。
> `book/content.bak/` 为 `book/content/` 的整份备份，二者内容一致，正常编译用不到。
> 根目录 `images/` 与 `book/images/` 各有一份 14 张插图副本；正文按 `images/fig_*.png`（相对工程根）引用，封面与图标按 `book/images/cover.png`、`book/images/icon_*.png` 引用。

---

## 环境依赖

| 依赖 | 版本 / 说明 |
|---|---|
| TeX Live | 2026（含 `xelatex`、minted **v3** / `latexminted`、`pygmentize`） |
| Python | 3.x（提供 `pygmentize`，且 `latexminted` 脚本需要可用的 `python.exe`） |
| 中文字体 | **Source Han Serif SC（思源宋体）**、**Microsoft YaHei** |
| 拉丁字体 | Cambria、Franklin Gothic Medium、Consolas |
| 等宽字体 | JetBrains Mono NL（字体文件已放 `fonts/`，也可系统自带 Consolas） |

> 字体缺失会导致编译报 "font not found"。思源宋体与微软雅黑需本机已安装。

---

## 编译

### 方式一：一键脚本（推荐）
在本目录（`latex_source/`）**双击 `build.bat`** 即可。脚本会自动：
1. 校验 `xelatex.exe` 与 `python.exe` 路径（默认指向
   `D:\ProgramData\texlive\2026\bin\windows` 与 `D:\anaconda3`，如不符请改脚本顶部两行）；
2. 把 Python 根目录前置到 `PATH`，并清空 `PYTHONPATH`（保证 `latexminted` 命中正确解释器）；
3. 取工程绝对根写入 `TEXMF_OUTPUT_DIRECTORY` / `TEXMFOUTPUT`（**无尾斜杠**），解决 minted v3 拒绝相对路径写入的问题；
4. 连跑两遍 `xelatex -shell-escape -interaction=nonstopmode -synctex=1 main.tex`；
5. 扫 `main.log` 检查 `Cannot write file` / `Cannot highlight code`（高亮失败不会让 xelatex 返回非 0）；
6. 清理工程根下的 `*.minted` / `*.listing` 临时文件（保留 `_minted/` 缓存以加速下次构建）。

> 脚本为 **ASCII-only**（无中文注释），避免在 zh-CN 代码页（GBK/936）下中文注释变乱码、被当成命令执行。

### 方式二：命令行（等效）
```bash
# 1) 设置 PATH：Python 根目录必须在 TeX bin 之前
export PATH="/d/anaconda3:/d/ProgramData/texlive/2026/bin/windows:$PATH"
# 2) 让 minted v3 把缓存写到绝对路径（见「已知坑」第 1 条），必须指向本工程绝对根
export TEXMF_OUTPUT_DIRECTORY="D:/Ego/trans/C++17 In Detail/latex_source"
export TEXMFOUTPUT="$TEXMF_OUTPUT_DIRECTORY"

# 3) 跑两遍（封面用 remember picture，需两遍才定位）
xelatex -shell-escape -interaction=nonstopmode -file-line-error main.tex
xelatex -shell-escape -interaction=nonstopmode -file-line-error main.tex
```

- `-shell-escape` 是必须的（minted 调外部高亮器）。
- 首次或 `book/content/**` 有变动后，minted 会重高亮全部代码清单（数分钟）；
  缓存命中时一遍约 1 分钟。
- 用 `-file-line-error` 可快速定位报错所在的文件与行号。
- 编译产物：`main.pdf`（约 287 页）、`main.aux`、`main.toc`、`main.log`、`main.synctex.gz` 等。

---

## 封面与提示框

- **封面**（`book/content/front/00-title.tex`）：用原书英文封面 `book/images/cover.png`，
  经 TikZ `remember picture, overlay` 满版铺放（无变形、无黑边），**必须跑满两遍**，
  否则 `current page` 定位失败，封面错位、页数也会异常。
- **提示框**：沿用模板两个页边图标：
  - `book/images/icon_info.png`（ℹ 信息框）
  - `book/images/icon_warning.png`（⚠ 警告框）
  导言区提供 `myInfoBox` / `myWarnBox` 两个环境（左侧页边图标 + 正文缩进），无参数。

---

## 已知坑与约定

1. **minted v3 缓存必须绝对路径**：`latexminted` 拒绝相对路径写入，默认 `_minted`
   缓存会全部失败。已在 `book/ccs.tex` 把 `\minted@cachedir` 设成
   `TEXMF_OUTPUT_DIRECTORY` / `TEXMFOUTPUT` 给出的工程绝对根；`build.bat` 会自动设好这两个变量。
   若改手动编译，务必照「编译 / 方式二」设置，否则报错 `Cannot write file "_xxx.index.minted" outside working directory`。
2. **封面必须两遍**：只跑一遍 `xelatex` 会因 `remember picture` 未定位而封面错位，且页数骤降。
   两遍后恢复正常（约 287 页）。
3. **高亮失败不报错退出码**：`latexminted` 出错时 `xelatex` 仍返回 0，`build.bat` 因此显式
   扫 `main.log` 的 `Cannot write file` / `Cannot highlight code`；手动编译后请自行确认日志无误。
4. **LaTeX 转义铁律**：`\texttt{}` 不改变字符类别，`_ # % & ^ $` 仍需转义；
   `\url{}` 中 `_` 直接写、`# %` 必须转义；代码体 / `minted` 内部无需转义；
   `tabular` 单元格内的字面 `&` 必须写成 `\&`，否则表格不闭合并级联污染后续所有表格。
5. **tabularx 必须 \arraybackslash**：列格式 `>{\raggedright}X`、`>{\ttfamily\raggedright}X`
   等带 `>{}` 的列必须在末尾补 `\arraybackslash`，否则行尾 `\\` 失效、表格不闭合（曾导致 200+ 处级联报错）。
6. **tcolorbox 首行吞参**：`\begin{codebox}` 后若首行以 `[` 开头，会被误吞为语言参数；
   代码清单首行是 `[` 时需在前面加空行或注释规避。
7. **章节装配**：增删/调序/改标题只改 `book/index.tex` 里的 `\myFront` / `\myFrontNoToc` /
   `\myPart` / `\myChapter` / `\myAppendix` 行即可，无需动其它文件。

---

## 字体映射

| 用途 | 字体 |
|---|---|
| 中文正文（衬线） | Source Han Serif SC |
| 中文标题（无衬线） | Microsoft YaHei |
| 拉丁正文 | Cambria |
| 拉丁标题 | Franklin Gothic Medium |
| 代码 / 等宽 | Consolas / JetBrains Mono NL |

---

## 工作流（供复核）

1. 从原书 PDF 用 `../work/extract_figures.py` 等脚本提取章节文本与插图（`../work/chapters/`）。
2. 对照 `../work/TRANSLATION_GUIDE.md`（若有）翻译，产出 `book/content/**/*.tex`。
3. 用 `build.bat` 或上节命令编译，检查 `main.log` 无报错、目视核对封面与提示框。

> 本书索引未纳入本翻译工程（全书采用无行号 + 静态章节装配，未做页级索引）。
