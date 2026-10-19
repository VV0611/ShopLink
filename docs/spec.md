# 小网店下单系统 — 项目规格 (Project Spec)

> 暂定名：**ShopLink**（可之后再改）
> 版本：v0.1（MVP 规划）
> 第一个真实用户：Snack Lah

---

## 1. 项目概述

### 1.1 一句话定位
让马来西亚的小型网店卖家（IG / FB / 小红书 / WhatsApp 卖家）**几分钟开好一个下单页面**，取代"私讯 → 报价 → 转账截图 → 手记 Excel"的混乱流程。

### 1.2 解决的问题
| 现在的痛点 | 我们的解决方式 |
|---|---|
| 订单散落在 DM / WhatsApp，容易漏单 | 所有订单集中在一个后台 |
| 每个客人都要重复问价格、规格、库存 | 店面页面清楚列出，客人自己选 |
| 转账截图难对账 | 订单绑定付款凭证，一眼看出已付 / 未付 |
| 预购截单、成团数量靠人工数 | 系统自动倒数、自动统计 |
| Shopify / EasyStore 太贵太复杂，Shopee 抽佣 | 轻量、便宜、专为社交媒体卖家设计 |

### 1.3 目标用户
- **卖家 (Seller)**：个人或小团队卖家，每月几十到几百单，没有技术背景
- **买家 (Customer)**：通过卖家分享的链接进来，**不需要注册**就能下单

### 1.4 核心概念
每个卖家得到一个专属店面链接：`shoplink.my/{shop-slug}`，例如 `shoplink.my/snacklah`，放在 IG bio 或 WhatsApp 就能接单。

---

## 2. 技术栈 (Tech Stack)

| 层 | 选择 | 原因 |
|---|---|---|
| Frontend | **Next.js** (App Router) + TypeScript | 店面需要 SEO、手机打开快 |
| Styling | Tailwind CSS + shadcn/ui | 开发快、好看 |
| Backend / DB | **Supabase** (Postgres) | Auth、数据库、Storage、Realtime 一站搞定 |
| Auth | Supabase Auth（Email / Google） | 只有卖家需要登录 |
| Storage | Supabase Storage | 商品图片、付款截图 |
| Realtime | Supabase Realtime | 新订单即时通知 |
| Deployment | **Vercel** + GitHub | push 自动部署 |
| Payment (Phase 2) | Billplz / ToyyibPay | 本地 FPX / DuitNow |
| Shipping (Phase 3) | EasyParcel API | 批量运单 |

### 2.1 多租户设计 (Multi-tenancy) ⚠️ 最关键
- 所有业务数据表都带 `shop_id`
- 用 Supabase **Row Level Security (RLS)** 确保卖家只能读写自己店的数据
- 买家（未登录）只能：读取已上架商品、创建订单、用订单号 + 电话查询自己的订单

---

## 3. 用户角色 (Roles)

| 角色 | 需要登录？ | 能做什么 |
|---|---|---|
| Customer 买家 | ❌ | 浏览店面、下单、上传付款凭证、查订单 |
| Seller 卖家 | ✅ | 管理店铺、商品、订单 |
| Admin 平台管理员 | ✅ | 管理所有店铺（Phase 2 才做） |

---

## 4. 功能清单 (Features)

优先级：🔴 MVP 必做 ｜ 🟡 Phase 2 ｜ 🟢 Phase 3 / Nice to have

### 4.1 卖家端 — 账号与店铺
- 🔴 注册 / 登录 / 忘记密码
- 🔴 创建店铺：店名、slug（店面网址）、logo、简介、联系 WhatsApp 号码
- 🔴 店铺设置：营业状态（开 / 暂停接单）、货币 (MYR)
- 🔴 付款资料设置：银行名称、户口名字、户口号码、DuitNow QR 图片
- 🔴 配送设置：自取（地点说明）/ 邮寄（运费：固定价 或 按西马 / 东马分区）
- 🟡 店铺主题颜色、横幅图片
- 🟡 多语言店面（English / 华文 / BM）

