# MaterialSearch 代码框架与架构文档

## 项目概述

MaterialSearch 是一个本地素材搜索系统，用于扫描和索引本地的图片和视频文件，支持基于自然语言和图像的多种搜索方式。项目采用 Python + Flask 的技术栈，使用深度学习模型（CLIP）进行语义特征提取，利用 SQLAlchemy 进行数据库管理。

## 技术栈

| 技术 | 用途 | 版本范围 |
|------|------|--------|
| **Flask** | Web 框架 | >= 2.2.2 |
| **SQLAlchemy** | ORM 数据库框架 | >= 2.0.20 |
| **PyTorch** | 深度学习框架 | >= 1.13.1 |
| **Transformers** | CLIP 模型库 | >= 4.28.1 |
| **NumPy** | 科学计算库 | >= 1.20.3 |
| **OpenCV** | 视频处理库 | >= 4.7.0.68 |
| **Pillow** | 图片处理库 | >= 8.1.0 |
| **Vue.js** | 前端框架 | (通过 CDN) |

## 项目结构

```
MaterialSearch/
├── 核心模块
│   ├── main.py                 # Flask 应用主入口，API 路由定义
│   ├── config.py               # 配置管理，环境变量加载
│   ├── models.py               # 数据库模型定义，SQLAlchemy ORM
│   ├── database.py             # 数据库操作函数集合
│   ├── scan.py                 # 文件扫描与索引核心模块
│   ├── search.py               # 搜索功能实现
│   ├── process_assets.py       # 资源处理和特征提取
│   └── utils.py                # 工具函数库
│
├── 前端资源
│   └── static/
│       ├── index.html          # 主页面 HTML
│       ├── login.html          # 登录页面
│       ├── assets/             # 静态资源
│       │   ├── *.js            # JavaScript 库（Vue、Axios 等）
│       │   └── *.css           # 样式文件
│       └── locales/            # 多语言配置
│           ├── zh.json         # 中文语言文件
│           └── en.json         # 英文语言文件
│
├── 配置文件
│   ├── requirements.txt        # Python 依赖列表
│   ├── config.py               # 默认配置
│   ├── .env                    # 环境变量文件（可选）
│   ├── docker-compose.yml      # Docker 容器编排配置
│   └── Dockerfile              # Docker 镜像构建文件
│
├── 脚本工具
│   ├── api_test.py             # API 测试脚本
│   ├── benchmark.py            # 性能基准测试
│   ├── process_pexels.py       # Pexels 视频数据处理
│   ├── create_web_demo_database_json.py # Web 演示数据库生成
│   └── scan.py                 # 文件扫描模块
│
├── 部署文件
│   ├── install.bat             # Windows 依赖安装脚本
│   ├── install_cpu.bat         # Windows CPU 版本安装
│   ├── install_torch_gpu.bat   # Windows GPU 版本安装
│   ├── run.bat                 # Windows 启动脚本
│   └── benchmark.bat           # Windows 基准测试脚本
│
├── 文档
│   ├── README.md               # 中文说明文档
│   ├── README_EN.md            # 英文说明文档
│   └── ARCHITECTURE.md         # 本文件
│
└── 其他
    ├── .git/                   # Git 版本控制
    ├── .github/                # GitHub 配置
    │   ├── workflows/          # CI/CD 工作流
    │   └── dependabot.yml      # 依赖自动更新配置
    ├── LICENSE                 # 项目许可证
    └── test.png                # 测试图片
```

## 核心模块详解

### 1. main.py - Web 应用主入口

**功能**: Flask Web 服务器和 API 路由定义

**主要组件**:
- **Flask 应用实例**: 创建 Web 服务器
- **登录认证**: 会话管理和身份验证
- **API 路由** (共 11 个):
  - `GET /` - 主页面
  - `GET/POST /login` - 登录
  - `GET/POST /logout` - 登出
  - `GET /api/scan` - 启动扫描
  - `GET /api/status` - 获取扫描状态
  - `GET/POST /api/clean_cache` - 清空搜索缓存
  - `POST /api/match` - 执行搜索（多种搜索类型）
  - `GET /api/get_image/<id>` - 获取图片
  - `GET /api/get_video/<path>` - 获取视频
  - `GET /api/download_video_clip/<path>/<start>/<end>` - 下载视频片段
  - `POST /api/upload` - 上传文件

