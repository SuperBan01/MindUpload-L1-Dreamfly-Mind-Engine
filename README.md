# 云己 - 意识上传平台

云己（upme）是世界上第一款意识上传平台，基于梦蝶心智模型（Dreamfly Mind Model）实现用户意识数据的采集、处理和生成。通过上传用户的思想、声音和形象数据，生成独特的数字意识体，实现数字永生。

## 技术栈

- 前端框架：Vue 3 + uni-app
- UI组件：自定义组件
- 状态管理：Vuex
- 网络请求：uni.request
- 文件处理：uni.uploadFile

## 项目结构

```
src/
├── components/          # 公共组件
├── pages/              # 页面文件
│   ├── welcome/        # 欢迎页
│   ├── auth/           # 登录注册相关页面
│   └── upload/         # 数据上传页面
├── static/             # 静态资源
├── store/              # Vuex状态管理
├── utils/              # 工具函数
├── App.vue             # 应用入口
├── main.js            # 主入口文件
├── manifest.json      # 应用配置
└── pages.json         # 页面路由配置
```

## 接口规范

### 1. 用户认证接口

#### 1.1 登录

```
POST /api/v1/auth/login
Content-Type: application/json

Request:
{
    "email": string,
    "password": string
}

Response:
{
    "code": number,
    "message": string,
    "data": {
        "token": string,
        "user": {
            "id": string,
            "username": string,
            "email": string,
            "avatar": string
        }
    }
}
```

#### 1.2 注册

```
POST /api/v1/auth/register
Content-Type: application/json

Request:
{
    "username": string,
    "email": string,
    "password": string
}

Response:
{
    "code": number,
    "message": string,
    "data": {
        "token": string,
        "user": {
            "id": string,
            "username": string,
            "email": string
        }
    }
}
```

### 2. 数据上传接口

#### 2.1 思想数据上传

```
POST /api/v1/upload/thoughts
Content-Type: multipart/form-data

Request:
{
    "file": File,
    "type": string,  // 文件类型：text, doc, pdf
    "description": string
}

Response:
{
    "code": number,
    "message": string,
    "data": {
        "fileId": string,
        "size": number,
        "progress": number
    }
}
```

#### 2.2 声音数据上传

```
POST /api/v1/upload/voice
Content-Type: multipart/form-data

Request:
{
    "file": File,
    "type": string,  // 文件类型：mp3, wav, m4a
    "duration": number,
    "description": string
}

Response:
{
    "code": number,
    "message": string,
    "data": {
        "fileId": string,
        "size": number,
        "progress": number
    }
}
```

#### 2.3 形象数据上传

```
POST /api/v1/upload/image
Content-Type: multipart/form-data

Request:
{
    "files": File[],
    "type": string,  // 文件类型：image, video
    "description": string
}

Response:
{
    "code": number,
    "message": string,
    "data": {
        "fileIds": string[],
        "totalSize": number,
        "progress": number
    }
}
```

### 3. 意识体生成接口

```
POST /api/v1/consciousness/generate
Content-Type: application/json

Request:
{
    "userId": string,
    "thoughtsData": string[],  // 思想数据文件ID列表
    "voiceData": string[],     // 声音数据文件ID列表
    "imageData": string[]      // 形象数据文件ID列表
}

Response:
{
    "code": number,
    "message": string,
    "data": {
        "consciousnessId": string,
        "status": string,
        "estimatedTime": number
    }
}
```

## 数据流通标准

### 1. 思想数据标准

- 文本文件：UTF-8编码，支持txt、doc、docx、pdf格式
- 单个文件大小限制：5MB
- 文件内容要求：个人日记、文章、社交媒体内容等个人思想记录

### 2. 声音数据标准

- 音频格式：MP3、WAV、M4A
- 采样率：≥ 44.1kHz
- 比特率：≥ 128kbps
- 单个文件大小限制：10MB
- 时长限制：单个文件不超过10分钟

### 3. 形象数据标准

- 图片格式：JPG、PNG、WEBP
- 图片分辨率：≥ 1080p
- 视频格式：MP4、MOV
- 视频分辨率：≥ 720p
- 单个文件大小限制：20MB

### 4. 意识体文件标准

- 格式：JSON
- 大小限制：10MB
- 结构：
  ```json
  {
    "version": "1.0",
    "id": "unique_consciousness_id",
    "creator": {
      "id": "user_id",
      "username": "username"
    },
    "metadata": {
      "createdAt": "timestamp",
      "updatedAt": "timestamp",
      "dataSize": {
        "thoughts": number,
        "voice": number,
        "image": number
      }
    },
    "consciousness": {
      "thoughts": object,
      "voice": object,
      "image": object,
      "personality": object,
      "memories": array
    }
  }
  ```

## 开发指南

1. 克隆项目
```bash
git clone https://github.com/your-username/upme.git
```

2. 安装依赖
```bash
npm install
```

3. 运行开发服务器
```bash
npm run dev:h5
```

4. 打包
```bash
npm run build:h5
```

## 预览

在浏览器中打开以下地址即可预览移动端效果：
```
http://localhost:8080/
```

## 注意事项

1. 所有上传的数据都需要进行加密处理
2. 用户数据需要严格遵守隐私政策
3. 意识体生成过程中需要显示进度和预计完成时间
4. 确保跨平台兼容性（H5、小程序、App）
5. 实现断点续传功能
6. 添加数据备份功能 