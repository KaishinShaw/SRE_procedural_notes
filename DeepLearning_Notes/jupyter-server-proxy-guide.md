# 技术笔记：精通 jupyter-server-proxy：从简单 UI 到 API 服务

> **版本**: 2.0
> **日期**: 2025-07-31
> **作者**: Hydraulik

## 1. 问题背景

在远程服务器上进行数据科学或机器学习开发时，我们经常需要同时运行多个服务。一个典型的场景是：

*   **Jupyter 环境**: 一个 JupyterLab/Notebook 实例运行在服务器的公开端口上，作为我们的主开发环境。
*   **内部 Web 应用**: 同时，我们可能需要启动其他内部 Web 应用，例如：
    *   一个简单的 Web 页面用于展示结果。
    *   一个 TensorBoard 或 MLflow 用于模型监控。
    *   一个**原型 API 服务** (使用 FastAPI/Flask) 用于测试模型端点。
*   **安全限制**: 出于安全考虑，这些内部应用的端口（如 `8700`, `8701`）通常不会对公网开放，只允许服务器本机 (`localhost`) 访问。

**核心挑战**: 如何在本地浏览器中，通过已公开的 Jupyter 环境，安全、便捷地访问到这些未公开的内部 Web 应用？

## 2. 核心原理与解决方案

**核心原理**: 利用 Jupyter 服务作为跳板。由于 Jupyter 服务和内部 Web 应用都运行在同一台服务器上，它们之间可以通过 `localhost` 进行通信。我们只需要一个机制，让 Jupyter 服务能将我们从外部发来的请求“转发”给内部服务。

**最佳解决方案**: 使用官方推荐的扩展 `jupyter-server-proxy`。

`jupyter-server-proxy` 是一个 Jupyter Server 的扩展，它能将 Jupyter 变成一个轻量级的反向代理。它会监听特定格式的 URL 请求，并将这些请求代理到服务器本地的其他端口上，然后将响应返回给客户端浏览器。

## 3. 基础操作指南

### 步骤一：安装扩展

通过 SSH 登录到你的远程服务器，并激活运行 Jupyter 的 Python 环境。

```bash
# 使用 pip 安装
pip install jupyter-server-proxy

# 或者，如果你使用 conda/mamba
# conda install -c conda-forge jupyter-server-proxy
```

> **⚠️ 重要提示**：安装完成后，**必须完全停止并重启**你的 Jupyter 服务。这是为了让 Jupyter Server 能够加载新安装的扩展。仅仅刷新浏览器是无效的。

### 步骤二：构建代理访问 URL

`jupyter-server-proxy` 的 URL 格式遵循特定规则。

*   **标准部署 (根路径)**: `http://<SERVER_IP>:<PORT>/proxy/<TARGET_PORT>/`
*   **子路径部署 (如您的 gpuhub 环境)**: `https://<HOSTNAME>:<PORT>/<BASE_URL>/proxy/<TARGET_PORT>/`

**您的环境示例**:
*   **Jupyter 地址**: `https://a485820-9cc0-2d969d3b.westc.gpuhub.com:8443/jupyter/lab`
*   **基础 URL**: `/jupyter/`
*   **访问 8700 端口的代理 URL**: `.../jupyter/proxy/8700/`

## 4. 实战演练：两大经典用例

我们将通过两个由浅入深的例子，展示 `jupyter-server-proxy` 的强大功能。

### 4.1 用例一：访问简单的 Web UI

这是最基础的验证，证明代理功能已正常工作。

**A. 准备一个本地 Web 服务 (端口 `8700`)**
在服务器上创建 `simple_server_8700.py` 并运行它。

```python
# simple_server_8700.py
import http.server, socketserver
PORT = 8700
class Handler(http.server.SimpleHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200); self.send_header("Content-type", "text/html; charset=utf-8"); self.end_headers()
        html = f'<!DOCTYPE html><html><head><title>端口 {PORT} 测试</title></head><body style="text-align:center;font-family:sans-serif;"><h1>✅ UI 代理验证成功！</h1><p>这个页面运行在 <strong>localhost:{PORT}</strong>，通过 jupyter-server-proxy 成功访问。</p></body></html>'
        self.wfile.write(html.encode('utf-8'))
with socketserver.TCPServer(("", PORT), Handler) as httpd:
    print(f"UI 服务已在 http://localhost:{PORT} 启动...")
    httpd.serve_forever()
``````bash
python3 simple_server_8700.py
```