**搜索类型** (search_type 参数):
- `0`: 文字搜图
- `1`: 以图搜图（上传）
- `2`: 文字搜视频
- `3`: 以图搜视频（上传）
- `4`: 图文相似度匹配
- `5`: 以图搜图（数据库中）
- `6`: 以图搜视频（数据库中）
- `7`: 路径搜图
- `8`: 路径搜视频
- `9`: Pexels 视频搜索

### 2. config.py - 配置管理

**功能**: 集中管理所有配置参数，支持环境变量和 .env 文件

**配置分类**:
- **服务器配置**: HOST, PORT
- **扫描配置**: 
  - `ASSETS_PATH`: 素材目录（多个目录用逗号分隔）
  - `SKIP_PATH`: 跳过扫描的目录
  - `IMAGE_EXTENSIONS`: 支持的图片格式
  - `VIDEO_EXTENSIONS`: 支持的视频格式
  - `FRAME_INTERVAL`: 视频帧间隔（秒）
  - `IMAGE_MIN_WIDTH/HEIGHT`: 最小图片尺寸
  - `AUTO_SCAN`: 是否自动扫描
  - `AUTO_SCAN_START_TIME/END_TIME`: 自动扫描时间段

- **模型配置**:
  - `MODEL_NAME`: CLIP 模型名称（中文/英文，小/大/超大）
  - `DEVICE`: 推理设备（cpu/cuda/mps）

- **搜索配置**:
  - `POSITIVE_THRESHOLD`: 正向匹配阈值
  - `NEGATIVE_THRESHOLD`: 反向匹配阈值
  - `IMAGE_THRESHOLD`: 以图搜图阈值

- **其他配置**: 日志、数据库、登录、调试

### 3. models.py - 数据库模型

**功能**: 定义 SQLAlchemy 数据库表模型和数据库连接

**数据库表**:

#### Image 表（本地扫描图片）
```python
- id: Integer (主键)
- path: String(4096) - 文件路径 (索引)
- modify_time: DateTime - 文件修改时间
- features: BINARY - 图片特征向量 (CLIP 提取)
```

#### Video 表（本地扫描视频）
```python
- id: Integer (主键)
- path: String(4096) - 文件路径 (索引)
- frame_time: Integer - 帧时间戳 (索引)
- modify_time: DateTime - 文件修改时间
- features: BINARY - 视频帧特征向量
```

#### PexelsVideo 表（在线 Pexels 视频）
```python
- id: Integer (主键)
- title: String(128) - 视频标题
- description: String(256) - 视频描述
- duration: Integer - 视频时长 (索引)
- view_count: Integer - 播放量 (索引)
- thumbnail_loc: String(256) - 缩略图链接 (索引)
- content_loc: String(256) - 视频链接
- thumbnail_feature: BINARY - 缩略图特征向量
```

**数据库连接**:
- 本地扫描数据库: SQLite (配置项: `SQLALCHEMY_DATABASE_URL`)
- Pexels 视频数据库: 独立 SQLite (`PexelsVideo.db`)

### 4. database.py - 数据库操作层

**功能**: 提供数据库 CRUD 操作的函数接口

**主要函数分类**:

**查询函数**:
- `get_image_id_path_features()` - 获取全部图片信息
- `get_image_features_by_id()` - 按 ID 获取图片特征
- `get_image_path_by_id()` - 按 ID 获取图片路径
- `get_frame_times_features_by_path()` - 按视频路径获取所有帧特征
- `get_video_paths()` - 获取所有视频路径
- `search_image_by_path()` - 按路径模式搜索图片
- `search_video_by_path()` - 按路径模式搜索视频
- `get_pexels_video_features()` - 获取所有 Pexels 视频特征

**统计函数**:
- `get_image_count()` - 图片数量
- `get_video_count()` - 视频数量
- `get_video_frame_count()` - 视频帧总数
- `get_pexels_video_count()` - Pexels 视频数量

**添加函数**:
- `add_image()` - 添加图片
- `add_video()` - 批量添加视频帧
- `add_pexels_video()` - 添加 Pexels 视频

