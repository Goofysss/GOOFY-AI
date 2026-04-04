# 自动备份管理

管理「每隔3步自动存档」功能的开关。

## 使用方式

- `/auto-backup` 或 `/auto-backup status` → 查看当前是否开启
- `/auto-backup on` → 开启（每3次操作自动存档）
- `/auto-backup off` → 关闭

---

## 执行步骤

### 查看状态

读取 `~/.claude/settings.json`，检查是否存在 `hooks.UserPromptSubmit` 且包含 `claude_backup_counter`：
- 存在 → 告诉用户「✓ 自动备份已开启，每3次操作存一个档」
- 不存在 → 告诉用户「✗ 自动备份未开启，输入 /auto-backup on 开启」

同时显示当前计数：`cat /tmp/claude_backup_counter 2>/dev/null || echo 0`

---

### 开启（on）

在 `~/.claude/settings.json` 的 `hooks.UserPromptSubmit` 中添加以下 hook（注意保留已有内容）：

```json
{
  "hooks": [
    {
      "type": "command",
      "command": "count=$(cat /tmp/claude_backup_counter 2>/dev/null || echo '0'); count=$((count + 1)); if [ \"$count\" -ge 3 ]; then echo '0' > /tmp/claude_backup_counter; echo '{\"hookSpecificOutput\":{\"hookEventName\":\"UserPromptSubmit\",\"additionalContext\":\"【自动备份提醒】用户已累计3次操作，请在本次回复结束前主动执行 /backup 存档，备注用当前时间自动生成。\"}}'; else echo \"$count\" > /tmp/claude_backup_counter; fi",
      "statusMessage": "检查备份计数..."
    }
  ]
}
```

完成后告诉用户：「✓ 已开启，每3次操作自动存档。重启 Claude Code 后生效。」

---

### 关闭（off）

从 `~/.claude/settings.json` 中删除包含 `claude_backup_counter` 的 UserPromptSubmit hook。
同时重置计数器：`echo 0 > /tmp/claude_backup_counter`

完成后告诉用户：「✓ 已关闭自动备份。」

---

## 注意事项
- 修改 settings.json 后需要重启 Claude Code 才生效
- 计数器存在 /tmp/claude_backup_counter，重启电脑后会重置（正常现象）
- 用中文大白话说明操作结果
