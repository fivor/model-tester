# 模型测试器 / Model Tester

[English](#english) | [中文](#中文)

---
来L站[https://linux.do/](https://linux.do/)
## 中文

一个单页静态 Web 应用，用于测试 OpenAI 兼容 API 端点。无需后端，无需构建，直接浏览器打开即用。

**在线体验：** https://yuanzhi-yw.github.io/model-tester/

### 功能

- **多渠道管理** — 添加多个 API 端点（名称、接口地址、密钥），支持导入/导出 JSON
- **Provider 预设** — 支持 OpenAI-compatible、Azure OpenAI、OpenRouter、Ollama、LM Studio 和自定义路径，避免固定强制补 `/v1`
- **密钥保存模式** — API Key 可选择本地保存或仅本次会话保存，并支持一键清除全部密钥
- **模型发现** — 从每个渠道拉取 `/models` 列表，汇总展示所有可用模型
- **Benchmark 测试** — 流式请求 `/chat/completions`，支持多轮重复测试，统计平均耗时、p95、TTFT、Token/s、成功率
- **工具调用测试** — 测试 Function Calling 能力，支持 `tool_choice` 强制调用 → 自动回退
- **自定义评测套件** — 支持导入/导出 JSON 评测用例，断言类型包含 contains、not_contains、regex、exact、json_path
- **兼容性矩阵** — 汇总 `/models`、流式、usage、工具调用、JSON mode、响应头可读性等兼容状态
- **取消与重跑** — 长任务可取消，并可只重跑失败/部分失败的模型
- **安全导出** — 渠道配置支持普通导出和不含 API Key 的安全导出
- **并发控制** — 可配置并发数（1 / 3 / 5 / 10），批量测试更快
- **历史记录** — 测试结果持久化到 `localStorage`，模型列表显示彩色历史点
- **导出结果** — 支持 JSON 和 CSV 格式下载
- **中英文切换** — 界面支持中文 / English，偏好自动保存
- **明暗主题** — 支持浅色 / 深色主题，偏好自动保存

### 使用方法

#### 第一步：添加渠道

1. 打开页面，在「渠道」区域填写：
   - **服务商**：选择 Provider 预设。OpenAI-compatible / OpenRouter / Ollama / LM Studio 会按需补 `/v1`；Azure OpenAI 和自定义路径不会强制补路径
   - **名称**：自定义名称，如 `OpenAI`、`我的代理`
   - **接口地址**：API 的 base URL，如 `https://api.openai.com/v1`
   - **密钥**：API Key，如 `sk-...`
   - **密钥保存**：选择「本地」会写入 `localStorage`；选择「会话」仅保存在当前标签页会话中
2. 点击「添加渠道」可添加多个端点同时测试
3. 点击「导出」可将渠道配置保存为 JSON；点击「导入」可从 JSON 文件恢复

#### 第二步：获取模型

点击「**获取模型**」按钮，工具会请求每个渠道的 `/models` 接口，汇总所有可用模型。

- 多渠道时，顶部会出现渠道筛选标签，可按渠道过滤模型列表
- 点击「仅文本」可自动过滤掉图像、音频、嵌入等非文本模型

#### 第三步：选择并测试

1. 勾选要测试的模型（支持全选 / 全不选 / 仅文本）
2. 右下角选择并发数（默认 3）
3. 点击「**测试选中**」开始测试
4. 测试过程中可点击「**取消**」中断；测试结束后可点击「**重跑失败项**」只重试失败或部分失败的模型

每个模型会依次进行：
- **Benchmark 测试**：发送 `"Say exactly: hello"`，按设置轮次重复测试，记录 TTFT、总延迟、Token/s、p95 和成功率
- **工具调用测试**：发送天气查询，强制调用 `get_weather` 工具，验证是否返回 tool_calls
- **JSON mode 测试**：使用 `response_format: { type: "json_object" }` 验证 JSON 输出兼容性
- **自定义 Eval**：按当前评测套件逐条发送 prompt，并用断言计算通过率和得分

#### 自定义评测套件

「评测套件」区域可导入/导出 JSON。格式示例：

```json
{
  "version": 1,
  "name": "My eval suite",
  "cases": [
    {
      "name": "exact_hello",
      "prompt": "Reply with exactly this word and no punctuation: hello",
      "assert": { "type": "regex", "value": "^hello$" },
      "maxTokens": 20,
      "weight": 1
    },
    {
      "name": "json_shape",
      "prompt": "Return only JSON: {\"ok\": true}",
      "assert": { "type": "json_path", "path": "ok", "value": true },
      "maxTokens": 60,
      "weight": 1
    }
  ]
}
```

#### 第四步：查看结果

结果表格显示每个模型的：
- **聊天**：通过 / 失败 + TTFT + 总延迟 + Token 用量 + 响应预览
- **工具**：通过 / 部分 / 失败 + 函数名 + 参数 + 耗时
- **Benchmark**：平均总耗时、p95、平均 TTFT、Token/s、成功轮次
- **兼容性矩阵**：按模型和渠道展示流式、usage、tools、JSON、headers 等能力

点击「导出 JSON」或「导出 CSV」可下载完整结果。

### 注意事项

- **跨域（CORS）**：浏览器直接请求第三方 API 可能被 CORS 策略拦截。解决方案：
  - 使用支持跨域的 API 代理
  - 安装浏览器 CORS 扩展（仅用于开发测试）
  - 在本地起一个反向代理
- **数据安全**：渠道信息（含 API Key）保存在浏览器 `localStorage` 中，导出文件包含明文密钥，请妥善保管
- **会话密钥模式**：选择「会话」时，API Key 不会写入 `localStorage`，刷新当前会话内可用，关闭标签页后失效

---

## English

A single-page static web app for testing OpenAI-compatible API endpoints. No backend, no build step — just open `index.html` in a browser.

**Live demo:** https://yuanzhi-yw.github.io/model-tester/

### Features

- **Channel Management** — Add multiple API endpoints (name, base URL, API key). Import/export as JSON
- **Provider Presets** — OpenAI-compatible, Azure OpenAI, OpenRouter, Ollama, LM Studio, and custom-path modes
- **Key Storage Mode** — Store API keys locally or only for the current session, with one-click key clearing
- **Model Discovery** — Fetch `/models` from each channel and list all available models
- **Benchmark Test** — Streaming `/chat/completions` with repeated runs, average latency, p95, TTFT, token/s, and success-rate metrics
- **Tool Calling Test** — Tests Function Calling with forced `tool_choice` → auto fallback
- **Custom Eval Suite** — Import/export JSON eval cases with contains, not_contains, regex, exact, and json_path assertions
- **Compatibility Matrix** — Summarizes `/models`, streaming, usage, tools, JSON mode, and readable response headers
- **Cancel and Rerun** — Cancel long-running tests and rerun only failed/partial items
- **Safe Export** — Export channel config with or without API keys
- **Concurrency Control** — Configurable pool size (1 / 3 / 5 / 10) for batch testing
- **Test History** — Results persisted in `localStorage`, shown as colored dots in the model list
- **Export Results** — Download as JSON or CSV
- **i18n** — Switch between Chinese and English; preference is saved
- **Light/Dark Theme** — Toggle between themes; preference is saved

### How to Use

#### Step 1: Add Channels

1. Fill in the **Channels** section:
   - **Name**: A label for this endpoint, e.g. `OpenAI`, `My Proxy`
   - **Base URL**: The API base URL, e.g. `https://api.openai.com/v1`
   - **API Key**: Your API key, e.g. `sk-...`
2. Click **Add channel** to add more endpoints for parallel testing
3. Use **Export** to save your channel config as JSON; **Import** to restore it

#### Step 2: Fetch Models

Click **Fetch models** — the tool calls `/models` on each channel and aggregates the results.

- With multiple channels, filter chips appear so you can narrow the model list by channel
- Click **Text only** to auto-deselect image, audio, embedding, and other non-text models

#### Step 3: Select and Test

1. Check the models you want to test (Select all / Deselect all / Text only)
2. Choose concurrency in the bottom-right (default: 3)
3. Click **Test selected**

Each model runs two tests:
- **Benchmark**: Sends `"Say exactly: hello"` for the configured number of runs and records TTFT, total latency, token/s, p95, and success rate
- **Tools**: Sends a weather query with a forced `get_weather` tool call, checks for `tool_calls` in the response
- **JSON mode**: Sends a `response_format: { type: "json_object" }` request and validates parseable JSON output

#### Step 4: View Results

The results table shows for each model:
- **Chat**: Pass/Fail + TTFT + total latency + token usage + response preview
- **Tools**: Pass/Partial/Fail + function name + arguments + latency

Click **Export JSON** or **Export CSV** to download the full results.

### Notes

- **CORS**: Browser requests to third-party APIs may be blocked by CORS policy. Workarounds:
  - Use an API proxy that allows cross-origin requests
  - Install a browser CORS extension (for development only)
  - Run a local reverse proxy
- **Security**: Channel data (including API keys) is stored in `localStorage`. Exported files contain keys in plain text — handle with care.

### License

MIT
