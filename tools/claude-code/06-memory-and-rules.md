# 06 · 记忆与规则系统

## 6.1 记忆层级 (优先级从高到低)

| 优先级 | 位置 | 作用域 | Git |
|--------|------|--------|-----|
| 1 (最高) | 托管策略 CLAUDE.md (系统目录) | 组织 | ❌ |
| 1.5 | `managed-settings.d/*.md` | 组织 | ❌ |
| 2 | `./CLAUDE.md` 或 `./.claude/CLAUDE.md` | 项目 | ✅ |
| 3 | `./.claude/rules/*.md` | 项目 (路径限定) | ✅ |
| 4 | `~/.claude/CLAUDE.md` | 用户全局 | ❌ |
| 5 | `~/.claude/rules/*.md` | 用户全局 | ❌ |
| 6 | `./CLAUDE.local.md` | 项目个人 | ❌ (gitignore) |
| 7 (最低) | Auto Memory | 项目个人 | ❌ |

## 6.2 CLAUDE.md 指令文件

### 基础模板

```markdown
# 项目名称

## 常用命令
- `npm run dev` — 启动开发服务器
- `npm test` — 运行测试
- `npm run build` — 构建
- `npm run lint` — 代码检查

## 架构说明
- `src/` — 源代码
- `src/components/` — React 组件
- `src/api/` — API 路由
- `tests/` — 测试

## 编码规范
- TypeScript 严格模式
- 函数组件，不用 class
- 使用 Zod 进行输入验证
- 测试文件放在 `__tests__` 目录

## 注意事项
- 不要直接修改 `generated/` 目录
- API 调用必须经过 `src/api/client.ts`
```

### @import 语法
在 CLAUDE.md 中导入外部文件（最多 5 层级深）:
```markdown
@docs/architecture.md
@docs/api-spec.md
```

### 排除特定 CLAUDE.md (monorepo)
```json
{
  "claudeMdExcludes": [
    "**/vendor/**/CLAUDE.md",
    "**/node_modules/**/CLAUDE.md",
    "packages/legacy-app/CLAUDE.md"
  ]
}
```

## 6.3 Auto Memory 自动记忆

### 存储位置
```
~/.claude/projects/<编码后的项目路径>/memory/
```
路径编码: `/` → `-` (例: `/Users/carl/my-project` → `-Users-carl-my-project`)

### MEMORY.md (索引文件)
- 始终加载到系统提示中
- 限制: **前 200 行** 或 **25KB** (先达到为准)
- 应包含简洁指针/链接，而非详细内容

```markdown
# 记忆索引

- [用户偏好](user_preferences.md) — 代码风格、沟通偏好
- [调试经验](debugging.md) — 常见问题和解决方案
- [API 规范](api_conventions.md) — API 设计约定
```

### 四种记忆类型

#### user — 用户信息
记录角色、目标、知识背景。帮助 Claude 为不同用户量身定制回复。

```markdown
---
name: user-role
description: 用户角色和偏好
metadata:
  type: user
---

资深后端工程师，熟悉 Go 和 Rust，对前端 React 较陌生。
偏好简洁直接的回答。
```

#### feedback — 行为反馈
记录对 Claude 行为的纠正或认可，避免重复犯错。

```markdown
---
name: feedback-testing
description: 测试相关反馈
metadata:
  type: feedback
---

规则: 集成测试必须连接真实数据库，不要 mock。
原因: 之前 mock 测试通过但生产迁移失败。
适用场景: 任何涉及数据库操作的测试。
```

#### project — 项目上下文
记录目标、决策、约束、时间线等非代码信息。

```markdown
---
name: auth-rewrite-compliance
description: 认证中间件重写项目背景
metadata:
  type: project
---

认证中间件重写由法务/合规驱动，不是技术债。
优先级: 合规 > 性能 > 美观。
截止: 2026-06-15。
```

#### reference — 外部引用
记录外部系统、资源、文档的指针。

```markdown
---
name: grafana-dashboard
description: 值班监控面板链接
metadata:
  type: reference
---

Grafana: grafana.internal/d/api-latency
修改请求处理代码时务必检查此面板。
```

### 记忆管理
- `/memory` — 在编辑器中打开记忆文件
- `autoMemoryEnabled: false` — 禁用自动记忆
- 每月审查，删除过时内容
- MEMORY.md 保持 200 行以内

## 6.4 Rules 规则系统

### 放置位置
- `.claude/rules/` (项目)
- `~/.claude/rules/` (用户)

### 规则文件格式
支持 YAML frontmatter 进行路径限定:

```markdown
---
paths: src/api/**/*.ts
---
# API 开发规则

- 所有端点必须包含输入验证
- 使用 Zod 进行 schema 验证
- 错误响应标准格式: `{ error: string, code: number }`
- 不要在路由中直接操作数据库，通过 service 层
```

### 高级特性
- **glob 限定** — `paths` 字段精确控制适用文件
- **子目录** — `rules/api/`, `rules/testing/`
- **符号链接** — 跨项目共享规则
- **优先级** — 项目 rules > 用户 rules
