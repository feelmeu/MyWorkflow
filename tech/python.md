# Python 技术栈增补

> 本文件补充在 Python 项目中使用的特定术语与约定。主术语见 `CONTEXT.md`。

---

## Language

**PyPI**：Python 官方包索引，第三方库的主要来源。`pip install <包名>` 的默认仓库。

**venv / Virtual Environment**：Python 虚拟环境，用于隔离项目依赖，避免全局包污染。

**requirements.txt**：传统的依赖清单文件，每行一个 `package==version`。用 `pip install -r requirements.txt` 安装。

**pyproject.toml**：PEP 518/621 引入的现代项目配置文件，可以同时承载构建配置、依赖声明、工具配置。新项目优先使用它。

**Module**：一个 `.py` 文件，可被 `import` 导入。

**Package**：一个包含 `__init__.py`（或 `__init__.py` 非必需的命名空间包）的目录，是 module 的集合。

**CLI Entry Point**：在 `pyproject.toml` 的 `[project.scripts]` 中声明的命令入口，用户通过它调用程序。

**Chunk**：文件下载时按字节切分的一段。在 HTTP 中通过 `Range: bytes=start-end` 请求头获取。

**Range Request**：HTTP 协议中的字节范围请求，服务端通过 `Accept-Ranges: bytes` 响应头表明支持。

**Content-Disposition**：HTTP 响应头，可能包含服务端推荐的文件名（`attachment; filename="xxx"`）。

**Progress Bar**：终端中的实时进度显示组件，常见库为 `tqdm`。

**Threading / Multithreading**：Python 内置的多线程库。受 GIL 影响，CPU 密集型任务收益有限，但对 I/O 密集型任务（如网络下载）效果很好。

**GIL（Global Interpreter Lock）**：CPython 中的全局解释器锁，导致同一进程内同一时刻只有一个线程在执行 Python 字节码。对于 I/O 阻塞任务影响不大，因为阻塞时会释放 GIL。

**Asyncio / async-await**：Python 3.5+ 的异步 I/O 框架，基于事件循环。对高并发 I/O 任务是 threading 之外的另一种选择。

---

## Conventions

- **依赖声明**：使用 `pyproject.toml` 声明依赖和项目元数据，不再使用 `setup.py` / `setup.cfg`。
- **测试**：使用 `pytest` 作为测试框架。
- **CLI 参数解析**：使用标准库 `argparse`（无第三方依赖），除非功能复杂度需要 `click`。
- **代码风格**：遵循 PEP 8。
- **最小依赖原则**：优先使用标准库，仅在必要时引入第三方依赖。
