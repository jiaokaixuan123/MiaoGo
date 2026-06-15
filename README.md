# 秒购（MiaoGo）— 高并发电商秒杀交易平台

> 整点不超卖，尖峰不宕机。

一套从零搭建、支撑限时秒杀与限量发售等高并发营销活动的电商交易系统。核心目标：**在流量尖峰下保证不超卖、不丢单、数据库不被打垮。**

> 这是个人练手 / 高并发场景模拟与压测验证项目。采用「单体起步 → 压测暴露瓶颈 → 针对性引入中间件」的演进式开发，每个阶段都用 JMeter 压测验证收益。完整方案见 [doc/秒购-项目方案.md](doc/秒购-项目方案.md)。

---

## 技术栈

| 层 | 选型 | 引入阶段 |
|---|---|---|
| 语言 / 框架 | Java 17 + Spring Boot 3.x | Phase 0 |
| 持久层 | MyBatis-Plus | Phase 0 |
| 数据库 | MySQL 8.x（InnoDB） | Phase 0 |
| 缓存 | Redis 7.x + Caffeine（多级缓存） | Phase 2 |
| 分布式锁 | Redisson | Phase 1 |
| 消息队列 | RocketMQ（事务消息 + 延迟消息） | Phase 3 |
| 分库分表 | Apache ShardingSphere | Phase 4 |
| 搜索 | Elasticsearch | Phase 4 |
| 微服务 | Spring Cloud Alibaba（Nacos / Gateway / Sentinel / Seata / OpenFeign） | Phase 5 |
| 分布式任务 | XXL-JOB | Phase 5 |
| 可观测 | Prometheus + Grafana + SkyWalking + ELK | Phase 6 |
| 压测 | JMeter（贯穿全程） | 全程 |

---

## 架构演进路线

```
Phase0 单体           ──> Phase1 并发治理（锁 / Redis 预扣）
   │                          │
   ▼                          ▼
Phase2 多级缓存        ──> Phase3 MQ 削峰 / 解耦 / 延迟队列
   │                          │
   ▼                          ▼
Phase4 分库分表 / ES   ──> Phase5 微服务化（Nacos/Sentinel/Seata/XXL-JOB）
                              │
                              ▼
                        Phase6 可观测 + 全链路压测闭环
```

核心理念：**问题驱动而非技术驱动**——先用压测制造出"非用不可的痛点"，再引入对应技术解决。

---

## 进度看板

- [ ] **Phase 0** 单体跑通业务（下单链路、状态机、责任链、本地事务）
- [ ] **Phase 1** 制造并发，治理超卖（乐观锁 / 悲观锁 / Redis 预扣 / Redisson）
- [ ] **Phase 2** 多级缓存（穿透 / 击穿 / 雪崩 / 一致性）
- [ ] **Phase 3** 消息队列（削峰 / 解耦 / 事务消息 / 延迟取消）
- [ ] **Phase 4** 分库分表 + 搜索（分片键 / 冗余表 / 慢查询优化 / ES）
- [ ] **Phase 5** 微服务化（Nacos / Gateway / Sentinel / Seata / XXL-JOB）
- [ ] **Phase 6** 可观测 + 全链路压测闭环（监控 / 日志 / 性能报告）

---

## 本地环境要求

| 组件 | 版本 | 说明 |
|---|---|---|
| JDK | 17 | Spring Boot 3 最低要求 |
| Maven | 3.9+ | 构建 |
| MySQL | 8.x | Phase 0 起 |
| Docker | 任意 | 推荐用容器起 MySQL/Redis 等中间件 |

启动本地 MySQL（示例）：

```bash
docker run -d --name miaogo-mysql -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=miaogo mysql:8
```

---

## 快速开始

```bash
# 1. 准备数据库（执行建表脚本，待 Phase 0 补充）
#    src/main/resources/db/schema.sql

# 2. 配置本地数据源（敏感配置放 application-local.yml，已在 .gitignore 中）

# 3. 启动
mvn spring-boot:run
```

> 敏感配置（数据库密码、第三方 API key）请放入 `application-local.yml`，**不要提交到 Git**。

---

## 工程实践

- **分支模型**：GitHub Flow，每个 Phase 拉 `feature/phaseN-xxx` 分支。
- **提交规范**：Conventional Commits（如 `feat(order): 实现单体下单核心链路`）。
- **技术笔记**：每解决一个问题写一篇（现象 → 排查 → 候选方案 → 选型理由 → 复盘），沉淀在 `doc/notes/`。
- **压测先行**：每个 Phase 引入技术前后都做 JMeter 压测对比，用数据说话。

---

## 目录结构（规划中）

```
MiaoGo/
├── doc/                    # 方案文档、技术笔记、压测报告
│   └── 秒购-项目方案.md
├── src/main/java/com/miaogo/
│   ├── controller/         # 接口层
│   ├── service/            # 业务逻辑
│   ├── mapper/             # MyBatis-Plus Mapper
│   ├── domain/             # entity / dto / vo
│   ├── common/             # 统一返回、异常、枚举
│   ├── config/             # 配置（线程池等）
│   └── order/              # 状态机、责任链
└── src/main/resources/
    ├── application.yml
    └── db/schema.sql
```
