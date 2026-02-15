# 转发 /telegram-webhook 到容器 8787 端口

## Context

容器内有第二个服务运行在 8787 端口，需要将 `/telegram-webhook` 路径的请求转发到该端口，其余请求保持走默认的 6658 端口。

## 修改方案

### 修改文件：`src/container.ts`

在 `AgentContainer` 类中 override `fetch` 方法，当路径以 `/telegram-webhook` 开头时，用 `containerFetch` 转发到 `http://container:8787`，否则走 `super.fetch()`。

## 验证

- `nr typecheck` 检查类型
- `nr lint` 检查代码风格
