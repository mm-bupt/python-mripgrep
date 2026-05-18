[English](README_EN.md) | 中文

# python-mripgrep

一个 ripgrep 的 Python 封装，提供快速高效的文本搜索能力。

## 项目简介

python-mripgrep 是 [python-ripgrep](https://github.com/Indent/python-ripgrep) 的克隆版本，旨在解决原版不支持 Python 3.13 / 3.14 的问题。通过精简编译目标，减少了 x86 架构的构建，目前支持以下平台：

| 平台        | 架构                    |
|-----------|------------------------|
| macOS     | x86_64, aarch64 (Apple Silicon) |
| Linux     | x86_64, aarch64              |
| Windows   | x64                        |

**支持的 Python 版本**：3.8 ~ 3.14

## 特性

- 基于 ripgrep 算法的快速文本搜索
- 递归目录搜索
- 正则表达式支持
- 可自定义搜索参数

## 安装

通过 pip 安装：

```
pip install python-mripgrep
```

## 使用示例

```python
from python_mripgrep import search

# 执行简单搜索，返回按文件分组的结果列表
results = search(
    patterns=["pattern"],
    paths=["path/to/search"],
    globs=["*.py"],
)

# 处理搜索结果
for result in results:
    print(result)
```

## API 参考

主要组件：

- `search`：执行搜索的主要函数
- `files`：列出可搜索的文件（等同于 `--files`）
- `PySortMode` 和 `PySortModeKind`：指定排序模式的枚举

详细 API 文档请参考源代码注释。

## 实现细节

### 直接 Rust 集成

与许多其他 ripgrep 的 Python 绑定不同，python-mripgrep **不会通过调用 ripgrep 命令行工具**来工作。相反，它在 Rust 中重新实现了 ripgrep 的核心逻辑，并直接提供给 Python 接口。这种方式有以下优势：

1. **性能**：避免了创建新进程和解析 stdout 的开销，在大规模搜索或频繁调用时更高效。
2. **精细控制**：可以对搜索过程提供更详细的控制，直接返回结构化数据给 Python。
3. **更好的集成**：与 Python 代码紧密集成，更容易融入大型 Python 应用。

### 当前限制

目前库实现了 ripgrep 功能的一个子集，主要支持的搜索选项：

1. `patterns`：搜索模式
2. `paths`：搜索路径
3. `globs`：文件匹配模式（包含或排除）
4. `sort`：搜索结果的排序模式
5. `max_count`：每个文件最多显示的匹配数

## 已实现的标志

以下 ripgrep 标志已在本 Python 封装中实现：

- ✅ `patterns`：搜索模式
- ✅ `paths`：搜索路径（默认：当前目录）
- ✅ `globs`：文件匹配模式（默认：所有非忽略文件）
- ✅ `heading`：是否在匹配行上方显示文件名
- ✅ `sort`：搜索结果的排序模式
- ✅ `max_count`：每个文件最多显示的匹配数
- ✅ `after_context`：每条匹配后显示的行数
- ✅ `before_context`：每条匹配前显示的行数
- ✅ `separator_field_context`：上下文行中字段间的分隔符
- ✅ `separator_field_match`：匹配行中字段间的分隔符
- ✅ `separator_context`：上下文行间的分隔符
- ✅ `-U, --multiline`：启用跨多行匹配

<details>
<summary>尚未实现的标志（点击展开）</summary>

- `-C, --context`：显示每条匹配前后行
- `--color`：控制输出中的颜色使用
- `-c, --count`：仅显示匹配行数
- `--debug`：显示调试信息
- `--dfa-size-limit`：限制正则 DFA 大小
- `-E, --encoding`：指定文件文本编码
- `-F, --fixed-strings`：将模式视为字面字符串
- `-i, --ignore-case`：不区分大小写搜索
- `-v, --invert-match`：反转匹配
- `-n, --line-number`：显示行号
- `-x, --line-regexp`：仅匹配行边界内的模式
- `-M, --max-columns`：不打印超过此长度的行
- `--mmap`：尽可能使用内存映射搜索文件
- `--no-ignore`：不遵循忽略文件
- `--no-unicode`：禁用 Unicode 感知搜索
- `-0, --null`：文件名后打印 NUL 字节
- `-o, --only-matching`：仅打印匹配部分
- `--passthru`：同时打印匹配和非匹配行
- `-P, --pcre2`：使用 PCRE2 正则引擎
- `-p, --pretty`：`--color=always --heading -n` 的别名
- `-r, --replace`：用给定文本替换匹配
- `-S, --smart-case`：智能大小写搜索
- `-s, --case-sensitive`：区分大小写搜索
- `--stats`：打印搜索统计信息
- `-a, --text`：将二进制文件当作文本搜索
- `-t, --type`：仅搜索匹配类型的文件
- `-T, --type-not`：不搜索匹配类型的文件
- `-u, --unrestricted`：减少"智能搜索"级别
- `-V, --version`：打印版本信息
- `-w, --word-regexp`：仅匹配单词边界内的模式
- `-z, --search-zip`：在压缩文件中搜索

</details>

### 扩展功能

要添加更多 ripgrep 选项，需要同时修改 Rust 和 Python 侧的代码：

1. 在 `src/ripgrep_core.rs` 的 `PyArgs` 结构体中添加新选项。
2. 在同一文件的 `pyargs_to_hiargs` 函数中，将新 Python 参数转换为对应的 ripgrep 参数。
3. 更新 Python 封装代码，将新选项暴露给 Python 用户。

示例——添加 `case_sensitive` 选项：

1. 添加到 `PyArgs`：

   ```rust
   pub case_sensitive: Option<bool>,
   ```

2. 在 `pyargs_to_hiargs` 中添加：

   ```rust
   if let Some(case_sensitive) = py_args.case_sensitive {
       low_args.case_sensitive = case_sensitive;
   }
   ```

3. 更新 Python 封装以包含新选项。

## 开发

本项目使用 [maturin](https://github.com/PyO3/maturin) 从 Rust 代码构建 Python 包。开发环境设置：

1. 安装 Rust 和 Python
2. 安装 maturin：`pip install maturin`
3. 克隆仓库
4. 运行 `maturin develop` 本地构建和安装

## 贡献

欢迎贡献！请随时提交 Pull Request。

## 许可证

MIT 许可证 - 详情见 [LICENSE](LICENSE) 文件。

## 致谢

本项目基于 Andrew Gallant 的 [ripgrep](https://github.com/BurntSushi/ripgrep)。

## 发布到 PyPI

本包使用 GitHub Actions 自动发布。在 GitHub 上创建新 release 时，会自动为多平台（Linux、macOS、Windows）构建 wheels 并发布到 PyPI。

发布新版本的步骤：

1. 更新 `pyproject.toml` 和 `Cargo.toml` 中的版本号
2. 提交并推送更改
3. 创建带有标签的新 GitHub release（如 `v0.1.0`）
4. GitHub Actions 将自动构建并发布到 PyPI

---

本项目由 [mm-bupt](https://github.com/mm-bupt) 维护。
