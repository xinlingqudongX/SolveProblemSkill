# 项目上下文

> 本文档记录用户确定的项目规范和技术栈，供 skill 在判断阶段和决策时读取。

---

## 项目信息

| 字段 | 值 |
|------|-----|
| 项目名称 | _(待填写)_ |
| 项目类型 | _(待填写：Web应用 / 微服务 / CLI工具 / ...)_ |
| 仓库地址 | _(待填写)_ |
| 主要负责人 | _(待填写)_ |

---

## 技术栈

### 后端
| 类别 | 技术 | 版本 | 备注 |
|------|------|------|------|
| 运行时 | _(例：Node.js / Deno)_ | | |
| 框架 | _(例：NestJS / Express / Fastify)_ | | |
| 语言 | _(例：TypeScript)_ | | |
| 数据库 | _(例：PostgreSQL / MySQL / MongoDB)_ | | |
| ORM | _(例：Prisma / TypeORM / Drizzle)_ | | |
| 消息队列 | _(例：RabbitMQ / Kafka / Pulsar)_ | | |
| 缓存 | _(例：Redis / Memcached)_ | | |
| 对象存储 | _(例：AWS S3 / MinIO)_ | | |

### 前端
| 类别 | 技术 | 版本 | 备注 |
|------|------|------|------|
| 框架 | | | |
| UI库 | | | |
| 状态管理 | | | |
| 构建工具 | | | |

### 基础设施
| 类别 | 技术 | 备注 |
|------|------|------|
| 容器化 | _(例：Docker)_ | |
| 编排 | _(例：K8s / Docker Compose)_ | |
| CI/CD | _(例：GitHub Actions / Jenkins)_ | |
| 监控 | _(例：Grafana / Prometheus)_ | |
| 日志 | _(例：ELK / Loki)_ | |

---

## 项目架构

### 目录结构

```
src/
├── modules/        # 业务模块
├── common/         # 公共组件（装饰器、过滤器、守卫、拦截器等）
├── config/         # 配置管理
├── database/       # 数据库相关
├── infra/          # 基础设施层
├── grpc-clients/   # gRPC 客户端
├── grpc-controllers/ # gRPC 控制器
├── rpc/            # RPC 层
├── schedule/       # 定时任务
├── ws/             # WebSocket
└── proto/          # Proto 定义
```

### 模块规范

- 模块命名：`kebab-case`
- 每个模块包含：`*.module.ts`, `*.controller.ts`, `*.service.ts`, `*.test.ts`
- 配置通过 `src/config/` 集中管理

---

## 编码规范

### TypeScript
- _(例：严格模式、ESLint规则、命名约定等)_

### NestJS
- _(例：Provider注册方式、模块组织、装饰器使用等)_

### 测试
- 框架：_(例：Jest)_
- 覆盖率要求：_(例：80%+)_
- 测试文件命名：`*.test.ts`

### 提交规范
- 格式：_(例：feat: / fix: / chore:)_
- 分支策略：_(例：feature/*, fix/*, main)_

---

## 环境配置

| 环境 | 说明 |
|------|------|
| local | 本地开发环境 |
| dev | 开发环境 |
| staging | 预发布环境 |
| prod | 生产环境 |

配置管理方式：_(例：.env文件 / 配置中心)_

---

## 用户偏好

> 记录用户在对话中明确表达的偏好和决策。

- _(例：偏好先写测试再实现功能)_
- _(例：Provider 必须有配置，不能用默认值)_

---

## 更新日志

| 日期 | 更新内容 | 更新人 |
|------|---------|--------|
| | 创建文档 | |
