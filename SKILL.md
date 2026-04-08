---
name: coding-guide-gen
description: 为项目生成 AI 编程指南 skill。当用户需要：为微服务创建 coding skill、从代码归纳规范、生成开发指南时使用。支持 Java、Python、Go、TypeScript。
---

# Coding Guide Generator

为项目生成 AI 编程指南 skill。

**核心理念**：分析代码/文档 → 归纳规范 → 生成可执行的编程指南

---

## 工作流程

```
1. 确认项目信息（路径、语言、示例文件、文档）
2. 分析项目结构（目录、分层、入口类型）
3. 归纳规范（命名、注解、方法模式）
4. 生成 skill 文档
```

---

## 第一步：确认项目信息

向用户确认：

| 信息 | 说明 | 示例 |
|------|------|------|
| 项目路径 | 代码库根目录 | `/path/to/project` |
| 服务名称 | 用于命名 skill | `dichat-msg` → `dichat-msg-coding` |
| 示例文件 | Controller、Listener 等关键类 | 提供路径，逗号分隔 |
| 编程语言 | Java、 Python等 | Python |

---

## 第二步：分析项目结构

### 2.1 检测语言

若用户未提供语言则根据检查项目根目录特征文件区分：

| 文件 | 语言 |
|-----|------|
| `pom.xml` / `build.gradle` | Java |
| `go.mod` | Go |
| `pyproject.toml` / `setup.py` | Python |
| `tsconfig.json` | TypeScript |
| `package.json` | JavaScript |

### 2.2 扫描目录结构

使用 Glob 扫描源码目录，识别分层：

```bash
# Java
**/src/main/java/**/*.java

# Python
**/*.py

# Go
**/*.go

# TypeScript
**/src/**/*.ts
```

### 2.3 识别分层

从目录名推断分层职责（常见模式，可根据项目实际情况调整）：

| 目录关键词 | 分层 |
|-----------|------|
| controller, handler, api, resource | 接口层 |
| application, app, usecase | 应用层 |
| service, facade | 服务层 |
| domain, entity, model | 领域层 |
| infrastructure, infra, gateway | 基础设施层 |
| repository, repo, dao, mapper | 数据层 |
| dto, vo, request, response | 传输对象 |
| util, utils, helper, common | 工具层 |
| config, configuration | 配置层 |
| mq, consumer, listener, event | 消息层 |

若项目使用非标准命名（如 presentation、logic、persistence），根据目录职责灵活判断。

### 2.4 识别入口类型

| 入口类型 | 识别方式 |
|---------|---------|
| API | 类名包含 Controller |
| MQ | 类名包含 Consumer |
| CRON | 方法上有 @Scheduled 或类名包含 ScheduledTask |

---

## 第三步：归纳规范

### 3.1 命名规范

读取各层代码，分析类名后缀模式：

**Java**：
```bash
grep -r "public class\|public interface" --include="*.java"
```

**Python**：
```bash
grep -r "^class \|^class " --include="*.py"
```

**Go**：
```bash
grep -r "^type.*struct\|^type.*interface" --include="*.go"
```

**TypeScript**：
```bash
grep -r "^class \|^interface " --include="*.ts"
```

归纳：
- 接口入参后缀（如 Form、RequestBody、Cmd）
- 接口出参后缀（如 Result、VO、DTO）
- 服务类后缀（如 Service、DomainService）
- 数据层后缀（如 Repository、Repo、Dao）

### 3.2 注解规范

读取代码，提取**在代码中使用的注解/装饰器**：

**Java**：
```bash
grep -r "@[A-Z]" --include="*.java" | grep -v "/\*\*"
```
过滤：Javadoc tag（@param、@return）、Java 内置（@Override、@Deprecated）、测试注解（@Test）

**Python**：
```bash
grep -r "@" --include="*.py" | grep -v "#"
```
过滤：内置装饰器（@property、@staticmethod）、测试装饰器（@pytest.mark）

**Go**：
```go
// Go 无注解，分析结构体 tag
// 例如：`json:"name" db:"name"`
```

**TypeScript**：
```bash
grep -r "@" --include="*.ts"
```
过滤：内置装饰器（@readonly、@sealed）

