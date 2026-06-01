# Windows 从零安装 Ollama + Qwen 最新 7B + OpenAI 兼容接口完整指南

> 适用环境：Windows 10/11 + NVIDIA RTX 4070 Ti  
> 默认服务地址：`http://localhost:11434`  
> OpenAI 兼容地址：`http://localhost:11434/v1`

## 目录

- [1. 前置条件检查](#1-前置条件检查)
- [2. 第一步：安装 NVIDIA 驱动](#2-第一步安装-nvidia-驱动)
- [3. 第二步：安装 Ollama](#3-第二步安装-ollama)
- [4. 第三步：拉取 Qwen 7B 模型](#4-第三步拉取-qwen-7b-模型)
- [5. 第四步：启动 Ollama 服务](#5-第四步启动-ollama-服务)
- [6. 第五步：测试本地运行](#6-第五步测试本地运行)
- [7. 第六步：配置 OpenAI 兼容接口](#7-第六步配置-openai-兼容接口)
- [8. 第七步：Python 代码调用示例](#8-第七步python-代码调用示例)
- [9. 第八步：故障排查](#9-第八步故障排查)
- [10. 附录](#10-附录)

---

## 1. 前置条件检查

### 1.1 检查 NVIDIA 驱动

在 PowerShell 或 CMD 执行：

```powershell
nvidia-smi
```

预期结果：
- 能看到 `NVIDIA-SMI` 版本信息
- 能看到 `GeForce RTX 4070 Ti`
- 驱动版本正常显示（`Driver Version`）

截图建议：
- `nvidia-smi` 完整输出窗口（建议保存为 `01-nvidia-smi.png`）

### 1.2 检查系统配置要求

建议最低配置：
- Windows 10/11（64 位）
- 内存：16GB 及以上（建议 32GB）
- 显存：RTX 4070 Ti（12GB）
- 可用磁盘空间：至少 20GB（模型下载 + 缓存）

检查命令：

```powershell
systeminfo
```

截图建议：
- `OS Name`、`Total Physical Memory`（建议保存为 `02-systeminfo.png`）

### 1.3 验证硬件支持（RTX 4070 Ti）

```powershell
wmic path win32_VideoController get name
```

预期应包含：
- `NVIDIA GeForce RTX 4070 Ti`

截图建议：
- 显卡型号查询结果（建议保存为 `03-gpu-model.png`）

---

## 2. 第一步：安装 NVIDIA 驱动

### 2.1 官网下载链接

- NVIDIA 驱动下载页：<https://www.nvidia.com/Download/index.aspx>

选择：
- Product Type: `GeForce`
- Product Series: `GeForce RTX 40 Series`
- Product: `GeForce RTX 4070 Ti`
- OS: 你的 Windows 版本

截图建议：
- 驱动筛选页面（`04-driver-select.png`）
- 下载结果页面（`05-driver-download.png`）

### 2.2 安装步骤

1. 双击下载的驱动安装包（`.exe`）
2. 选择 `NVIDIA Graphics Driver and GeForce Experience`（或仅驱动）
3. 选择 `Express`（推荐）或 `Custom`
4. 等待安装完成并重启

截图建议：
- 安装向导页面（`06-driver-installer.png`）
- 安装完成页面（`07-driver-finish.png`）

### 2.3 验证安装成功

重启后再次执行：

```powershell
nvidia-smi
```

确认驱动版本和 GPU 信息正常显示。

截图建议：
- 重启后 `nvidia-smi` 输出（`08-driver-verified.png`）

---

## 3. 第二步：安装 Ollama

### 3.1 官网下载链接

- Ollama 官网：<https://ollama.com/download/windows>

截图建议：
- 下载页面（`09-ollama-download-page.png`）

### 3.2 Windows 安装步骤

1. 下载 `OllamaSetup.exe`
2. 双击安装，按向导下一步
3. 安装完成后启动 Ollama（系统托盘可见图标）

截图建议：
- 安装器页面（`10-ollama-installer.png`）
- 托盘图标页面（`11-ollama-tray.png`）

### 3.3 验证安装成功

打开新的 PowerShell：

```powershell
ollama --version
```

预期：输出版本号。

截图建议：
- `ollama --version` 输出（`12-ollama-version.png`）

---

## 4. 第三步：拉取 Qwen 7B 模型

### 4.1 打开命令行

推荐使用 PowerShell（管理员非必须）。

### 4.2 执行拉取命令

优先尝试最新常用标签：

```powershell
ollama pull qwen3:7b
```

若标签不存在，可尝试：

```powershell
ollama pull qwen2.5:7b
```

### 4.3 下载进度

拉取时会显示分层下载进度，等待直到出现 `success` 或完成提示。

截图建议：
- 下载进度中间状态（`13-pull-progress.png`）
- 下载完成状态（`14-pull-finished.png`）

### 4.4 验证模型已安装

```powershell
ollama list
```

应看到 `qwen3:7b`（或你实际拉取的 7B 标签）。

截图建议：
- `ollama list` 输出（`15-ollama-list.png`）

---

## 5. 第四步：启动 Ollama 服务

### 5.1 确认后台运行

安装后通常已在后台运行。若需要手动启动：

```powershell
ollama serve
```

### 5.2 验证服务监听端口

```powershell
netstat -ano | findstr 11434
```

应看到 `LISTENING`。

也可直接访问模型列表接口：

```powershell
curl http://localhost:11434/v1/models
```

### 5.3 使用 nvidia-smi 验证 GPU 调用

在一个窗口保持模型调用，另开窗口执行：

```powershell
nvidia-smi -l 1
```

观察显存占用、GPU Util 是否上升。

截图建议：
- `netstat` 监听结果（`16-port-listening.png`）
- `nvidia-smi -l 1` GPU 使用变化（`17-gpu-usage.png`）

---

## 6. 第五步：测试本地运行

命令行执行：

```powershell
ollama run qwen3:7b
```

输入测试问题：

```text
你好，请用三句话介绍你自己。
```

预期：
- 模型返回连贯回答
- 首次响应可能稍慢（模型冷启动）

退出会话：

```text
/bye
```

截图建议：
- 提问与回答全过程（`18-local-chat-test.png`）

---

## 7. 第六步：配置 OpenAI 兼容接口

### 7.1 接口地址

- Base URL：`http://localhost:11434/v1`
- Chat Completions：`POST /chat/completions`
- Models：`GET /models`

### 7.2 基本参数说明

- `model`：模型名（如 `qwen3:7b`）
- `messages`：对话数组（`role` + `content`）
- `temperature`：采样温度（可选）
- `Authorization`：本地 Ollama 接口通常可省略；若你的 SDK/网关要求该字段，可传任意非空占位值。

### 7.3 curl 命令测试

#### 读取模型列表

```powershell
curl http://localhost:11434/v1/models
```

#### 发送对话请求

```powershell
curl http://localhost:11434/v1/chat/completions ^
  -H "Content-Type: application/json" ^
  -d "{\"model\":\"qwen3:7b\",\"messages\":[{\"role\":\"user\",\"content\":\"请简要介绍 RTX 4070 Ti 运行 7B 模型的优势\"}]}"
```

截图建议：
- `v1/models` 返回结果（`19-api-models.png`）
- `chat/completions` 返回结果（`20-api-chat.png`）

---

## 8. 第七步：Python 代码调用示例

### 8.1 安装 OpenAI SDK

```powershell
pip install openai
```

### 8.2 完整代码示例

保存为 `ollama_openai_test.py`：

```python
from openai import OpenAI

client = OpenAI(
    api_key="ollama",
    base_url="http://localhost:11434/v1",
)

response = client.chat.completions.create(
    model="qwen3:7b",
    messages=[
        {"role": "system", "content": "你是一个简洁、准确的中文助手。"},
        {"role": "user", "content": "请用要点说明 Windows 本地部署 LLM 的核心步骤。"},
    ],
    temperature=0.7,
)

print(response.choices[0].message.content)
```

运行：

```powershell
python .\ollama_openai_test.py
```

### 8.3 运行结果截图

截图建议：
- Python 脚本执行输出（`21-python-result.png`）

---

## 9. 第八步：故障排查

### 9.1 常见问题与解决方案

1. `ollama` 命令未找到  
   - 重新打开终端  
   - 检查是否安装成功  
   - 检查 PATH 环境变量

2. `pull` 提示模型不存在  
   - 尝试 `qwen2.5:7b`  
   - 到 Ollama 模型库确认最新可用标签

3. 模型推理速度慢、疑似未使用 GPU  
   - 升级 NVIDIA 驱动  
   - 升级 Ollama  
   - 用 `nvidia-smi` 观察负载

4. OpenAI SDK 报认证错误  
   - 保留 `api_key`，可填占位值：`ollama`

5. 端口冲突（11434 被占用）  
   - 先定位进程并释放端口

### 9.2 常用调试命令

```powershell
ollama --version
ollama list
ollama ps
netstat -ano | findstr 11434
nvidia-smi
curl http://localhost:11434/v1/models
```

---

## 10. 附录

### 10.1 一键检查脚本（可选）

```powershell
Write-Host "==== NVIDIA ===="
nvidia-smi

Write-Host "==== OLLAMA VERSION ===="
ollama --version

Write-Host "==== OLLAMA MODELS ===="
ollama list

Write-Host "==== API MODELS ===="
curl http://localhost:11434/v1/models
```

### 10.2 参考资源链接

- Ollama 下载：<https://ollama.com/download/windows>
- Ollama 模型库：<https://ollama.com/library>
- NVIDIA 驱动下载：<https://www.nvidia.com/Download/index.aspx>
- OpenAI Python SDK：<https://github.com/openai/openai-python>

---

## 截图清单（建议）

将截图保存到同目录 `images/` 文件夹（可选），并按如下命名：

1. `01-nvidia-smi.png`
2. `02-systeminfo.png`
3. `03-gpu-model.png`
4. `04-driver-select.png`
5. `05-driver-download.png`
6. `06-driver-installer.png`
7. `07-driver-finish.png`
8. `08-driver-verified.png`
9. `09-ollama-download-page.png`
10. `10-ollama-installer.png`
11. `11-ollama-tray.png`
12. `12-ollama-version.png`
13. `13-pull-progress.png`
14. `14-pull-finished.png`
15. `15-ollama-list.png`
16. `16-port-listening.png`
17. `17-gpu-usage.png`
18. `18-local-chat-test.png`
19. `19-api-models.png`
20. `20-api-chat.png`
21. `21-python-result.png`