### 4.2 卖家端 — 商品管理
- 🔴 新增 / 编辑 / 删除商品：名称、描述、价格、图片（多张）、分类
- 🔴 商品规格 (Variants)：例如口味、尺寸，每个规格独立价格与库存
- 🔴 库存管理：库存数量，卖完自动显示 **Sold Out**
- 🔴 上架 / 下架切换
- 🔴 商品分类管理
- 🟡 **预购模式 (Pre-order)**：截单时间、预计出货日期、倒数计时
- 🟡 **团购模式**：最低成团数量，进度条显示
- 🟢 批量导入商品 (CSV)

### 4.3 卖家端 — 订单管理
- 🔴 订单列表：按状态筛选、搜索（订单号 / 名字 / 电话）
- 🔴 订单详情：商品、数量、金额、客人资料、配送方式、付款凭证
- 🔴 更新订单状态（见第 6 节状态流程）
- 🔴 查看 / 放大付款截图，确认收款
- 🔴 一键 WhatsApp 联系客人（自动带入订单信息）
- 🔴 填写运单号 (Tracking number)，客人可在查询页看到
- 🔴 新订单即时通知（Realtime + 页面提示音）
- 🟡 一键提醒未付款客人（生成 WhatsApp 讯息）
- 🟡 取消订单时自动退回库存
- 🟡 导出订单 CSV
- 🟢 导出 J&T / Poslaju / Ninja Van 格式；接 EasyParcel 打印面单

### 4.4 卖家端 — 数据面板 (Dashboard)
- 🔴 今日 / 本月订单数、营业额
- 🔴 待处理订单数量（待确认付款、待出货）
- 🟡 热卖商品排行
- 🟡 回头客统计
- 🟢 营业额趋势图

### 4.5 买家端 — 店面
- 🔴 店面首页：店铺资料、商品列表、分类筛选
- 🔴 商品详情页：图片、描述、选规格、选数量
- 🔴 购物车（存在 localStorage，不需注册）
- 🔴 手机优先 (Mobile-first) 设计
- 🟡 商品搜索
- 🟡 分享商品链接（带预览图 Open Graph）

### 4.6 买家端 — 下单与付款
- 🔴 结账表单：名字、电话、配送方式、地址（邮寄时）、备注
- 🔴 自动计算：商品小计 + 运费 = 总额
- 🔴 下单前再次检查库存
- 🔴 下单成功页：订单号、付款资料（银行 / QR）、上传付款截图按钮
- 🔴 上传付款凭证（下单后也可以回来补上传）
- 🟡 WhatsApp 一键下单：生成格式化订单讯息发给卖家
- 🟡 线上付款 (Billplz / ToyyibPay)，付款成功自动标记已付款
- 🟢 优惠码 (Voucher)

### 4.7 买家端 — 订单查询
- 🔴 用 **订单号 + 电话号码** 查询订单状态（不需注册）
- 🔴 显示状态进度、运单号
- 🟢 买家账号（保存地址、历史订单）

### 4.8 平台管理员 (Phase 2)
- 🟡 店铺列表、停用违规店铺
- 🟡 平台整体数据
- 🟢 订阅方案与收费

---

## 5. 数据库设计 (Database Schema) — 初版

```
shops
  id (uuid, PK)
  owner_id (uuid → auth.users)
  name, slug (unique), logo_url, description
  whatsapp_number
  is_open (bool)
  bank_name, bank_account_name, bank_account_number, duitnow_qr_url
  pickup_enabled (bool), pickup_note
  shipping_enabled (bool), shipping_fee_west, shipping_fee_east
  created_at

categories
  id, shop_id → shops, name, sort_order

products
  id, shop_id → shops, category_id → categories (nullable)
  name, description, base_price
  is_active (bool)
  is_preorder (bool), preorder_deadline, preorder_ship_date   -- Phase 2
  created_at

product_images
  id, product_id → products, url, sort_order

product_variants
  id, product_id → products
  name (e.g. "辣味 / 500g"), price, stock
  sku (nullable)

orders
  id, shop_id → shops
  order_number (short readable, e.g. SL-000123)
  customer_name, customer_phone
  delivery_method ('pickup' | 'shipping')
  shipping_address, shipping_region ('west' | 'east')
  note
  subtotal, shipping_fee, total
  status (见第 6 节)
  payment_proof_url, paid_at
  tracking_number, courier
  created_at, updated_at

order_items
  id, order_id → orders
  product_id, variant_id
  product_name, variant_name, unit_price, quantity   -- 下单时快照，商品改价不影响旧订单
  line_total
```