为每个注解标注含义和用途。

### 3.3 方法模式

读取示例代码，分析典型方法结构：
- 方法命名模式（如 handleXxx、queryXxx、saveXxx）
- 参数模式（如 Form 入参、DTO 出参）
- 异常处理模式

### 3.4 工具类

扫描工具类目录，记录可复用的工具：

```bash
# 搜索工具类
find . -path "*/util/*" -o -path "*/utils/*" -o -name "*Util.java" -o -name "*Utils.java"
```

---

## 第四步：生成 Skill 文档

### 输出结构

```
{service-name}-coding/
├── SKILL.md                      # 主文档
└── references/
    ├── principles.md             # 核心原则（禁止重复造轮子）
    ├── architecture.md           # 架构分层详解
    ├── naming.md                 # 命名规范
    ├── components.md             # 通用组件手册
    ├── api-guide.md              # API 开发指南
    ├── mq-guide.md               # MQ 消费者指南
    └── example.md                # 示例代码
```

### SKILL.md 结构

```markdown
# {服务名} AI 编程指南

## ⚠️ 核心原则

**禁止重复造轮子** — 开发前必读 `references/principles.md`

## 快速判断：我需要开发哪种类型的入口？

| 你要开发的是 | 阅读这份参考 |
|------------|-------------|
| API 接口 | `references/api-guide.md` |
| MQ 消费者 | `references/mq-guide.md` |
| 不确定，看架构 | `references/architecture.md` |

## 架构分层

[从分析结果生成]

## 命名规范速查

[从命名分析结果生成]

## 通用组件

[从工具类分析结果生成]

## 开发流程

1. 搜索现有实现（见 principles.md）
2. 确定入口类型
3. 阅读对应参考文档
4. 编写代码
5. 提交代码
```

### principles.md 结构

```markdown
# 核心原则

## 一、禁止重复造轮子

### 1. 工具类

[从工具类分析结果生成]

**禁止**：自行实现已有功能

### 2. 组件

[从注解分析结果生成]

## 二、开发前搜索现有实现

[生成搜索命令]

## 三、调用已有类（禁止编造方法）

[说明如何确认方法存在]
```

---

## 生成示例

用户输入：
```
项目路径: /path/to/dichat-msg
服务名称: dichat-msg
示例文件: MessageControllerV2.java, VchannelEventConsumer.java
```

Agent 执行：
1. 检测语言 → Java（存在 pom.xml）
2. 扫描目录 → 识别出接口层、服务层、领域层、消息层等
3. 归纳命名 → Controller 后缀、Service 后缀、DTO 后缀
4. 归纳注解 → @Autowired、@Component、@RequestMapping 等
5. 生成文档 → SKILL.md + references/*.md

---

## 注意事项

1. **不要硬编码** — 所有规范从代码中归纳
2. **不要编造** — 不确定的内容标注"请根据项目上下文理解"
3. **提供搜索命令** — 让 AI 能够验证和查找
4. **示例代码要完整** — 包含 import、注解、类定义
5. **生成后验证** — 检查规范是否与代码一致

---

## ⚠️ 严格约束

### 禁止编造规范

**所有规范必须从代码中实际存在的内容归纳，禁止"自认为合理"的编造。**

❌ 错误做法：
```markdown
# 编造不存在的命名规范
| Application 入参（读） | `XxxQry` | `QueryMessageQry` |  # 代码中不存在！
```

✅ 正确做法：
```bash
# 先搜索确认存在
grep -r "Qry" --include="*.java"
# 结果为空 → 不生成此规范
```

### 归纳规范前必须验证

每个归纳的规范，必须先搜索确认代码中存在：

| 规范类型 | 验证方式 |
|---------|---------|
| 命名后缀 | `grep -r "XxxSuffix" --include="*.java"` |
| 注解使用 | `grep -r "@Annotation" --include="*.java"` |
| 方法模式 | `grep -r "methodName" --include="*.java"` |
| 工具类 | `find . -name "XxxUtils.java"` |

**验证结果为空 → 不生成该规范**

### 不确定时的处理

如果搜索结果不确定：
- 标注"请根据项目上下文理解"
- 或直接省略该规范
- **绝对不要编造**