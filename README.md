# THCUNews Classifier (Frontend)

这是一个基于 React + TypeScript + Vite 构建的现代化新闻文本分类系统前端界面。项目采用了 Hero Page 设计风格，集成了拖拽上传、智能交互与数据可视化功能。

## 🛠 技术栈

- **核心框架**: React 18, TypeScript, Vite
- **样式方案**: Tailwind CSS (极简原子化 CSS)
- **UI 组件**: 自定义 Glassmorphism (毛玻璃) 风格组件, Lucide React (图标)
- **工具库**: `clsx`, `tailwind-merge` (样式合并)
- **可视化**: Recharts (数据图表)

---

## 🚀 1. 环境准备与项目构建 (从零开始)

如果您需要在本地机器上从头构建此项目，请按照以下步骤操作：

### 第一步：初始化 Vite 项目
打开终端（Terminal），运行以下命令创建一个 React + TypeScript 项目：

```bash
npm create vite@latest thcunews-classifier -- --template react-ts
cd thcunews-classifier
```

### 第二步：安装依赖
安装项目所需的第三方库：

```bash
# 安装 UI 相关库
npm install lucide-react recharts clsx tailwind-merge

# (可选) 如果不使用 CDN，建议本地安装 Tailwind CSS
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

### 第三步：配置 Tailwind CSS
修改 `tailwind.config.js` 以支持文件扫描：

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

并在 `src/index.css` 中添加指令：
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

## 🏃 2. 启动与打包

### 启动开发服务器
在项目根目录下运行：

```bash
npm run dev
```
打开浏览器访问 `http://localhost:5173` 即可看到实时预览。

### 构建生产版本
当准备部署时，运行：

```bash
npm run build
```
该命令会在 `dist` 目录下生成优化后的静态文件，可直接部署到 Nginx、Vercel 或 Netlify。

---

## 🔌 3. 核心代码修改指南：对接真实后端

当前项目使用 `mockApi.ts` 模拟后端响应。要对接您部署的 FastAPI (BERT/RoBERTa) 服务，请修改 `src/services/mockApi.ts`。

### 修改步骤

1.  找到 `src/services/mockApi.ts` 文件。
2.  删除原有的 `setTimeout` 模拟逻辑。
3.  使用 `fetch` 或 `axios` 发起真实的 HTTP 请求。

### 代码示例

假设您的后端 API 地址为 `http://localhost:8000/predict`，修改后的代码如下：

```typescript
// src/services/mockApi.ts

import { PredictResponse, ModelType } from '../types';

const API_URL = "http://localhost:8000/predict"; // 您的真实后端地址

export const mockAnalyze = async (text: string, model: ModelType): Promise<PredictResponse> => {
  try {
    const response = await fetch(API_URL, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        text: text,
        model: model // 传递选中的模型 (BERT 或 RoBERTa)
      }),
    });

    if (!response.ok) {
      throw new Error(`API Error: ${response.statusText}`);
    }

    const data = await response.json();
    
    // 确保后端返回的数据格式与前端 PredictResponse 接口一致
    // 如果不一致，在这里进行格式转换
    return data as PredictResponse;

  } catch (error) {
    console.error("API Request Failed:", error);
    throw error; // 抛出错误以便 UI 层捕获并提示用户
  }
};
```

### 后端 FastAPI 数据格式要求

为了适配前端展示，您的后端返回 JSON 结构应如下所示（或者在前端手动转换）：

```json
{
  "category": "财经",
  "confidence": 0.98,
  "inference_time_ms": 120,
  "probabilities": [
    { "name": "财经", "value": 98.2 },
    { "name": "科技", "value": 1.5 },
    { "name": "政治", "value": 0.3 }
    // ... Top 5
  ]
}
```

### 跨域问题 (CORS)

由于前端通常运行在 `localhost:5173`，后端在 `localhost:8000`，浏览器会拦截跨域请求。请务必在 **FastAPI** 中配置 CORS：

```python
# FastAPI backend setup
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"], # 生产环境建议设置为具体的前端域名
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

## 📂 项目结构说明

```
src/
├── components/          # UI 组件
│   ├── InputSection.tsx # 输入框与拖拽逻辑
│   ├── ResultSection.tsx# 结果展示与图表
│   └── ui/              # 基础通用组件 (Button 等)
├── services/            # API 服务层 (在此处修改后端连接)
├── types.ts             # TypeScript 类型定义
├── constants.ts         # 常量与示例数据
└── App.tsx              # 主页面逻辑与布局
```
