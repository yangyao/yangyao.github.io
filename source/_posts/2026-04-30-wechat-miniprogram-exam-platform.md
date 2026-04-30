---
layout: post
title: 从零构建微信小程序刷题平台：守正题库开发实录
date: 2026-04-30 10:00:00
tags:
- wechat
- miniprogram
- nextjs
- typescript
- postgresql
categories:
- engineering
---

守正题库是一个面向 FOCO 品牌的微信小程序刷题考试平台。用户通过小程序进行日常练习、模拟考试，管理端通过 Web 后台管理题库内容和用户数据。整个项目从零搭建，涵盖小程序端、管理后台、数据库、支付系统等完整链路。

<!-- more -->

## 技术选型

### 小程序端

| 选型 | 决定 | 原因 |
|------|------|------|
| 语言 | TypeScript | 类型安全，减少运行时错误 |
| 样式 | Less | 嵌套、变量支持，比原生 wxss 更高效 |
| 组件框架 | Glass-easel | 微信新一代组件框架，性能更优 |
| 渲染器 | Skyline | 相比 WebView 渲染，滚动和动画更流畅 |

### 管理后台

| 选型 | 决定 | 原因 |
|------|------|------|
| 框架 | Next.js 16 (App Router) | API Routes 同时承担后端职责，全栈一套代码 |
| UI 库 | shadcn/ui + Tailwind CSS | 组件质量高，可定制，无需自己造轮子 |
| ORM | Drizzle | 类型安全的 SQL 查询，迁移体验好 |
| 数据库 | PostgreSQL | JSON 支持好（题目选项），生态成熟 |
| 认证 | NextAuth.js (后台) + JWT (小程序) | 两套独立认证体系，各取所需 |
| 部署 | 腾讯云 EdgeOne Pages | 国内访问快，与微信生态打通 |

### 为什么 Next.js 兼做后端？

很多人会问：小程序为什么不用独立的后端服务？实际上 Next.js 的 API Routes 完全能胜任这个场景：

- **减少运维复杂度**：一个 Next.js 应用同时服务管理后台页面和小程序 API
- **共享数据库层**：Drizzle schema、工具函数在前后端间复用
- **部署简单**：EdgeOne Pages 一个应用搞定，不需要额外配置服务器
- **类型贯穿**：从数据库 schema 到 API 响应，TypeScript 类型全程覆盖

## 架构设计

### 整体架构

```
┌─────────────────┐     ┌──────────────────────────────────┐
│  微信小程序       │     │  Next.js 管理后台 (EdgeOne)       │
│  (用户端)         │────▶│                                  │
│                  │     │  /api/v1/*        ← JWT 认证     │
│  TypeScript      │     │  /api/admin/v1/*  ← Session 认证 │
│  Less            │     │  /api/auth/*      ← NextAuth     │
│  Glass-easel     │     │  /api/callbacks/* ← 微信支付回调  │
│  Skyline         │     │                                  │
└─────────────────┘     │         Drizzle ORM               │
                        │            │                      │
                        │     ┌──────▼──────┐               │
                        │     │ PostgreSQL  │               │
                        │     └─────────────┘               │
                        └──────────────────────────────────┘
```

### 双认证体系

系统有两套独立的认证机制：

**管理后台认证**：NextAuth.js Credentials Provider → Cookie-based Session。管理员通过用户名密码登录，中间件 (`middleware.ts`) 拦截未认证请求跳转登录页。

**小程序用户认证**：`wx.login()` 获取微信 code → 服务端用 code 换取 openid → 签发 JWT → 小程序存储在本地。`request.ts` 中封装了自动重登录逻辑：遇到 401 时自动重新调用 `wx.login()` 获取新 token。

### API 设计规范

所有接口遵循统一的响应格式：

```json
// 成功
{ "code": 0, "message": "ok", "data": { ... } }

// 失败
{ "code": -1, "message": "错误描述", "data": null }

// 分页
{ "items": [], "page": 1, "pageSize": 20, "total": 100 }
```

### 数据库设计

21 张表，覆盖完整业务：

- **内容体系**：分类 (`bank_categories`) → 题库 (`question_banks`) → 版本 (`bank_editions`) → 章节 (`bank_sections`) → 题目 (`questions`)，五级结构
- **学习功能**：练习会话 (`practice_sessions`)、答题记录 (`practice_answers`)、错题 (`wrong_questions`)、收藏 (`favorites`)、笔记 (`notes`)
- **考试系统**：试卷 (`mock_papers`)、考试会话和答案
- **商业体系**：商品 (`products`)、订单 (`orders`)、支付 (`payments`)、钱包 (`wallet`)、激活码 (`activation_codes`)
- **权限控制**：题库授权 (`bank_entitlements`)

## 开发历程

整个项目通过 OpenSpec 管理 24 个变更（c01-c24），按模块递进开发：

### Phase 1：管理后台基础（c01-c06）

```
c01  项目初始化       → Next.js 16 + shadcn/ui + Tailwind 搭建
c02  数据库迁移       → Drizzle 配置、连接 PostgreSQL、迁移流程
c03  管理后台认证     → NextAuth.js Credentials Provider
c04  分类管理 CRUD    → 题库分类的增删改查
c05  版本与章节       → 题库下的版本、章节管理
c06  题目管理         → 题目 CRUD，支持单选/多选/判断
```

这个阶段的目标很明确：先把后台管理能力搭起来，能够录入题库内容。选 Drizzle 而非 Prisma 是因为它的 schema 写法更接近 SQL，生成的迁移文件也更可控。

### Phase 2：小程序核心体验（c07-c12）

