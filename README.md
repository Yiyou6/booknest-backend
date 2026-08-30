# 🏡 StayBooking

**你的下一个"爱彼迎"，从后端开始。** StayBooking 是一个短租预订平台的后端服务——房东发布带照片和定位的房源，房客在附近搜索心仪的住处、圈定入住日期，然后一键下单。

想象一下：把"树屋民宿"、"海景别墅"、"山间小筑"统统装进数据库，让它们在正确的时间、正确的距离内，自动匹配给正在找房子的人。这就是 StayBooking 干的事。

基于 **Spring Boot 4** + **Java 21** + **PostgreSQL / PostGIS** + **JWT 认证** 打造。

---

## ✨ 能做什么

- 🔐 **注册登录一把梭** —— JWT 无状态认证，两大角色各司其职：`ROLE_HOST`（房东）和 `ROLE_GUEST`（房客）。
- 🏠 **房源随心管理** —— 房东创建、查看、删除自己的房源，全程掌握在自己的手心里。
- 🖼️ **照片上云** —— 房源图片直传 Google Cloud Storage，UUID 命名、公开可读。
- 📍 **地址变坐标** —— 输入一个街道地址，Google Maps Geocoding 帮你算出经纬度。
- 🔎 **附近搜房** —— 房客以中心点 + 半径（米）划一个圈，配合入住/退房日期和入住人数，用 PostGIS 空间查询精准搜房。**已经撞档的房源会自动隐身**，绝不让你白跑一趟。
- 📅 **预订无忧** —— 房客下单、取消、查看自己的行程；房东查看名下房源的预订。**日期冲突直接拒绝**，避免"一间房租给两个人"的尴尬。

---

## 🧰 技术栈

| 模块       | 技术                                                               |
| ---------- | ------------------------------------------------------------------ |
| 语言       | Java 21                                                            |
| 框架       | Spring Boot 4.1.0（Web MVC / Data JPA / Security）                 |
| 构建       | Gradle                                                             |
| 数据库     | PostgreSQL + PostGIS（地理空间扩展）                               |
| ORM / 空间 | Hibernate ORM + Hibernate Spatial + JTS                            |
| 认证       | Spring Security + JJWT（JWT）                                      |
| 存储       | Google Cloud Storage                                               |
| 地理编码   | Google Maps Services（Geocoding API）                              |

---

## 🧩 准备工作

- JDK 21
- Docker 与 Docker Compose（本地跑 PostGIS 数据库用）
- 一个 Google Cloud Platform 项目，包含：
  - 一个 Cloud Storage 存储桶（存房源图片）
  - 一个服务账号密钥文件（`credentials.json`）
  - 一个 Geocoding API Key

---

## 🚀 快速上手

### 1. 拉起数据库

一条命令，PostGIS 版的 PostgreSQL 就位：

```bash
docker compose up -d
```

数据库跑在 `localhost:5432`，库名 `postgres`，密码 `secret`。

### 2. 配置密钥

应用通过环境变量读取外部配置，逐个填好：

| 变量                 | 说明                                |
| -------------------- | ----------------------------------- |
| `DATABASE_URL`       | 数据库主机（默认 `localhost`）      |
| `DATABASE_PORT`      | 数据库端口（默认 `5432`）           |
| `DATABASE_USERNAME`  | 数据库用户（默认 `postgres`）       |
| `DATABASE_PASSWORD`  | 数据库密码（默认 `secret`）         |
| `GCS_BUCKET`         | 存储房源图片的 GCS 存储桶名称       |
| `GEOCODING_KEY`      | Google Maps Geocoding API Key       |
| `JWT_SECRET_KEY`     | 签发 JWT 的密钥                     |

另外，把 GCP 服务账号密钥放到：

```
src/main/resources/credentials.json
```

### 3. 启动服务

```bash
./gradlew bootRun
```

服务在 `http://localhost:8080` 苏醒。

> 💡 **小提示：** 启动时 `DevRunner` 会自动灌入一批示例用户、房源和预订，开箱即玩。JPA 配置为 `ddl-auto: create-drop`，每次重启都会重建表结构（仅限开发环境）。

---

## 📡 API 一览

除认证接口外，所有请求都要在 `Authorization` 头里带上 `Bearer` Token。

### 认证

| 方法 | 接口             | 权限   | 说明                 |
| ---- | ---------------- | ------ | -------------------- |
| POST | `/auth/register` | 公开   | 注册新用户           |
| POST | `/auth/login`    | 公开   | 登录，返回 JWT       |

### 房源（房东）

| 方法   | 接口                             | 权限        | 说明                        |
| ------ | -------------------------------- | ----------- | --------------------------- |
| GET    | `/listings`                      | `ROLE_HOST` | 查看房东自己的房源          |
| POST   | `/listings`                      | `ROLE_HOST` | 创建房源（multipart 表单）  |
| DELETE | `/listings/{listingId}`          | `ROLE_HOST` | 删除房源                    |
| GET    | `/listings/{listingId}/bookings` | `ROLE_HOST` | 查看某房源的预订            |

### 搜索（房客）

| 方法 | 接口               | 权限         | 说明                       |
| ---- | ------------------ | ------------ | -------------------------- |
| GET  | `/listings/search` | `ROLE_GUEST` | 按位置和日期搜索房源       |

查询参数：`lat`、`lon`、`checkin_date`、`checkout_date`、`guest_number`，以及可选的 `distance`（米，默认 `500000`）。

### 预订（房客）

| 方法   | 接口                       | 权限         | 说明               |
| ------ | -------------------------- | ------------ | ------------------ |
| GET    | `/bookings`                | `ROLE_GUEST` | 查看房客的预订     |
| POST   | `/bookings`                | `ROLE_GUEST` | 创建预订           |
| DELETE | `/bookings/{bookingId}`    | `ROLE_GUEST` | 取消预订           |

---

## 🗂️ 项目结构

```
src/main/java/com/laioffer/staybooking/
├── authentication/   # 注册 / 登录服务与控制器
├── booking/          # 预订服务、控制器、异常
├── listing/          # 房源服务、控制器、异常
├── location/         # 地理编码（地址 → 经纬度）
├── model/            # JPA 实体、DTO、请求/响应记录
├── repository/       # Spring Data JPA 仓库
├── security/         # JWT 过滤器、处理器、UserDetailsService
├── storage/          # Google Cloud Storage 图片上传
├── AppConfig.java    # 安全链、Bean、GCS/地理编码配置
├── DevRunner.java    # 启动时灌入示例数据
└── StaybookingApplication.java
```

---

## ⚙️ 配置速查

核心配置在 `src/main/resources/application.yaml`：

- `spring.datasource.*` —— PostgreSQL 连接（由环境变量驱动）。
- `spring.jpa.hibernate.ddl-auto: create-drop` —— 每次运行重建表结构（仅开发）。
- `spring.servlet.multipart.max-file-size: 10MB` —— 图片上传上限。
- `spring.sql.init.schema-locations: postgis_extension.sql` —— 启动时启用 PostGIS 扩展。

---

## 🧪 测试

```bash
./gradlew test
```

---

## 📄 License

本项目仅用于学习目的。
