# 数字人朱嘉明项目 - AI功能开发文档

## 项目概述

本项目是一个以朱嘉明教授为原型的数字人交互系统，主要包含大语言模型对话、语音合成、声音复刻等AI功能。

## 技术架构

```
AI功能架构
├── 对话引擎 (阿里云百炼API)
│   ├── 普通对话
│   ├── 知识库检索对话
│   └── 流式输出支持
├── 语音合成 (多服务商)
│   ├── 火山引擎TTS (主要)
│   ├── 字节跳动MegaTTS
│   └── 讯飞语音合成
└── 声音复刻
    ├── 字节跳动声音复刻
    ├── 讯飞声音训练
    └── 本地音频处理
```

## 环境要求

### Python版本
- Python 3.8+

### 依赖包
```txt
Flask==2.3.3
Werkzeug==2.3.7
requests==2.31.0
dashscope==1.20.11
websocket-client
requests-toolbelt
crypto
```

### API服务商账号
1. **阿里云百炼** - 大语言模型服务
2. **火山引擎** - TTS语音合成服务
3. **字节跳动** - MegaTTS声音复刻服务
4. **讯飞开放平台** - 语音训练和合成服务

## 核心功能模块

### 1. 阿里云百炼API集成

#### 配置文件 (API.py)
```python
# 阿里云百炼配置
APP_ID = "your_app_id"  # 应用ID
API_KEY = "your_api_key"  # API密钥
KNOWLEDGE_BASE_IDS = ["kb_id_1", "kb_id_2"]  # 知识库ID列表
SYSTEM_PROMPT = "你是朱嘉明教授..."  # 系统提示词
MODEL = "deepseek-v3"  # 使用的模型
```

#### 主要功能
- **普通对话**: `chat_completion()` - 支持多轮对话
- **知识库对话**: `knowledge_chat()` - 基于专门知识库的对话
- **流式输出**: 支持实时流式响应

#### 接口说明
```python
# 普通对话
POST /chat
{
    "message": "用户消息",
    "use_knowledge_base": false
}

# 知识库对话  
POST /chat
{
    "message": "用户消息", 
    "use_knowledge_base": true
}
```

### 2. 火山引擎TTS语音合成

#### 配置文件 (tts_config.py)
```python
# 火山引擎TTS配置
APPID = "your_app_id"
ACCESS_TOKEN = "your_access_token"  
CLUSTER = "volcano_tts"
VOICE_TYPE = "zjm_cloned_voice_id"  # 朱嘉明克隆音色ID
ENCODING = "mp3"
SPEED_RATIO = 1.0
VOLUME_RATIO = 1.0
PITCH_RATIO = 1.0
TTS_API_URL = "wss://openspeech.bytedance.com/api/v1/tts/ws_binary"
```

#### 主要功能
- **WebSocket连接**: 使用websocket-client库建立长连接
- **二进制协议**: 处理火山引擎专用的二进制协议格式
- **流式合成**: 实时接收合成的音频流
- **音频保存**: 自动保存为MP3文件

#### 核心类: TTSWebSocketClient
```python
class TTSWebSocketClient:
    def __init__(self):
        self.audio_data = bytearray()
        self.audio_file_path = None
        self.ws = None
        
    async def synthesize_speech(self, text, output_path=None):
        # 异步语音合成主方法
```

#### 协议格式
```
消息头格式 (4字节):
- version (4 bits) + header size (4 bits) = 0x11
- message type (4 bits) + message flags (4 bits) = 0x10  
- serialization method (4 bits) + compression (4 bits) = 0x11
- reserved data = 0x00
```

### 3. 字节跳动声音复刻 (MegaTTS)

#### 配置
```python
HOST = "https://openspeech.bytedance.com"
RESOURCE_ID = "volc.megatts.voiceclone"
```

#### 主要功能
- **音频上传**: `train_voice()` - 上传训练音频
- **训练状态查询**: `get_train_status()` - 查询训练进度
- **支持格式**: WAV, MP3, OGG, FLAC
- **文件限制**: 最大10MB

#### API接口
```python
# 训练音色
def train_voice(appid, token, audio_path, spk_id):
    url = HOST + "/api/v1/mega_tts/audio/upload"
    # 音频Base64编码上传

# 查询状态
def get_train_status(appid, token, spk_id):
    url = HOST + "/api/v1/mega_tts/status"
```

### 4. 讯飞语音训练

#### 配置
```python
appid = 'your_xunfei_appid'
apikey = 'your_xunfei_apikey'  
```