**B. 验证访问**
在 JupyterLab 中新建 Notebook，运行以下代码，即可在输出区看到页面。

```python
from IPython.display import IFrame
# 使用包含 base_url 的绝对路径
display(IFrame(src='/jupyter/proxy/8700/', width='100%', height=150))
```

---

### 4.2 用例二：访问自定义 API 服务器 (高级用法)

这是一个更强大、更实用的场景。我们用 FastAPI 创建一个 API 服务，并通过代理来调用它。

> **概念澄清**: 我们使用 FastAPI 等框架来 **创建** API 服务，而使用 `jupyter-server-proxy` 来 **访问** 它。

**A. 安装 API 框架 (FastAPI & Uvicorn)**
在您的服务器 Python 环境中安装必要的库。

```bash
pip install "fastapi[all]"
```

**B. 编写 API 服务器代码 (端口 `8701`)**
在服务器上创建 `my_api_server.py` 文件。

```python
# my_api_server.py
import uvicorn
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class TextData(BaseModel):
    text: str
    id: int

@app.get("/")
def read_root():
    return {"message": "欢迎来到我的API服务器！它正通过 jupyter-server-proxy 运行。"}

@app.get("/api/v1/greet")
def greet_user(name: str = "Guest"):
    return {"greeting": f"你好, {name}!"}

@app.post("/api/v1/analyze")
def analyze_text(data: TextData):
    word_count = len(data.text.split())
    return {
        "received_id": data.id,
        "analysis_result": { "word_count": word_count, "is_long_text": word_count > 10 }
    }

if __name__ == "__main__":
    uvicorn.run(app, host="127.0.0.1", port=8701)
```

**C. 运行 API 服务器**
在 SSH 终端中启动服务，并保持其运行。

```bash
python3 my_api_server.py
```
您将看到 Uvicorn 在 `http://127.0.0.1:8701` 上启动服务。

**D. 验证 API 访问**

*   **方法一：访问交互式 API 文档**
    FastAPI 会自动生成 API 文档。通过代理访问它：
    `https://a485820-9cc0-2d969d3b.westc.gpuhub.com:8443/jupyter/proxy/8701/docs`
    你可以在这个页面上直接测试你的 API！

*   **方法二：在 Notebook 中调用 API**
    这是最强大的工作流。在 Jupyter Notebook 中使用 `requests` 库调用 API。

    ```python
    import requests
    import json

    # 构建 API 的代理基础 URL
    api_base_url = "https://a485820-9cc0-2d969d3b.westc.gpuhub.com:8443/jupyter/proxy/8701"

    # 1. 测试 GET 端点
    print("--- 测试 GET /api/v1/greet ---")
    try:
        response_get = requests.get(f"{api_base_url}/api/v1/greet", params={"name": "远程开发者"})
        response_get.raise_for_status()
        print(json.dumps(response_get.json(), indent=2, ensure_ascii=False))
    except Exception as e:
        print(f"请求失败: {e}")

    print("\n--- 测试 POST /api/v1/analyze ---")
    # 2. 测试 POST 端点
    try:
        post_data = {"text": "这是一个通过 Jupyter Notebook 发送的示例文本。", "id": 123}
        response_post = requests.post(f"{api_base_url}/api/v1/analyze", json=post_data)
        response_post.raise_for_status()
        print(json.dumps(response_post.json(), indent=2, ensure_ascii=False))
    except Exception as e:
        print(f"请求失败: {e}")
    ```

## 5. 常见问题 (FAQ)

*   **问：访问代理 URL 显示 404 Not Found？**
    *   答：检查三点：1) 是否在安装扩展后**重启了 Jupyter 服务**？ 2) 你的目标服务（如 API server）是否确实在运行？ 3) URL 路径是否正确（特别是对于子路径部署）？

*   **问：我可以使用 Flask, Streamlit 或其他框架吗？**
    *   答：**完全可以**。`jupyter-server-proxy` 是应用无关的。只要你的应用在服务器的某个 `localhost` 端口上提供 HTTP 服务，就可以通过同样的方式进行代理。

## 6. 结论

`jupyter-server-proxy` 是一个极其强大的工具，它将 Jupyter 从一个单纯的笔记本环境，**提升为一个安全的、统一的远程开发网关**。通过它，开发者可以在不暴露额外端口的情况下，无缝地访问、测试和集成各种内部 Web 服务和 API，极大地提高了远程开发工作流的效率和安全性。