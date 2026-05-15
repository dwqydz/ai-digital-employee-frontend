# ⚠️ 重要：Railway 部署问题解决方案

## 🔴 当前问题

Railway 显示错误：`sh: 1: npm: not found`

**原因**：Railway 没有使用 Dockerfile，而是在尝试用 Nixpacks 构建，但 Nixpacks 没有找到 Node.js/npm。

## ✅ 已执行的修复

我已经删除了可能干扰 Docker 检测的文件：
- ❌ 删除 `nixpacks.toml`（会强制使用 Nixpacks）
- ❌ 删除 `railway.json`（可能指定了错误的构建器）
- ❌ 删除 `Procfile`（Heroku 格式，不适用于 Railway）

现在只保留 **Dockerfile**，Railway 应该会自动检测到并使用它。

## 📋 你必须执行的操作

### 步骤 1：推送代码到 GitHub

```bash
git add .
git commit -m "Remove nixpacks config, use only Dockerfile for Railway"
git push
```

### 步骤 2：在 Railway 中强制使用 Docker

**这是最关键的一步！**

1. 登录 [railway.app](https://railway.app)
2. 进入你的前端服务
3. 点击 "**Settings**" 标签
4. 找到 "**Build Provider**" 或 "**Builder**" 选项
5. **手动选择 "Dockerfile"**（不要选 Automatic 或 Nixpacks）
6. 保存设置

### 步骤 3：重新部署

1. 在 Settings 页面找到 "**Danger Zone**"
2. 点击 "**Redeploy**" 或 "**Restart**"
3. 或者删除服务后重新创建

### 步骤 4：配置环境变量

在 Variables 标签中添加：

```
VITE_API_BASE_URL=https://langgraph-ai-digital-employee-production.up.railway.app/api
```

**不需要**设置 `NODE_VERSION` 或 `PORT`（Docker 方案会自动处理）

### 步骤 5：查看日志

点击 "**Deployments**" → 查看最新部署的 "**Logs**"

你应该看到类似这样的输出：

```
✓ Building with Docker
Step 1/12 : FROM node:18-alpine AS builder
Step 2/12 : WORKDIR /app
Step 3/12 : COPY package*.json ./
Step 4/12 : RUN npm ci
...
Step 7/12 : RUN npm run build
✓ Build successful
Step 8/12 : FROM nginx:alpine
...
✓ Deployment successful
```

## ❓ 如果仍然失败

### 检查清单：

- [ ] 确认已在 Railway Settings 中**手动选择 "Dockerfile"** 作为 Build Provider
- [ ] 确认 Dockerfile 已推送到 GitHub
- [ ] 确认删除了旧服务并创建了新服务（如果修改 Build Provider 后仍失败）

### 备选方案：本地测试 Docker 构建

在本地测试 Docker 是否能正常构建：

```bash
# 在项目根目录执行
docker build -t test-frontend .
docker run -p 8080:80 test-frontend
```

然后在浏览器访问 `http://localhost:8080`，看是否能正常运行。

如果本地 Docker 构建成功，说明 Dockerfile 没问题，问题出在 Railway 的配置上。

## 🎯 为什么这次会成功？

1. **删除了 nixpacks.toml**：不再强制使用 Nixpacks
2. **删除了 railway.json**：移除了可能的错误配置
3. **保留了 Dockerfile**：Railway 会自动检测并使用
4. **添加了 .dockerignore**：优化构建速度

**关键**：必须在 Railway Settings 中手动选择 "Dockerfile" 作为构建器！

## 📞 需要帮助？

如果按照上述步骤操作后仍然失败，请提供：
1. Railway Settings 中 "Build Provider" 的截图
2. Railway Logs 的完整输出
3. 浏览器控制台的错误信息（如果有）

---

**记住**：Railway 默认可能不会自动选择 Dockerfile，你必须手动在 Settings 中指定！
