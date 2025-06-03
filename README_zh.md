# Gemini Fullstack LangGraph 快速入门

本项目演示了一个全栈应用程序，采用 React 前端和 LangGraph 驱动的后端代理。该代理旨在通过动态生成搜索词条、使用 Google 搜索查询网络、反思结果以识别知识差距，并迭代优化其搜索，直到能够提供有充分依据且附带引用的答案，从而对用户查询进行全面研究。此应用程序可作为使用 LangGraph 和 Google Gemini 模型构建研究增强型对话式人工智能的示例。

![Gemini Fullstack LangGraph](./app.png)

## 特性

- 💬 全栈应用程序，包含 React 前端和 LangGraph 后端。
- 🧠 由 LangGraph 代理提供支持，用于高级研究和对话式人工智能。
- 🔍 使用 Google Gemini 模型动态生成搜索查询。
- 🌐 通过 Google Search API 集成网络研究。
- 🤔 反思性推理，以识别知识差距并优化搜索。
- 📄 生成包含所收集来源引用的答案。
- 🔄 在开发过程中为前端和后端开发提供热重载功能。

## 项目结构

项目分为两个主要目录：

- `frontend/`：包含使用 Vite 构建的 React 应用程序。
- `backend/`：包含 LangGraph/FastAPI 应用程序，包括研究代理逻辑。

## 入门：开发和本地测试

请按照以下步骤在本地运行应用程序以进行开发和测试。

**1. 先决条件：**

- Node.js 和 npm (或 yarn/pnpm)
- Python 3.8+
- **`GEMINI_API_KEY`**：后端代理需要 Google Gemini API 密钥。
    1. 导航到 `backend/` 目录。
    2. 通过复制 `backend/.env.example` 文件创建一个名为 `.env` 的文件。
    3. 打开 `.env` 文件并添加您的 Gemini API 密钥：`GEMINI_API_KEY="YOUR_ACTUAL_API_KEY"`

**2. 安装依赖项：**

**后端：**

```bash
cd backend
pip install .
```

**前端：**

```bash
cd frontend
npm install
```

**3. 运行开发服务器：**

**后端和前端：**

```bash
make dev
```
这将运行后端和前端开发服务器。 打开浏览器并导航到前端开发服务器 URL (例如, `http://localhost:5173/app`)。

_或者，您可以分别运行后端和前端开发服务器。对于后端，在 `backend/` 目录中打开一个终端并运行 `langgraph dev`。后端 API 将在 `http://127.0.0.1:2024` 上可用。它还将打开一个浏览器窗口到 LangGraph UI。对于前端，在 `frontend/` 目录中打开一个终端并运行 `npm run dev`。前端将在 `http://localhost:5173` 上可用。_

## 后端代理工作原理 (高级概述)

后端的的核心是在 `backend/src/agent/graph.py` 中定义的 LangGraph 代理。它遵循以下步骤：

![Agent Flow](./agent.png)

1. **生成初始查询：** 根据您的输入，它使用 Gemini 模型生成一组初始搜索查询。
2. **网络研究：** 对于每个查询，它使用带有 Google Search API 的 Gemini 模型来查找相关的网页。
3. **反思和知识差距分析：** 代理分析搜索结果以确定信息是否充分或是否存在知识差距。它使用 Gemini 模型进行此反思过程。
4. **迭代优化：** 如果发现差距或信息不足，它会生成后续查询并重复网络研究和反思步骤 (最多达到配置的最大循环次数)。
5. **最终确定答案：** 一旦研究被认为足够，代理会将收集到的信息综合成一个连贯的答案，包括来自网络来源的引用，并使用 Gemini 模型。

## 部署

在生产环境中，后端服务器提供优化的静态前端构建。LangGraph 需要一个 Redis 实例和一个 Postgres 数据库。Redis 用作发布/订阅代理，以启用来自后台运行的实时输出流。Postgres 用于存储助手、线程、运行、持久化线程状态和长期内存，并以“精确一次”语义管理后台任务队列的状态。有关如何部署后端服务器的更多详细信息，请参阅 [LangGraph 文档](https://langchain-ai.github.io/langgraph/concepts/deployment_options/)。以下是如何构建包含优化前端构建和后端服务器的 Docker 镜像并通过 `docker-compose` 运行它的示例。

_注意：对于 docker-compose.yml 示例，您需要一个 LangSmith API 密钥，您可以从 [LangSmith](https://smith.langchain.com/settings) 获取。_

_注意：如果您没有运行 docker-compose.yml 示例或将后端服务器暴露给公共互联网，请更新 `frontend/src/App.tsx` 文件中的 `apiUrl` 为您的主机。目前，`apiUrl` 对于 docker-compose 设置为 `http://localhost:8123`，对于开发设置为 `http://localhost:2024`。_

**1. 构建 Docker 镜像：**

   从**项目根目录**运行以下命令：
   ```bash
   docker build -t gemini-fullstack-langgraph -f Dockerfile .
   ```
**2. 运行生产服务器：**

   ```bash
   GEMINI_API_KEY=<your_gemini_api_key> LANGSMITH_API_KEY=<your_langsmith_api_key> docker-compose up
   ```

打开浏览器并导航到 `http://localhost:8123/app/` 以查看应用程序。API 将在 `http://localhost:8123` 上可用。

## 使用的技术

- [React](https://reactjs.org/) (与 [Vite](https://vitejs.dev/) 一起使用) - 用于前端用户界面。
- [Tailwind CSS](https://tailwindcss.com/) - 用于样式设计。
- [Shadcn UI](https://ui.shadcn.com/) - 用于组件。
- [LangGraph](https://github.com/langchain-ai/langgraph) - 用于构建后端研究代理。
- [Google Gemini](https://ai.google.dev/models/gemini) - 用于查询生成、反思和答案综合的 LLM。

## 许可证

本项目根据 Apache License 2.0 获得许可。有关详细信息，请参阅 [LICENSE](LICENSE) 文件。