**删除函数**:
- `delete_image_if_outdated()` - 删除过时的图片记录
- `delete_video_if_outdated()` - 删除过时的视频记录
- `delete_video_by_path()` - 删除视频的所有帧
- `delete_record_if_not_exist()` - 删除数据库中不存在的文件记录

**检查函数**:
- `is_video_exist()` - 检查视频是否存在
- `is_pexels_video_exist()` - 检查 Pexels 视频是否存在

### 5. scan.py - 文件扫描与索引

**功能**: 扫描文件系统并建立素材索引

**Scanner 类的核心方法**:

- `init()` - 初始化扫描器，创建数据库表，加载现有统计信息
- `get_status()` - 获取扫描进度状态（进度百分比、剩余时间等）
- `filter_path()` - 过滤文件路径（根据扩展名、跳过列表、忽略关键词）
- `collect_assets()` - 收集指定目录下的所有素材文件
- `scan()` - 主扫描流程：
  1. 收集文件列表
  2. 删除已过时的数据库记录
  3. 处理每个文件（图片 → process_images，视频 → process_video）
  4. 保存特征到数据库
  5. 定期自动保存进度

- `auto_scan()` - 在指定时间范围内自动扫描

**扫描流程**:
```
1. 收集资源 → 2. 检查文件修改时间 → 3. 提取特征 → 4. 保存到数据库 → 5. 清理过时数据
```

### 6. search.py - 搜索功能

**功能**: 实现多种搜索方式和特征匹配

**搜索函数** (都使用 LRU 缓存优化):

**特征搜索基础函数**:
- `search_image_by_feature()` - 通过特征向量搜索图片
- `search_video_by_feature()` - 通过特征向量搜索视频

**文本搜索**:
- `search_image_by_text()` - 文字搜图
- `search_video_by_text()` - 文字搜视频
- `search_pexels_video_by_text()` - 文字搜 Pexels 视频

**图片搜索**:
- `search_image_by_image()` - 以图搜图
- `search_video_by_image()` - 以图搜视频

**路径搜索**:
- `search_image_file()` - 按路径搜索图片
- `search_video_file()` - 按路径搜索视频

**缓存管理**:
- `clean_cache()` - 清空所有搜索缓存

**搜索结果格式**:
```json
[
  {
    "url": "api/get_image/123",
    "path": "/path/to/image.jpg",
    "score": 85.5
  }
]
```

**匹配逻辑**:
- 正向提示词：分数 > 正向阈值 → 显示
- 负向提示词：分数 < 负向阈值 → 显示
- 图片搜索：分数 > 图片阈值 → 显示

### 7. process_assets.py - 资源处理与特征提取

**功能**: 处理图片和视频，使用 CLIP 模型提取语义特征

**模型初始化**:
- 加载 CLIP 模型 (来自 Hugging Face)
- 初始化图像处理器 (Processor)

**图片处理**:
- `get_image_data()` - 读取图片像素数据，验证尺寸
- `process_image()` - 单张图片特征提取
- `process_images()` - 批量图片特征提取
- `process_web_image()` - 网络图片处理

**视频处理**:
- `get_frames()` - 逐帧读取视频
- `process_video()` - 视频特征提取（按 FRAME_INTERVAL 间隔取帧）

**文本处理**:
- `process_text()` - 文本特征提取

**特征匹配**:
- `match_batch()` - 批量特征相似度计算
- `match_text_and_image()` - 文字与图片相似度计算

**特征格式**: 
- CLIP 模型输出: 浮点向量，维度取决于模型大小
- 存储方式: 转换为 BINARY (NumPy 字节流) 存储到数据库

### 8. utils.py - 工具函数库

**功能**: 提供通用工具函数

**主要函数**:
- `get_hash()` - 计算字节流 SHA1 哈希（用于上传文件去重）
- `get_string_hash()` - 计算字符串哈希
- `softmax()` - Softmax 计算（特征归一化）
- `format_seconds()` - 秒数转时:分:秒格式
- `crop_video()` - 调用 ffmpeg 截取视频片段
- `resize_image_with_aspect_ratio()` - 按比例缩放图片

