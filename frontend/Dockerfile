# Stage 1: Build Vue app
FROM node:22-alpine AS builder
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .
RUN npm run build

# Stage 2: Serve bằng Nginx
FROM nginx:alpine
WORKDIR /usr/share/nginx/html

# Xóa default config
RUN rm -rf ./*

# Copy static files đã build
COPY --from=builder /app/dist .

# Nginx config để handle Vue Router (SPA)
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