```
c07  试卷管理         → 模拟考试试卷的创建和题目编排
c08  微信登录         → wx.login + openid + JWT 签发
c09  首页与题库目录   → Banner、分类筛选、题库列表
c10  练习会话 API     → 开始练习、提交答案、获取下一题
c11  练习页面         → 题目渲染、滑动切换、答题交互
c12  错题与历史       → 错题本、练习历史回顾
```

这是最核心的用户体验阶段。练习页面 (`practice`) 是整个小程序最复杂的页面——题目渲染需要支持三种题型、实时反馈正误、收藏/笔记操作、答题卡快速跳转、练习结束统计。组件拆分为 `question-renderer`（题目渲染）、`answer-sheet`（答题卡）、`practice-footer`（底部操作栏），各司其职。

小程序的网络层封装在 `services/request.ts`，一个关键设计是 401 自动重登录：请求失败时检测状态码，如果是 token 过期就自动 `wx.login()` 重新获取 token 并重试原请求，用户完全无感知。

### Phase 3：考试与个人功能（c13-c15）

```
c13  模拟考试 API     → 开考、交卷、评分接口
c14  模拟考试页面     → 倒计时、答题、自动交卷
c15  收藏与笔记       → 收藏题目、记录学习笔记
```

模拟考试的挑战在于限时机制。需要在页面进入时记录开始时间，设置倒计时，时间到自动提交。同时要处理用户手动提前交卷和异常退出的情况。

### Phase 4：商业化（c16-c22）

```
c16  商品 SKU 管理    → 题库作为商品上架
c17  题库授权         → 购买/激活码解锁题库访问权限
c18  资源包           → 学习资料的打包分发
c20  微信支付接入     → JSAPI 支付、回调处理
c21  钱包系统         → 余额、充值、消费记录
c22  激活码           → 批量生成、核销、关联题库
```

支付是最复杂的部分。微信支付涉及多个环节：前端调用 `wx.requestPayment`、后端统一下单、支付结果回调通知、退款流程。回调接口放在 `/api/callbacks/wechat/` 路径下，不需要认证（微信服务器回调），但需要验签确保请求来自微信。

钱包系统采用余额模式：用户充值到钱包 → 购买时从钱包扣款。流水记录在 `wallet` 表中，每笔交易有类型（充值/消费/退款）和关联订单。

### Phase 5：数据与运营（c23-c24）

```
c23  学习统计         → 练习数据汇总、学习轨迹
c24  管理后台仪表盘   → 数据概览、图表展示
```

## 关键技术决策

### 1. BigInt ID + JSON 序列化

数据库使用 BigInt 作为主键，但 JSON.stringify 不支持 BigInt。在 `api-utils.ts` 中 patch 了 `BigInt.prototype.toJSON`，让它序列化为字符串。这样前端拿到的 ID 都是字符串，传回后端时 Drizzle 能自动处理类型转换。

### 2. Skyline 渲染器

小程序启用了 Skyline 渲染器（在 `app.json` 的 `rendererOptions` 中配置），配合 Glass-easel 组件框架。Skyline 相比传统 WebView 渲染，在长列表滚动和动画方面性能更好。但需要注意兼容性，部分原生组件的行为可能不同。

### 3. 小程序本地代理

开发时，小程序通过 `request.ts` 中的代理配置将 API 请求转发到 `http://127.0.0.1:3001`（Next.js 开发服务器）。生产环境则指向 EdgeOne 的线上地址。这样开发时不需要额外的代理工具。

### 4. OpenSpec 驱动开发

整个项目使用 OpenSpec 管理需求变更（24 个 change 目录），每个变更包含完整的设计、规格和任务拆解。这种方式的优点是：

- 开发前先想清楚再动手，减少返工
- 每个变更粒度适中，可以独立审查和回滚
- 变更记录就是最好的开发文档

## 项目数据

| 指标 | 数值 |
|------|------|
| 小程序代码 | ~6,400 行 (ts + wxml + less) |
| 管理后台代码 | ~11,900 行 (ts + tsx) |
| 数据库表 | 21 张 |
| API 端点 | ~30 个 |
| 小程序页面 | 19 个 |
| 需求变更 | 24 个 |
| 开发周期 | 约 2 天（密集开发） |

## 踩坑记录

### Next.js 16 的坑

Next.js 16 有一些破坏性变更，特别是 App Router 的行为调整。遇到问题时需要查阅 `node_modules/next/dist/docs/` 中的文档，网上的资料很多已经过时。

### EdgeOne 部署的 trustHost

部署到 EdgeOne 后，NextAuth 报 `trustHost` 错误。原因是 EdgeOne 的反向代理改变了请求的 host，需要在 NextAuth 配置中显式设置 `trustHost: true`。

### 微信支付的回调验证

微信支付回调需要在没有认证中间件的情况下接收，同时要验证签名防止伪造请求。回调路由放在 `/api/callbacks/wechat/` 下，绕过认证中间件，但自行实现 XML 解析和签名校验。

## 总结

这个项目验证了一个思路：**用 Next.js 同时承担管理后台和小程序后端是完全可行的**。对于中小规模的业务，这种架构大大降低了运维复杂度。TypeScript 从数据库 schema 到 API 再到小程序的全程类型覆盖，让开发体验非常流畅。

主要收益：

- 一套代码同时服务 Web 后台和 API 后端
- Drizzle ORM 类型安全的数据库操作
- OpenSpec 规范化的需求管理流程
- Skyline + Glass-easel 带来的小程序性能提升

未来可以改进的方向：

- 加入单元测试和 E2E 测试
- 考虑将 API 层抽离为独立服务（如果业务量增长）
- 加入更多学习数据分析维度
- 国际化支持
