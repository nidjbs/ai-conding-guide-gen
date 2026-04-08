# 分析能力详解

coding-guide-gen 会自动识别以下内容：

---

## 架构模式识别

通过扫描目录结构和包路径，自动推断架构类型：

| 架构模式 | 识别特征 | 依赖规则 |
|---------|---------|---------|
| DDD 四层 | `interfaces/` `application/` `domain/` `infra/` | Interface → Application → Domain → Infra |
| 三层架构 | `controller/` `service/` `dao/` 或 `mapper/` | Controller → Service → DAO |
| 六边形架构 | `ports/` `adapters/` | 外层依赖内层，端口与适配器分离 |
| Clean Architecture | `entities/` `usecases/` `interface/` `infrastructure/` | 由内向外依赖 |

### 识别逻辑

1. 扫描前 4 层目录结构
2. 匹配预定义的架构模式
3. 匹配度 ≥ 75% 则确认为该架构
4. 无法匹配则标记为"自定义架构"

---

## 命名规范提取

从现有类名中统计后缀模式：

| 类别 | 识别规则 | 常见后缀 |
|------|---------|---------|
| 接口入参 | 后缀模式统计 | `RequestBody`, `Body`, `Params`, `Request`, `Input` |
| 接口出参 | 后缀模式统计 | `VO`, `Response`, `Output`, `Result` |
| 用例入参 | 后缀模式统计 | `Cmd`, `Command`, `Query`, `Qry` |
| 用例出参 | 后缀模式统计 | `DTO`, `Dto`, `Result` |
| 服务类 | 后缀模式统计 | `Service`, `Manager`, `DomainService` |
| 仓储 | 接口/实现配对 | `Repo` / `RepoImpl`, `Repository` / `RepositoryImpl` |
| 错误码 | 枚举/常量类 | `Errors`, `ErrorCode`, `ErrorCodes` |

### 提取逻辑

1. 扫描所有 `.java` 文件
2. 按后缀分类统计出现次数
3. 取 Top 3 作为推荐命名规范

---

## 工具类识别

扫描以下路径提取工具类：

### 标准路径

```
common/util/
common/utils/
infra/util/
infra/utils/
infrastructure/util/
infrastructure/utils/
util/
utils/
lib/
libs/
```

### 类名后缀

- `*Util` — 如 `JsonUtil`
- `*Utils` — 如 `DateUtils`
- `*Helper` — 如 `StringHelper`
- `*Helpers` — 如 `FileHelpers`

### 输出内容

每个工具类记录：
- 类名
- 相对路径
- 所在包名

---

## 通用组件识别

通过扫描注解使用频率识别：

### 支持的注解

| 注解 | 类型 | 用途 |
|------|------|------|
| `@DLock` | 分布式锁 | 防止并发 |
| `@Lock` | 分布式锁 | 分布式锁 |
| `@DistributedLock` | 分布式锁 | 分布式锁 |
| `@Cache` | 缓存 | Redis 缓存 |
| `@Cacheable` | 缓存 | Spring 缓存 |
| `@Async` | 异步执行 | 异步执行 |
| `@TraceableAsync` | 异步执行 | 带链路追踪的异步执行 |
| `@Idempotent` | 幂等 | 幂等控制 |
| `@IdempotentLock` | 幂等 | 幂等锁 |
| `@Transactional` | 事务 | 事务管理 |

### 识别逻辑

1. 扫描所有 `.java` 文件内容
2. 统计各注解出现次数
3. 取 Top 10 作为项目使用的组件

---

## 入口类型识别

扫描类名识别入口类型：

| 类型 | 类名模式 | 说明 |
|------|---------|------|
| API | `*Controller`, `*Resource` | REST API 接口 |
| MQ | `*Consumer`, `*Listener`, `*MQ*` | 消息队列消费者 |
| CRON | `*Scheduled*`, `*Job`, `*Task` | 定时任务 |
| GRPC | `*Grpc*` | gRPC 服务 |

---

## 真实示例提取

当提供 `--examples` 参数时：

### 提取流程

1. **解析文件** — 识别类的类型（Controller、Service、Repo 等）
2. **读取代码** — 保留完整类结构
3. **脱敏处理** — 替换敏感信息
4. **分类存储** — 按入口类型组织

### 脱敏规则

| 类型 | 处理方式 |
|------|---------|
| IP 地址 | `x.x.x.x` |
| 密码字段 | `***` |
| 密钥/Token | `***` |
| 其他敏感配置 | 保留占位符 |

### 保留内容

- 类结构
- 方法签名
- 注解
- 关键业务逻辑

### 简化内容

- 过长的实现细节（超过 3000 字符截断）
- 测试代码
- 注释中的敏感信息

---

## 输出文档内容

| 文档 | 内容来源 | 说明 |
|------|---------|------|
| `principles.md` | 工具类 + 组件扫描 | 禁止造轮子的原则 |
| `architecture.md` | 架构识别结果 | 分层结构 + 依赖规则 |
| `naming.md` | 类名后缀统计 | 命名规范表 |
| `components.md` | 工具类 + 组件列表 | 使用示例 |
| `api-guide.md` | Controller 例子 | API 开发规范 |
| `mq-guide.md` | Consumer 例子 | MQ 消费者规范 |
| `cron-guide.md` | ScheduledTask 例子 | 定时任务规范 |
| `example.md` | 所有例子 | 完整代码示例 |

---

## 分析准确性建议

### 提高准确性的方法

1. **代码库完整** — 避免只扫描部分代码
2. **提供例子文件** — 关键类作为例子能提高示例质量
3. **提供文档** — 现有架构文档会被整合
4. **标准目录结构** — 符合常见架构模式更易识别

### 生成后检查清单

- [ ] 架构模式是否正确识别
- [ ] 命名规范是否符合实际
- [ ] 工具类是否遗漏重要类
- [ ] 组件列表是否完整
- [ ] 示例代码是否脱敏