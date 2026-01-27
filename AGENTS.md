# AGENTS.md - Strapi项目开发指南

本文档为AI代理和开发者提供此Strapi项目的开发规范和最佳实践。

## 项目信息

- **框架**: Strapi v5.33.3
- **语言**: TypeScript
- **数据库**: SQLite (开发环境) / 支持MySQL、PostgreSQL
- **Node版本**: >=20.0.0 <=24.x.x

---

## 构建命令

### 开发
```bash
npm run develop      # 启动开发服务器（带自动重载）
npm run dev          # 开发服务器别名
```

### 生产
```bash
npm run build        # 构建管理面板
npm run start        # 启动生产服务器
```

### 其他命令
```bash
npm run console      # 启动Strapi控制台
npm run deploy       # 部署到Strapi Cloud
npm run seed:example # 加载示例数据
npm run upgrade      # 升级Strapi到最新版本
npm run upgrade:dry  # 试运行升级（不实际执行）
```

---

## 项目结构

```
src/
├── api/                    # API端点
│   └── {content-type}/
│       ├── content-types/  # 内容类型schema（JSON）
│       ├── controllers/    # 控制器（TypeScript）
│       ├── services/       # 服务层（TypeScript）
│       └── routes/         # 路由（TypeScript）
├── components/             # 可重用组件
│   └── {category}/          # 组件schema（JSON）
├── admin/                  # 管理面板前端（React/TypeScript）
└── index.ts               # 应用入口
config/                     # 配置文件
types/generated/           # 自动生成的类型定义
```

---

## 代码风格指南

### 导入规范

```typescript
// 优先使用Strapi核心导出
import { factories } from '@strapi/strapi';
import type { Core } from '@strapi/strapi';

// Node.js核心模块
import path from 'path';

// 绝对导入使用@strapi命名空间
import type { StrapiApp } from '@strapi/strapi/admin';
```

### 文件格式

#### Controllers (控制器)
```typescript
/**
 * {content-type} controller
 */

import { factories } from '@strapi/strapi';

export default factories.createCoreController('api::{content-type}.{content-type}');
```

#### Services (服务)
```typescript
/**
 * {content-type} service
 */

import { factories } from '@strapi/strapi';

export default factories.createCoreService('api::{content-type}.{content-type}');
```

#### Routes (路由)
```typescript
/**
 * {content-type} router
 */

import { factories } from '@strapi/strapi';

export default factories.createCoreRouter('api::{content-type}.{content-type}');
```

#### Schema (内容类型)
```json
{
  "kind": "collectionType" | "singleType",
  "collectionName": "{plural-name}",
  "info": {
    "singularName": "{singular}",
    "pluralName": "{plural}",
    "displayName": "Display Name",
    "description": "Description"
  },
  "options": {
    "draftAndPublish": true
  },
  "attributes": {}
}
```

### 命名约定

- **API路径**: `api::{content-type}.{content-type}` (例如: `api::article.article`)
- **文件名**: 小写，使用连字符分隔单词（schema除外）
- **集合名称**: 小写复数（例如: `articles`）
- **单数名称**: 小写单数（例如: `article`）
- **类型定义**: 使用PascalCase（例如: `Article`）
- **变量/函数**: 使用camelCase（例如: `getArticleById`）

### 类型规范

- 使用TypeScript的`type`关键字（而非`interface`）
- 导入时使用`type`关键字仅导入类型：
  ```typescript
  import type { StrapiApp } from '@strapi/strapi/admin';
  ```
- 利用自动生成的类型定义在`types/generated/`目录中

### 配置文件

#### 配置导出模式
```typescript
// 对象模式（无参数）
export default {
  rest: {
    defaultLimit: 25,
  },
};

// 函数模式（接收环境）
export default ({ env }) => ({
  host: env('HOST', '0.0.0.0'),
  port: env.int('PORT', 1337),
});
```

### 注释风格

使用JSDoc风格的块注释：
```typescript
/**
 * An asynchronous register function that runs before
 * your application is initialized.
 */
register(/* { strapi }: { strapi: Core.Strapi } */) {},
```

### 环境变量

使用`env()`辅助函数获取环境变量：
```typescript
const host = env('HOST', '0.0.0.0');        // 字符串（带默认值）
const port = env.int('PORT', 1337);         // 整数（带默认值）
const ssl = env.bool('DATABASE_SSL', false); // 布尔值（带默认值）
const keys = env.array('APP_KEYS');        // 数组
```

### Content Type属性

```json
{
  "string": {
    "type": "string"
  },
  "text": {
    "type": "text",
    "maxLength": 255
  },
  "uid": {
    "type": "uid",
    "targetField": "title"
  },
  "media": {
    "type": "media",
    "multiple": false,
    "required": false,
    "allowedTypes": ["images", "files", "videos"]
  },
  "relation": {
    "type": "relation",
    "relation": "manyToOne" | "oneToMany" | "manyToMany" | "oneToOne",
    "target": "api::{target}.{target}",
    "inversedBy": "{relation-field}"
  },
  "dynamiczone": {
    "type": "dynamiczone",
    "components": ["shared.{component}", ...]
  }
}
```

### 组件命名

组件路径格式：`{category}.{component-name}`
- 示例：`shared.media`、`shared.quote`、`shared.rich-text`
- 组件文件存放在`src/components/{category}/{name}.json`

### Admin自定义

管理面板使用React/TypeScript，入口文件：`src/admin/app.tsx`

```typescript
import type { StrapiApp } from '@strapi/strapi/admin';

export default {
  config: {
    locales: ['zh'],  // 默认中文
  },
  bootstrap(app: StrapiApp) {
    // 自定义逻辑
  },
};
```

---

## 关键注意事项

1. **无Linting/Formatting配置** - 项目未配置ESLint或Prettier，请手动保持代码风格一致
2. **测试** - 项目当前未配置测试框架
3. **类型安全** - 虽然TypeScript严格模式关闭，但建议使用类型定义
4. **数据库** - 默认使用SQLite，生产环境建议配置MySQL或PostgreSQL
5. **环境变量** - 查看`.env.example`了解必需的环境变量

---

## 常见任务

### 创建新的Content Type
```bash
npx strapi generate content-type article
```

### 创建新的组件
```bash
npx strapi generate component shared quote
```

### 创建API端点
使用Strapi工厂模式在相应目录创建controller/service/route文件

### 自定义中间件
在`config/middlewares.ts`中添加自定义中间件

---

## 安全最佳实践

1. 始终使用`.env`文件管理敏感信息，不要提交到版本控制
2. 生产环境必须更改默认的密钥（APP_KEYS, JWT_SECRET等）
3. 使用Strapi内置的角色和权限系统
4. 配置适当的CORS策略

---

## 代码提交

提交前请运行构建确保无错误：
```bash
npm run build
```
