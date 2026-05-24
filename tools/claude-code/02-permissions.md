# 02 · 权限系统

## 2.1 五级权限模式

| 模式 | 行为 | 适用场景 |
|------|------|----------|
| `default` | 可读文件；写/编辑/命令需审批 | 日常开发 |
| `acceptEdits` | 自动批准文件编辑；命令仍需审批 | 信任度较高 |
| `plan` | 先读代码出方案，待批准后执行 | 大型重构 |
| `auto` | 风险分类器自动判断；仅高风险需审批 | 高频迭代 |
| `bypassPermissions` | 完全自主，不需确认 ⚠️ | 隔离沙箱 |

**切换方式:**
- 交互界面: `Shift+Tab` 循环切换
- CLI 启动: `--dangerously-skip-permissions`

## 2.2 权限白名单/黑名单

在 `settings.json` 的 `permissions` 字段配置:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm test *)",         // 自动允许测试
      "Bash(npm run build)",      // 自动允许构建
      "Bash(git status)",         // 自动允许 git status
      "Bash(git diff)",           // 自动允许 git diff
      "Bash(git log *)",          // 自动允许 git log
      "Bash(git branch *)",       // 自动允许 git branch
      "Read",                     // 自动允许读取任何文件
      "WebSearch",                // 自动允许网络搜索
      "WebFetch"                  // 自动允许网页抓取
    ],
    "deny": [
      "Bash(rm -rf *)",           // 禁止递归删除
      "Bash(curl *)",             // 禁止 curl
      "Bash(wget *)",             // 禁止 wget
      "Bash(> *)",                // 禁止输出重定向
      "Bash(sudo *)",             // 禁止 sudo
      "Bash(git push --force *)", // 禁止强制推送
      "Bash(git reset --hard *)"  // 禁止硬重置
    ]
  }
}
```

## 2.3 匹配语法

| 语法 | 说明 | 示例 |
|------|------|------|
| 精确匹配 | 完全相同的命令 | `"Bash(npm test)"` |
| 通配符 `*` | 匹配任意内容 | `"Bash(npm *)"` — 所有 npm 命令 |
| 工具级 | 匹配整个工具类别 | `"Read"`, `"Write"`, `"Edit"` |
| 组合前缀 | 匹配命令前缀 | `"Bash(git *)"` — 所有 git 命令 |

**优先级:** deny > allow

## 2.4 管理级安全限制

以下键仅托管策略 (managed) 可用:

| 键 | 说明 |
|----|------|
| `disableAutoMode` | 阻止自动模式 (`"disable"`) |
| `disableSkillShellExecution` | 禁用 skill 中 `!` shell 执行 |
| `disableRemoteControl` | 禁用远程控制 |
| `allowManagedHooksOnly` | 仅加载托管 hooks |
| `allowManagedMcpServersOnly` | 仅允许管理员定义的 MCP 服务器 |
| `allowManagedPermissionRulesOnly` | 仅使用托管权限规则 |
| `forceLoginMethod` | 限制登录方式 (`"claudeai"` / `"console"`) |
| `forceRemoteSettingsRefresh` | 启动时阻止直到远程设置刷新 |

## 2.5 减少权限提示

使用 `/fewer-permission-prompts` 自动扫描使用历史并建议添加白名单规则。

手动策略: 把高频安全命令加入 `allow` 列表，把危险操作加入 `deny` 列表。
