# 使用 Node.js 18 作为基础镜像
FROM node:18-alpine AS builder

# 设置工作目录
WORKDIR /app

# 复制 package.json 和 package-lock.json
COPY package*.json ./

# 安装所有依赖（包括 devDependencies）
RUN npm install

# 复制所有文件
COPY . .

# 从环境变量获取 API URL（Railway 会在构建时注入）
ARG VITE_API_BASE_URL
ENV VITE_API_BASE_URL=$VITE_API_BASE_URL

# 构建应用
RUN npm run build

# 生产阶段 - 使用 nginx 提供静态文件服务
FROM nginx:alpine

# 复制构建产物到 nginx
COPY --from=builder /app/dist /usr/share/nginx/html

# 从 builder 阶段复制 nginx 配置
COPY --from=builder /app/nginx.conf /etc/nginx/conf.d/default.conf

# 暴露端口
EXPOSE 80

# 启动 nginx
CMD ["nginx", "-g", "daemon off;"]
