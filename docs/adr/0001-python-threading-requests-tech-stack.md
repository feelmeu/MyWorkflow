# 0001 — 使用 Python + threading + requests 作为 MVP 技术栈

- 状态：Accepted
- 日期：2026-06-21
- 替代：无

## 背景

我们需要构建一个命令行下载加速器，核心功能是接收 HTTP 下载直链，通过分段下载提升速度。在工具的技术选型上存在多个合理选择：语言（Go / Rust / Python / Node.js）、并发模型（threading / asyncio / 原生协程）、HTTP 客户端库（requests / urllib / aiohttp / httpx）。

## 决策

我们决定使用 **Python 3 + threading + requests** 作为 MVP 的技术栈。具体：

- **语言**：Python 3.8+
- **并发模型**：标准库 `threading`，每个分段一个工作线程
- **HTTP 客户端**：`requests` 库（`stream=True` 模式 + `Range` 请求头）
- **CLI 参数解析**：标准库 `argparse`
- **进度条**：`tqdm` 库
- **测试框架**：`pytest`

## 理由

- **理由 1**：Python 开发速度快，代码直观易读，对于 CLI 工具这种中等规模项目，开发效率比极致性能更重要。
- **理由 2**：下载是 I/O 密集型任务，`threading` 在 I/O 阻塞时会释放 GIL，实际并发效果良好。分段数通常在 8–16，`threading` 完全够用，不需要更复杂的 `asyncio`。
- **理由 3**：`requests` 生态成熟、文档丰富、处理 HTTP Range 请求和流式下载（`stream=True` + `iter_content`）非常方便，比标准库 `urllib` 省心。
- **理由 4**：`argparse` 是标准库，无需额外安装即可实现丰富的参数解析。
- **理由 5**：用户对 Python 熟悉，后续维护和扩展成本低。

## 备选方案与放弃原因

### 备选方案 A：Go + 标准库 net/http + goroutine

Go 静态编译单文件、原生 goroutine 轻量级并发、`net/http` 性能优秀，非常适合写网络工具。**放弃原因**：团队对 Go 不熟悉，学习曲线 + 生态工具链（测试、打包等）都需要重新适应，对 MVP 来说投入产出比不优。

### 备选方案 B：Python + asyncio + aiohttp

`asyncio` 理论资源利用率更高（单线程事件循环，无线程切换开销）。**放弃原因**：代码复杂度高（所有 I/O 需 `await`，错误处理和调试比同步代码难），`tqdm` 与 `asyncio` 的集成不如同步代码自然。对于 MVP 阶段"做对"比"做到极致"更重要。

### 备选方案 C：Rust + reqwest + tokio

性能最佳、内存安全。**放弃原因**：学习曲线最高，所有权机制 + `async-trait` 等概念对不熟悉 Rust 的人不友好。对于这个 MVP 项目，投入回报不成比例。

### 备选方案 D：仅使用 Python 标准库（urllib + threading，零第三方依赖）

完全不依赖 `requests` 和 `tqdm`，只用标准库。**放弃原因**：`urllib` 处理 Range 请求和流式下载需要手写更多样板代码，`tqdm` 提供的进度条效果远优于手写。少量第三方依赖带来的便利远大于引入依赖的成本。

## 后果

### 好处

- 开发速度快，MVP 可在 1–2 天内完成
- 代码直观，后续维护和扩展（加断点续传、加 GUI 等）的门槛低
- 生态成熟，遇到问题有大量现成参考

### 风险/代价

- Python 运行时依赖：用户环境需要有 Python 3.8+。如需分发为单文件可执行程序，后续需要加 `PyInstaller` 打包步骤（不在 MVP 范围内）
- `threading` 的线程数受 GIL 限制，但对 I/O 密集型下载影响可忽略
- `requests` + `tqdm` 两个第三方依赖，需要通过 `pyproject.toml` 声明并安装

## 不做的事

- 不打包为 Windows `.exe`（MVP 阶段用户用 `python -m` 调用，后续如有需要再加 PyInstaller）
- 不使用 `asyncio`，但在模块设计上保持接口清晰，未来如需切换并发模型不必重写核心逻辑
- 不引入 `click` / `httpx` 等更现代但不是必需的库，保持依赖最小化
