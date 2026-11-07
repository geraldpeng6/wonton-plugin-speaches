# Speaches

语音识别（Whisper）和语音合成（Kokoro、Piper），OpenAI 兼容接口，跑在自己家里的 NVIDIA
显卡上。上游项目：https://github.com/speaches-ai/speaches 。

## 这个插件做了什么

- 网页：经云吞网关访问，登录用户打开就能用（Speaches 没有账号体系，`sso: none`）。
- 接口：OpenAI 兼容，写在 `http://<这台主机的局域网地址>:8300`，主要端点：

  | 端点 | 用途 |
  | --- | --- |
  | `POST /v1/audio/transcriptions` | 语音转文字 |
  | `POST /v1/audio/translations` | 语音翻译成英文 |
  | `POST /v1/audio/speech` | 文字转语音 |
  | `GET /v1/models` | 列出可用模型 |

  局域网端口默认关着，在插件设置里打开；局域网接口不经过网关、没有密码（和 Ollama 的
  API 同一做法），只在家里网络用。

  接进云吞的模型网关（以后）：Speaches 不是文本模型服务，网关接入要么作为 magpie 的
  OpenAI 兼容 provider（`http://<主机>:8300/v1`，语音端点对文本请求无用），要么等网关
  支持 audio 端点路由；在那之前 Bot 想用语音，经 MCP 工具调本插件的接口。

## 模型

模型不打进镜像，第一次用到时从 Hugging Face 下载到插件数据目录，空闲后自动卸载。

| 默认模型 | 用途 | 下载大小 | 显存占用 |
| --- | --- | --- | --- |
| `Systran/faster-whisper-large-v3`（别名 `whisper-1`） | 识别 | 约 3.1 GB | 约 4 GB |
| `speaches-ai/Kokoro-82M-v1.0-ONNX`（别名 `tts-1`） | 合成 | 约 330 MB | 约 1 GB |

两个默认模型加起来 6 GB 显存够用，32 GB 显存可以再叠更大的模型。显存数字为 1×RTX 5090
实测（见 PR），换卡按比例估。模型清单见上游文档：https://speaches.ai 。

## 镜像

上游官方镜像只发到 CUDA 12.6（ghcr.io/speaches-ai/speaches 的 `*-cuda` 标签），RTX 50
系（sm_120）需要 CUDA 12.8 的运行库，所以 `images/speaches/` 里有一份自建 Dockerfile：
上游 Dockerfile 只改两处——基础镜像换成官方 `nvidia/cuda:12.8.1-base-ubuntu24.04`，源码
从上游仓库按提交号取；不改上游代码和依赖锁。发布时由 `scripts/publish.sh` 构建推送。
本机构建：

```bash
docker build -t ghcr.io/geraldpeng6/speaches:0.1.0 images/speaches
```

## 图标

上游没有官方标志，`icon.svg` 为本插件原创绘制，许可见 [NOTICE](NOTICE)。