**依赖工具**:
- ffmpeg: 视频截取功能
- pillow_heif: 支持 HEIC/HEIF 图片格式

## 数据流与架构

### 扫描流程
```
┌─────────────────────────────────────────────┐
│  用户触发扫描 (/api/scan)                   │
└─────────────┬───────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────┐
│  Scanner.scan() - 主扫描函数                │
├─────────────────────────────────────────────┤
│  1. collect_assets() - 收集文件列表          │
│     - 遍历 ASSETS_PATH 目录                 │
│     - 过滤扩展名、忽略路径、忽略关键词      │
│  2. 检查文件修改时间                         │
│     - 删除过时的数据库记录                  │
│  3. 批量处理文件                             │
│     - process_images() - 图片特征提取       │
│     - process_video() - 视频帧特征提取      │
│  4. 保存到数据库                             │
│     - database.add_image()                   │
│     - database.add_video()                   │
│  5. 清理                                      │
│     - delete_record_if_not_exist() - 清理   │
│     - clean_cache() - 清空搜索缓存          │
└─────────────┬───────────────────────────────┘
              │
              ▼
        ┌────────────────┐
        │  数据库更新    │
        │  素材索引完成  │
        └────────────────┘
```

### 搜索流程
```
┌─────────────────────────────────────────────┐
│  用户发起搜索请求                            │
│  POST /api/match (search_type, 参数)       │
└─────────────┬───────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────┐
│  main.py - api_match()                      │
│  根据 search_type 调用对应搜索函数           │
└─────────────┬───────────────────────────────┘
              │
    ┌─────────┴─────────┬─────────┬───────────┐
    │                   │         │           │
    ▼                   ▼         ▼           ▼
  文本搜索         以图搜索      相似度      Pexels
  ┌──────┐      ┌──────────┐   ┌──────┐    ┌──────┐
  │      │      │          │   │      │    │      │
  └──┬───┘      └────┬─────┘   └──┬───┘    └──┬───┘
     │               │            │           │
     ▼               ▼            ▼           ▼
┌──────────────────────────────────────────────┐
│  process_assets.py                           │
│  - process_text() - 文本特征提取              │
│  - process_image() - 图片特征提取             │
│  - match_batch() - 特征相似度计算             │
└──────────┬───────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────┐
│  search.py - 搜索函数                        │
│  - 从数据库加载特征                          │
│  - 应用匹配阈值过滤                          │
│  - 排序结果                                  │
│  - LRU 缓存优化                              │
└──────────┬───────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────┐
│  JSON 格式搜索结果                           │
│  [                                           │
│    {url, path, score},                       │
│    ...                                       │
│  ]                                           │
└──────────────────────────────────────────────┘
```

### 特征提取与存储
```
┌─────────────────────────────────┐
│  原始图片/视频帧 (JPG/PNG/等)    │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│  process_image/process_images   │
│  - 用 PIL 读取图片              │
│  - 转换为 RGB                   │
│  - Processor 预处理             │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│  CLIP 模型推理                  │
│  AutoModelForZeroShotImage      │
│  Classification                 │
│  (MODEL_NAME 配置)              │
│  Device: CPU/CUDA/MPS           │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│  特征向量 (浮点向量)            │
│  维度: 512/768/1024 等           │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│  NumPy 字节序列化               │
│  np.array.tobytes()             │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│  保存到数据库 BINARY 列          │
│  Image.features / Video.features │
│  / PexelsVideo.thumbnail_feature│
└─────────────────────────────────┘
```

## API 接口文档

### 认证
所有 API 端点都需要登录认证（如启用 `ENABLE_LOGIN=True`）。

### 核心接口

#### 1. 搜索接口
```
POST /api/match
Content-Type: application/json

请求体:
{
  "search_type": 0,           // 搜索类型 (0-9)
  "top_n": 20,                // 返回结果数
  "positive_threshold": 36,   // 正向阈值
  "negative_threshold": 36,   // 反向阈值
  "image_threshold": 85,      // 图片搜索阈值
  "positive": "blue sky",     // 正向提示词
  "negative": "dark",         // 反向提示词
  "text": "",                 // 文本
  "img_id": 0,                // 数据库图片 ID
  "path": ""                  // 文件路径
}

响应: JSON 数组
[
  {
    "url": "api/get_image/123",
    "path": "/path/to/image.jpg",
    "score": 85.5
  }
]
```

