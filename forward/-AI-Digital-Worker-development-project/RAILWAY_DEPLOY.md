# Railway 部署指南

## 前端部署配置

### 1. 环境变量配置

在 Railway 前端服务的 "Variables" 标签页中添加以下环境变量：

```
NODE_VERSION=18
VITE_API_BASE_URL=https://langgraph-ai-digital-employee-production.up.railway.app/api
PORT=4173
```

**重要**：`VITE_API_BASE_URL` 必须设置为你的后端服务地址

### 1.1 Node.js 版本配置（已自动配置）

项目已包含以下文件来确保 Railway 使用正确的 Node.js 版本：
- `.nvmrc` - 指定 Node.js 18
- `.npmrc` - npm 配置
- `package.json` 中的 `engines` 字段
- `Dockerfile` - 备选部署方案

### 2. 构建配置

Railway 会自动检测并使用以下配置：

- **Build Command**: `npm run build`
- **Start Command**: `npm run preview` 或 `npm start`

如果自动检测失败，请在 Settings 中手动配置。

### 3. 部署步骤

1. 推送代码到 GitHub
2. 在 Railway 创建新项目，选择从 GitHub 仓库部署
3. 选择你的前端仓库
4. 在 Variables 中添加上述环境变量
5. 等待自动部署完成
6. 获取分配的前端 URL（类似 https://xxx.railway.app）

### 4. 已创建的配置文件

- `Procfile`: 定义启动命令
- `public/_redirects`: Vue Router history 模式支持
- `.railwayignore`: 优化部署速度，排除不必要的文件
- `vite.config.js`: 已更新以支持 Railway 端口配置
- `package.json`: 已添加 start 脚本和更新 preview 脚本

### 5. 后端 CORS 配置

确保你的后端服务（langgraph-ai-digital-employee-production.up.railway.app）配置了 CORS，允许前端域名访问。

如果需要，在后端添加：
```
Access-Control-Allow-Origin: *
```

或者指定前端域名：
```
Access-Control-Allow-Origin: https://your-frontend-name.railway.app
```

### 6. 故障排查

**问题：前端无法连接后端 API**
- 检查浏览器控制台是否有 CORS 错误
- 确认 VITE_API_BASE_URL 环境变量配置正确
- 验证后端服务是否正常运行

**问题：页面刷新后 404**
- 确认 `public/_redirects` 文件已上传
- 检查 Vue Router 是否使用 history 模式

**问题：构建失败**
- 查看 Railway Logs 标签页的详细错误信息
- 确认所有依赖都在 package.json 中声明
- **如果看到 "npm: not found" 错误**：
  - 确保已推送 `.nvmrc` 文件到 GitHub
  - 检查 Railway Variables 中是否设置了 `NODE_VERSION=18`
  - 尝试删除 Railway 服务并重新部署

### 7. 测试部署

部署完成后：
1. 访问前端 URL
2. 打开浏览器开发者工具（F12）
3. 切换到 Network 标签
4. 检查 API 请求是否成功发送到后端
5. 确认响应状态码为 200