#### 主要功能
- **获取训练文本**: `getText()` - 获取官方训练文本列表
- **创建训练任务**: `createTask()` - 创建音色训练任务
- **音频上传**: 
  - `addAudio()` - 通过URL上传
  - `addAudiofromPC()` - 本地文件上传
- **任务提交**: `submitTask()` - 提交训练任务
- **进度查询**: `getProcess()` - 查询训练状态

#### 训练流程
```python
# 1. 获取训练文本
voiceTrain.getText()

# 2. 上传对应音频
voiceTrain.addAudiofromPC(textId=5001, textSegId=1, path='audio.wav')

# 3. 提交训练
voiceTrain.submitTask()

# 4. 查询结果
response = voiceTrain.getProcess()
```

## 部署配置

### 1. 环境变量设置
```bash
# 阿里云百炼
export DASHSCOPE_API_KEY="your_bailian_api_key"

# 火山引擎
export VOLC_APPID="your_volc_appid"
export VOLC_ACCESS_TOKEN="your_volc_token"

# 讯飞
export XUNFEI_APPID="your_xunfei_appid"
export XUNFEI_API_KEY="your_xunfei_key"
```

### 2. 目录结构
```
后端/
├── zhujiaming_chat/
│   ├── app.py              # Flask主应用
│   ├── API.py              # 百炼API配置
│   ├── bailian_api.py      # 百炼API封装
│   ├── tts_websocket.py    # 火山引擎TTS客户端
│   ├── tts_config.py       # TTS配置
│   └── requirements.txt    # 依赖包
├── voice_clone_app/        # 声音复刻应用
└── 声音复刻/
    ├── train.py            # 讯飞声音训练
    └── tts.py              # TTS合成
```

### 3. 启动服务
```bash
cd 后端/zhujiaming_chat/
pip install -r requirements.txt
python app.py
```

## API接口文档

### 1. 对话接口
```
POST /chat
Content-Type: application/json

{
    "message": "你好",
    "use_knowledge_base": false
}

Response (流式):
data: {"error": false, "text": "你好...", "session_id": "xxx"}
```

### 2. 语音合成接口
```
POST /synthesize-speech
Content-Type: application/json

{
    "text": "要合成的文本"
}

Response:
{
    "error": false,
    "audio_url": "/static/audio/tts_xxx.mp3",
    "message": "语音合成成功"
}
```

### 3. 声音复刻接口
```
POST /upload (MegaTTS)
Form Data:
- audio_file: 音频文件
- appid: 应用ID
- token: 访问令牌
- speaker_id: 说话人ID
```

## 模型配置

### 1. 大语言模型
- **模型**: DeepSeek-V3
- **提供商**: 阿里云百炼
- **特点**: 支持知识库检索、流式输出

### 2. 语音合成模型
- **主模型**: 火山引擎TTS
- **声音**: 朱嘉明教授克隆音色
- **格式**: MP3, 16kHz

### 3. 声音复刻模型
- **字节跳动MegaTTS**: 
  - 支持快速克隆
  - 仅需少量音频样本
  - 自动音质增强
- **讯飞声音训练**:
  - 需要官方文本录音
  - 训练时间较长
  - 音质稳定

## 常见问题和解决方案

### 1. TTS连接问题
```python
# 检查WebSocket连接
if not self.ws:
    print("WebSocket连接失败")
    
# 检查认证格式
headers = [f"Authorization: Bearer; {ACCESS_TOKEN}"]
```

### 2. 音频质量优化
- 使用WAV格式，16kHz采样率
- 确保音频时长在合理范围内
- 避免背景噪音

### 3. API调用限制
- 注意各服务商的调用频率限制
- 实现重试机制
- 监控API使用量

## 性能优化建议

### 1. 音频缓存
- 缓存常用TTS合成结果
- 使用Redis存储音频URL
- 设置合理的过期时间

### 2. 异步处理
- 使用asyncio处理TTS任务
- WebSocket连接池管理
- 避免阻塞主线程

### 3. 错误处理
- 完善的异常捕获机制
- 优雅降级策略
- 日志记录和监控

## 后续开发建议

### 1. 功能扩展
- 支持更多语音风格
- 添加情感控制
- 实现语音转换

### 2. 性能提升
- 增加GPU加速支持
- 优化模型推理速度
- 减少内存占用

### 3. 用户体验
- 实时语音交互
- 多语言支持
- 个性化设置

---

*注意：本文档中的API密钥等敏感信息需要根据实际部署环境进行配置。*
