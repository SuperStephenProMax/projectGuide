这是一个**非常好的问题**，而且你问的正是**工程成熟度的分水岭** 👍
日志（log）不是“随手打几行 println”，而是**可观测性、排障效率、合规审计**的基础。

下面我从**“为什么 → 什么时候打 → 打什么 → 怎么打 → 常见反模式”**五个层次，给你一套**可直接落地的日志规范（偏 Java / Spring Boot，但通用）**。

---

# 一、为什么要有日志规范（先统一认知）

日志的核心目标只有 4 个：

1. **定位问题（Debug / Incident）**
2. **理解系统行为（Observability）**
3. **审计与合规（Audit）**
4. **性能与容量分析（Perf / Capacity）**

👉 **日志不是给“人随便看”的，而是给：**

* 人（开发 / 运维 / 安全）
* 机器（ELK / Loki / Datadog / Splunk）
* 自动化规则（告警 / SLO / 异常检测）

所以日志**必须结构化、可搜索、可关联**。

---

# 二、什么时候应该打印日志（What to log）

## 1️⃣ 必须打日志的场景（硬性）

### ✅ 系统生命周期事件

* 服务启动 / 关闭
* 配置加载完成
* 关键依赖初始化成功/失败

```text
INFO  Application started, profile=prod, port=8080
```

---

### ✅ 请求/任务的「入口 & 出口」

* HTTP 请求
* MQ 消费
* 定时任务

```text
INFO  Incoming request: method=POST path=/api/orders requestId=abc123
INFO  Request finished: path=/api/orders status=200 costMs=34
```

---

### ✅ 状态变化（State Change）

> “状态改变” 比 “做了什么” 更重要

```text
INFO  Order status changed: orderId=123 from=CREATED to=PAID
```

---

### ✅ 异常（所有异常必须有日志）

* **被吞掉的异常 = 事故**
* 只 catch 不 log 是大忌

```java
catch (IOException e) {
    log.error("Failed to read config file, path={}", path, e);
    throw e;
}
```

---

### ✅ 外部系统交互（边界）

* 调用 DB / Redis / MQ / HTTP
* **失败必须打日志，成功通常只在 debug**

```text
WARN  Call payment service failed, orderId=123, retry=1, errorCode=TIMEOUT
```

---

## 2️⃣ 应该少打 / 不打日志的场景

❌ 循环内每一步
❌ 高频成功路径 INFO
❌ println / System.out
❌ “我到这里了”这种无信息日志

```java
// ❌ 错误示例
log.info("enter method");
log.info("step 1");
log.info("step 2");
```

---

# 三、日志级别规范（非常重要）

**统一级别语义，否则日志毫无价值**

| 级别    | 含义         | 是否默认开启 |
| ----- | ---------- | ------ |
| ERROR | 系统无法完成当前操作 | ✅      |
| WARN  | 非预期但系统还能跑  | ✅      |
| INFO  | 关键业务事件     | ✅      |
| DEBUG | 调试细节       | ❌      |
| TRACE | 极细粒度       | ❌      |

---

## 推荐判断口诀

* **ERROR**：需要人介入（PagerDuty 会响）
* **WARN**：不处理可能变成 ERROR
* **INFO**：业务关键路径发生了什么
* **DEBUG**：开发才关心
* **TRACE**：排查极端问题才开

---

## 示例（好 vs 坏）

❌ 错误：

```java
log.error("User not found"); // 实际是正常分支
```

✅ 正确：

```java
log.warn("User not found, userId={}", userId);
```

---

# 四、日志内容与格式规范（核心）

## 1️⃣ 永远使用结构化日志（Key-Value）

❌ 不推荐（不可搜索）：

```text
create order failed for user 123 at time 2024-01-01
```

✅ 推荐：

```text
Create order failed, userId=123, orderId=456, error=INSUFFICIENT_BALANCE
```

---

## 2️⃣ 日志必备字段（强烈建议）

| 字段                  | 说明                 |
| ------------------- | ------------------ |
| timestamp           | 自动                 |
| level               | INFO/WARN/ERROR    |
| service             | 服务名                |
| env                 | prod / staging     |
| traceId / requestId | **必需**             |
| event               | 发生了什么              |
| key fields          | orderId / userId 等 |

Spring Boot 示例（Logback + MDC）：

```java
MDC.put("traceId", traceId);
log.info("Create order success, orderId={}, amount={}", orderId, amount);
```

输出：

```json
{
  "level": "INFO",
  "service": "order-service",
  "traceId": "abc123",
  "event": "Create order success",
  "orderId": 456,
  "amount": 99.0
}
```

---

## 3️⃣ 错误日志规范（必须带上下文）

❌ 错误：

```java
log.error("error", e);
```

✅ 正确：

```java
log.error(
  "Create order failed, userId={}, orderId={}",
  userId, orderId, e
);
```

---

# 五、不同层该打什么日志（分层规范）

## Controller / API 层

* 请求入口 / 出口
* 参数校验失败
* 不打业务细节

```text
INFO Incoming request POST /api/orders traceId=xxx
```

---

## Service 层（核心）

* **业务状态变化**
* 关键决策点

```text
INFO Order paid, orderId=123, payType=CREDIT
```

---

## Repository / DAO 层

* 一般不打 INFO
* 慢 SQL / 异常才打

```text
WARN Slow query detected, sql=..., costMs=1200
```

---

## 定时任务 / MQ

```text
INFO Job started, job=ExpireOrderJob
INFO Job finished, job=ExpireOrderJob, success=98, failed=2
```

---

# 六、日志安全规范（非常重要）

🚨 **严禁打印以下内容：**

* 密码 / token / secret
* 完整身份证 / 银行卡
* Cookie / Authorization Header
* 私钥 / access key

❌ 错误：

```java
log.info("login request, password={}", password);
```

✅ 正确：

```java
log.info("login request, username={}", username);
```

---

# 七、常见反模式（踩一个就出事）

❌ 用日志代替异常
❌ 没 traceId，跨服务无法定位
❌ ERROR 打太多（告警疲劳）
❌ 日志和代码行为不一致
❌ 本地 DEBUG 忘了关

---

# 八、推荐一套「默认可落地」日志规范（总结版）

### ✅ 默认日志格式

* JSON（生产）
* key=value（本地也可）

### ✅ 默认级别

* 生产：INFO
* 开发：DEBUG

### ✅ 必须包含

* traceId / requestId
* 业务主键（orderId / userId）
* 明确 event 描述

### ✅ 打日志的黄金原则

> **“一次状态变化，一条 INFO；一次异常，一条 ERROR；一次非预期，一条 WARN”**

---