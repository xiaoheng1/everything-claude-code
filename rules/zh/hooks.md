# 钩子系统

> 官方文档：https://docs.anthropic.com/en/docs/claude-code/hooks

## 钩子类型

| 类型 | 触发时机 | 用途 |
|------|----------|------|
| **PreToolUse** | 工具执行前 | 验证、拦截、参数修改 |
| **PostToolUse** | 工具执行后 | 格式化、检查结果、日志记录 |
| **Stop** | 会话结束时 | 清理、保存状态、最终验证 |
| **Notification** | Claude 发送通知时 | 自定义通知处理 |

## 配置位置

| 范围 | 位置 |
|------|------|
| 项目级 | `.claude/settings.json` |
| 用户级 | `~/.claude/settings.json` |
| 企业级 | `/etc/claude-code/settings.json` |

## 配置结构

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": ["path/to/pre-bash-hook.sh"],
        "timeout": 60000
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": ["path/to/post-write-hook.sh"]
      }
    ]
  }
}
```

| 字段 | 描述 |
|------|------|
| `matcher` | 正则表达式匹配工具名称（如 `"Bash"`, `"Write"`, `"Read"`） |
| `hooks` | 要执行的命令/脚本数组 |
| `timeout` | 可选超时时间（毫秒），默认 60000 |

## 环境变量

钩子脚本中可用：

| 变量 | 描述 | 可用阶段 |
|------|------|----------|
| `CLAUDE_TOOL_NAME` | 工具名称（如 `Write`, `Bash`, `Read`） | PreToolUse, PostToolUse |
| `CLAUDE_TOOL_INPUT` | JSON 编码的工具输入参数 | PreToolUse, PostToolUse |
| `CLAUDE_TOOL_OUTPUT` | JSON 编码的工具输出 | PostToolUse |

## 钩子执行流程

```
工具调用请求
    ↓
PreToolUse 钩子
    ├─ 匹配工具名称？
    ├─ 执行钩子脚本
    ├─ Exit 0 → 继续执行工具
    └─ Exit 非0 → 拦截工具，返回错误
    ↓
工具执行（Write, Bash, Read 等）
    ↓
PostToolUse 钩子
    ├─ 匹配工具名称？
    ├─ 执行钩子脚本
    └─ 记录/处理结果
    ↓
返回结果给 Claude
```

## 钩子示例

### 1. Write 后自动格式化 JavaScript 文件

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": ["prettier --write $CLAUDE_TOOL_INPUT"]
      }
    ]
  }
}
```

### 2. 拦截危险的 Bash 命令

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          "if [[ \"$CLAUDE_TOOL_INPUT\" =~ rm.*-rf ]]; then echo '危险命令已拦截' && exit 1; fi"
        ]
      }
    ]
  }
}
```

### 3. 会话结束清理

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": ["echo '会话结束于 $(date)' >> ~/.claude/session-log.txt"]
      }
    ]
  }
}
```

## 自动接受权限

谨慎使用：
- 为可信、定义明确的计划启用
- 探索性工作时禁用
- 永远不要使用 `dangerously-skip-permissions` 标志
- 改为在 `settings.json` 中配置 `allowedTools`

## TodoWrite 最佳实践

使用 TodoWrite 工具：
- 跟踪多步骤任务的进度
- 验证对指令的理解
- 启用实时引导
- 显示细粒度的实现步骤

待办列表揭示：
- 顺序错误的步骤
- 缺失的项目
- 多余的不必要项目
- 错误的粒度
- 误解的需求