> ⚠️ `order_items` 必须**快照**商品名与价格，不能只存 ID。
> ⚠️ 库存扣减要用 Postgres function / transaction，避免两个人同时买最后一件。

---

## 6. 订单状态流程 (Order Status Flow)

```
pending_payment ──(客人上传截图)──▶ payment_submitted ──(卖家确认)──▶ paid
       │                                   │                          │
       │                                   └──(截图有问题)──▶ pending_payment
       ▼                                                              ▼
   cancelled ◀───────────────(卖家取消)──────────────────────    processing
                                                                      │
                                              ┌───────────────────────┴───────┐
                                              ▼                               ▼
                                      ready_for_pickup                     shipped
                                              │                               │
                                              └──────────▶ completed ◀────────┘
```

| 状态 | 中文 | 谁触发 |
|---|---|---|
| `pending_payment` | 待付款 | 系统（下单时） |
| `payment_submitted` | 已上传凭证，待确认 | 买家上传截图 |
| `paid` | 已确认付款 | 卖家 |
| `processing` | 准备中 | 卖家 |
| `ready_for_pickup` | 可自取 | 卖家 |
| `shipped` | 已出货 | 卖家（填运单号） |
| `completed` | 完成 | 卖家 |
| `cancelled` | 已取消 | 卖家 |

---

## 7. 页面清单 (Pages / Routes)

### 公开（买家）
| 路由 | 页面 |
|---|---|
| `/` | 平台首页（介绍 + 卖家注册入口） |
| `/[slug]` | 店面首页 |
| `/[slug]/product/[id]` | 商品详情 |
| `/[slug]/cart` | 购物车 |
| `/[slug]/checkout` | 结账 |
| `/[slug]/order/[orderNumber]` | 下单成功 / 付款资料 / 上传凭证 |
| `/[slug]/track` | 订单查询 |

### 卖家后台（需登录）
| 路由 | 页面 |
|---|---|
| `/login`、`/signup` | 登录 / 注册 |
| `/onboarding` | 首次创建店铺 |
| `/dashboard` | 数据面板 |
| `/dashboard/orders` | 订单列表 |
| `/dashboard/orders/[id]` | 订单详情 |
| `/dashboard/products` | 商品列表 |
| `/dashboard/products/new`、`/[id]` | 新增 / 编辑商品 |
| `/dashboard/categories` | 分类管理 |
| `/dashboard/settings` | 店铺、付款、配送设置 |

---

## 8. 非功能需求 (Non-functional)
- **Mobile-first**：90% 买家用手机打开
- **速度**：店面首屏 < 2 秒（Next.js SSR + 图片压缩）
- **安全**：RLS 全开；付款截图存 private bucket，只有该店卖家能看
- **简单**：卖家 10 分钟内能完成开店并上架第一个商品
- **防滥用**：结账表单 rate limit，图片上传限制大小与格式

---

## 9. 开发里程碑 (Milestones)

| 阶段 | 内容 | 完成标准 |
|---|---|---|
| **M0 Setup** | GitHub repo、Next.js + Tailwind + Supabase 骨架、Vercel 部署 | 空白网站能上线 |
| **M1 卖家基础** | 注册登录、创建店铺、店铺设置 | 能开好一间店 |
| **M2 商品** | 分类、商品 CRUD、图片上传、规格与库存 | 能上架商品 |
| **M3 店面** | 店面页、商品页、购物车 | 买家能浏览加购物车 |
| **M4 下单** | 结账、库存扣减、下单成功页、上传凭证 | 买家能完成一单 |
| **M5 订单管理** | 订单列表 / 详情、状态更新、WhatsApp 联系、运单号、Realtime 通知 | 卖家能处理订单 |
| **M6 查询 + 面板** | 订单查询页、Dashboard 数据 | **MVP 完成 → Snack Lah 实际使用** |
| Phase 2 | 预购 / 团购、线上付款、提醒未付款、导出 CSV、多语言 | — |
| Phase 3 | EasyParcel、优惠码、买家账号、订阅收费 | — |

---

## 10. 待决定事项 (Open Questions)
- [ ] 项目正式名称与域名
- [ ] 是否一个卖家可以有多间店？（建议 MVP 先一人一店）
- [ ] 东马 / 西马之外是否需要更细的运费规则？
- [ ] 未来收费模式：免费 + 抽成？月费订阅？