#### 2. 扫描接口
```
GET /api/scan
响应: {"status": "start scanning"} 或 {"status": "already scanning"}
```

#### 3. 状态接口
```
GET /api/status
响应:
{
  "status": false,                  // 是否正在扫描
  "total_images": 1000,             // 总图片数
  "total_videos": 50,               // 总视频数
  "total_video_frames": 5000,       // 总视频帧数
  "scanning_files": 0,              // 正在扫描的文件数
  "remain_files": 0,                // 剩余文件数
  "progress": 0,                    // 进度 (0-1)
  "remain_time": 0,                 // 剩余秒数
  "enable_login": false,            // 登录功能启用状态
  "total_pexels_videos": 0          // Pexels 视频数
}
```

#### 4. 上传接口
```
POST /api/upload
Content-Type: multipart/form-data

参数: file (文件对象)
响应: "file uploaded successfully"
```

#### 5. 文件获取接口
```
GET /api/get_image/<image_id>?thumbnail=true

查询参数:
- thumbnail: true - 返回缩略图 (JPEG, 640x480)
- 不指定 - 返回原始文件

响应: 图片文件流
```

```
GET /api/get_video/<video_path>

参数:
- video_path: base64 编码的视频路径

响应: 视频文件流
```

#### 6. 视频片段下载
```
GET /api/download_video_clip/<video_path>/<start_time>/<end_time>

参数:
- video_path: base64 编码的视频路径
- start_time: 开始时间 (秒)
- end_time: 结束时间 (秒)

响应: 视频片段文件流
```

#### 7. 缓存清理
```
GET/POST /api/clean_cache
响应: 204 No Content
```

#### 8. 登录接口
```
POST /login
Content-Type: application/x-www-form-urlencoded

参数: username, password
响应: 重定向或登录页面
```

## 关键文件清单

| 文件 | 行数 | 功能 | 关键类/函数 |
|------|------|------|-----------|
| main.py | 284 | Flask 应用, API 路由 | Flask app, login_required, api_match |
| config.py | 68 | 配置管理 | 全局配置常量 |
| models.py | 66 | ORM 模型 | Image, Video, PexelsVideo, DatabaseSession |
| database.py | 247 | 数据库操作 | 20+ CRUD 函数 |
| scan.py | 235 | 文件扫描 | Scanner 类 |
| search.py | 350 | 搜索功能 | 7+ 搜索函数 |
| process_assets.py | 241 | 特征提取 | process_image, process_video, match_batch |
| utils.py | 118 | 工具函数 | 6+ 工具函数 |

## 性能优化策略

### 1. 特征提取优化
- **批处理**: `SCAN_PROCESS_BATCH_SIZE` - 收集多个帧后一次性输入模型
- **设备选择**: `DEVICE` - 支持 CPU/CUDA/MPS，根据硬件自动选择
- **型号选择**: 小/中/大模型，权衡速度和准确度

### 2. 搜索优化
- **LRU 缓存**: 缓存最近 N 次搜索结果（`CACHE_SIZE`）
- **索引**: 数据库字段建立索引加速查询
- **阈值过滤**: 提前过滤低分结果

### 3. 扫描优化
- **增量扫描**: 检查文件修改时间，跳过未改变的文件
- **自动保存**: `AUTO_SAVE_INTERVAL` - 每 N 个文件自动保存进度
- **路径过滤**: 提前排除不必要的目录和文件

### 4. 视频处理优化
- **帧间隔**: `FRAME_INTERVAL` - 设置更大的间隔减少处理帧数
- **分片保存**: 使用 `bulk_save_objects` 批量提交数据库

