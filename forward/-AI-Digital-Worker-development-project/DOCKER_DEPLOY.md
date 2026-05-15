# Railway Docker 部署方案

## 🎯 为什么使用 Docker？

之前的 Nixpacks 配置可能因为以下原因失败：
1. Railway 缓存了旧的构建配置
2. Nixpacks 版本兼容性问题
3. 环境变量传递问题

**Docker 方案的优势：**
- ✅ 完全可控的构建环境
- ✅ 不依赖 Railway 的自动检测
- ✅ 使用 nginx 提供静态文件，性能更好
- ✅ 多阶段构建，最终镜像体积小

## 📦 Docker 方案说明

### 构建流程

1. **第一阶段（builder）**：
   - 使用 Node.js 18 Alpine 镜像
   - 安装所有依赖（包括 devDependencies）
   - 执行 `npm run build` 生成静态文件

2. **第二阶段（production）**：
   - 使用 nginx Alpine 镜像（非常轻量）
   - 复制构建产物到 nginx
   - 使用自定义 nginx 配置支持 Vue Router history 模式

### 优势

- **性能**：nginx 比 Node.js preview 服务器快得多
- **体积**：最终镜像只有 ~25MB（vs Node.js 的 ~100MB+）
- **稳定**：不依赖 npm/node 运行时
- **安全**：生产环境没有 node_modules，攻击面更小

## 🚀 部署步骤

### 1. 推送代码到 GitHub

```bash
git add .
git commit -m "Use Docker with nginx for production deployment"
git push
```

### 2. 在 Railway 中配置

#### 方式 A：自动检测（推荐）

Railway 会自动检测到 `Dockerfile` 并使用它构建。

1. 删除旧的前端服务（如果存在）
2. 创建新服务 → Deploy from GitHub repo
3. Railway 会自动使用 Dockerfile

#### 方式 B：强制使用 Docker

如果 Railway 没有自动使用 Docker：

1. 进入服务 Settings
2. 找到 "Build Provider"
3. 选择 "Dockerfile"

### 3. 配置环境变量

在 Railway Variables 中添加：

```
VITE_API_BASE_URL=https://langgraph-ai-digital-employee-production.up.railway.app/api
```

**注意**：使用 Docker 方案时，不需要设置 `NODE_VERSION` 和 `PORT`，因为：
- Node.js 版本在 Dockerfile 中指定
- nginx 固定使用 80 端口，Railway 会自动映射

### 4. 等待部署完成

查看 Logs，你应该看到：

```
Step 1/12 : FROM node:18-alpine AS builder
Step 2/12 : WORKDIR /app
...
Step 7/12 : RUN npm run build
...
Step 9/12 : COPY --from=builder /app/dist /usr/share/nginx/html
...
Successfully built xxxxx
```

## 🔍 验证部署

1. 获取 Railway 分配的 URL
2. 访问前端应用
3. 打开浏览器开发者工具 → Network 标签
4. 检查 API 请求是否正确发送到后端
5. 测试页面刷新（Vue Router history 模式应该正常工作）

## 📝 配置文件说明

### Dockerfile
- 多阶段构建
- 第一阶段：Node.js 18 + 构建
- 第二阶段：nginx + 静态文件

### nginx.conf
- 支持 Vue Router history 模式（try_files）
- 静态资源缓存优化
- 404 错误重定向到 index.html

### .railwayignore
- 排除不必要的文件，加速构建
- 保留 dist 目录（在 Docker 构建时会重新生成）

## ❓ 常见问题

### Q: 为什么不用 Node.js preview 服务器？

A: 
- nginx 性能更好，资源占用更少
- 更适合生产环境的静态文件服务
- 更好的并发处理能力

### Q: 如何修改 API 代理配置？

A: 
如果需要在前端服务器代理 API 请求（避免 CORS），可以取消注释 `nginx.conf` 中的代理配置：

```nginx
location /api/ {
    proxy_pass https://langgraph-ai-digital-employee-production.up.railway.app/api/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

然后重新构建部署。

### Q: 构建失败怎么办？

A:
1. 检查 Railway Logs 查看详细错误
2. 确认 `package.json` 中的所有依赖都正确
3. 确认 `.env` 中的 `VITE_API_BASE_URL` 在 Railway Variables 中已设置

## 🎉 完成！

使用 Docker + nginx 方案后，你的应用应该能够：
- ✅ 成功构建
- ✅ 快速启动
- ✅ 高性能运行
- ✅ 正确处理路由
- ✅ 正确连接后端 API