## 配置关键参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| ASSETS_PATH | /home,/srv | 素材扫描目录 |
| MODEL_NAME | OFA-Sys/chinese-clip-vit-base-patch16 | CLIP 模型 |
| DEVICE | cpu | 推理设备 |
| FRAME_INTERVAL | 2 | 视频帧间隔 (秒) |
| POSITIVE_THRESHOLD | 36 | 正向匹配阈值 |
| NEGATIVE_THRESHOLD | 36 | 反向匹配阈值 |
| IMAGE_THRESHOLD | 85 | 以图搜图阈值 |
| CACHE_SIZE | 64 | 搜索缓存条目数 |
| AUTO_SCAN | False | 自动扫描开关 |
| ENABLE_LOGIN | False | 登录功能开关 |

## 部署架构

### 本地部署
```
用户 → Web 浏览器 → Flask Server (main.py)
                 ↓
            SQLite 数据库
                 ↓
            CLIP 模型 (CPU/GPU)
                 ↓
            本地文件系统
```

### Docker 部署
```
Docker 容器
  ├── Python + Flask
  ├── PyTorch (CPU/GPU)
  ├── SQLite 数据库
  └── 挂载本地卷
      └── 素材文件 + 数据库
```

### 端口
- 默认: `8085`
- 可配置: `PORT` 环境变量

## 技术难点与解决方案

### 1. 模型加载
**问题**: 首次运行需要从 Hugging Face 下载模型，网络不稳定会失败
**解决**: 
- 重试机制 (重新执行程序)
- 支持离线模式 (`TRANSFORMERS_OFFLINE=1`)
- 可预先下载模型

### 2. 内存管理
**问题**: 处理大量图片/视频时内存溢出
**解决**:
- 批处理控制 (`SCAN_PROCESS_BATCH_SIZE`)
- 及时释放临时对象
- 建议最少 4G 内存

### 3. GPU 加速
**问题**: GPU 不一定比 CPU 快（数据传输开销）
**解决**:
- 提供 `benchmark.py` 工具对比测试
- 支持多种设备选择
- 根据硬件配置参数

### 4. 视频处理
**问题**: 某些视频格式浏览器不支持
**解决**:
- 支持多种格式 (mp4, mkv, webm 等)
- 视频片段下载使用 ffmpeg 重编码

### 5. 路径安全
**问题**: 恶意用户可能读取任意文件
**解决**:
- 验证视频路径在数据库中存在
- Base64 编码视频路径参数
- 路径检查防护

## 扩展点

### 1. 新增搜索类型
在 `main.py:api_match()` 中添加新的 `search_type` 分支

### 2. 新增搜索函数
在 `search.py` 中添加新函数，使用 `@lru_cache` 装饰器

### 3. 新增数据库表
在 `models.py` 中定义新的模型类，继承 `BaseModel` 或 `BaseModelPexelsVideo`

### 4. 新增特征提取方法
在 `process_assets.py` 中添加新的处理函数

### 5. 前端开发
修改 `static/index.html` (Vue.js 应用)

## 常见问题

1. **扫描速度慢**
   - 调整 `FRAME_INTERVAL` (更大的值)
   - 调整 `SCAN_PROCESS_BATCH_SIZE`
   - 选择更小的模型

2. **内存占用高**
   - 减小 `SCAN_PROCESS_BATCH_SIZE`
   - 选择更小的模型
   - 增加物理内存

3. **某些文件无法被扫描**
   - 检查文件扩展名是否在 `IMAGE_EXTENSIONS` / `VIDEO_EXTENSIONS` 中
   - 检查文件是否在 `SKIP_PATH` 中
   - 检查文件路径或名称是否在 `IGNORE_STRINGS` 中

4. **搜索结果不准确**
   - 调整 `POSITIVE_THRESHOLD` / `NEGATIVE_THRESHOLD`
   - 尝试其他模型
   - 优化搜索词

## 总结

MaterialSearch 是一个架构清晰、功能完整的本地素材搜索系统：

- **模块划分明确**: 扫描、搜索、处理、存储分离
- **可配置性强**: 支持环境变量、.env 文件、Docker 等多种部署方式
- **性能优化完善**: 缓存、索引、批处理、增量扫描
- **安全考虑周到**: 路径验证、身份认证、会话管理
- **易于扩展**: 模块化设计方便添加新功能

项目采用 Python + Flask + SQLAlchemy + CLIP 的经典组合，充分利用深度学习模型的语义理解能力，为用户提供强大的自然语言搜索体验。
