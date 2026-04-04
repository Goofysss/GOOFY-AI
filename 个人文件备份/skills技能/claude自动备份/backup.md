# 存档 · 一键保存 + 更新仪表盘

本地存档、更新仪表盘版本历史、可选同步 GitHub，一步完成。

## 使用方式

- `/backup` → 本地存档 + 更新仪表盘
- `/backup push` → 本地存档 + 更新仪表盘 + 同步 GitHub

---

## 执行步骤

### 第一步：确认 git 状态

运行 `git status`：
- 提示 "not a git repository" → 先运行 `git init`，告诉用户「已初始化存档系统」
- 正常输出 → 继续

### 第二步：检查是否有改动

运行 `git status --short`：
- 没有任何输出 → 告诉用户「当前没有新改动，无需存档」，跳到第五步（只更新仪表盘日期）
- 有输出 → 继续

### 第三步：自动生成存档备注

运行以下命令了解改动内容：
```bash
git diff --stat HEAD
git status --short
```

根据改动自动生成备注，要求：
- **15字以内**，中文，简洁高效
- 优先描述「做了什么」而不是「改了哪个文件」
- 例子：新增登录页、修复导航样式、整理文件结构、更新仪表盘配色
- 多个改动时取最主要的那个
- 不用加日期时间（git 自动记录）

### 第四步：本地存档

```bash
git add -A
git commit -m "备注内容"
```

告诉用户：「✓ 存档成功」

### 第五步：更新仪表盘版本历史

运行以下命令获取 git log：

```bash
git log --format='%H|%s|%ad|%an' --date=format:'%Y-%m-%d %H:%M'
```

编辑仪表盘文件：
`~/Library/Mobile Documents/com~apple~CloudDocs/claude-dashboard.html`

找到 `<!-- 版本历史 -->` 区块，用最新 git log 重新生成 `.history-list` 内所有 `.history-item`。

**每条记录格式：**
```html
<div class="history-item">
  <div class="history-dot [latest]"></div>
  <div class="history-line"></div>
  <div class="history-content">
    <div class="history-msg">存档备注</div>
    <div class="history-meta">
      <span class="history-date">日期时间</span>
      <span class="history-hash">短hash前7位</span>
      <span class="history-project">当前文件夹名</span>
    </div>
  </div>
</div>
```

规则：
- 最新一条加 `latest` class（橙点），其余为普通灰点
- 同时将仪表盘底部「最后更新」日期更新为今天

告诉用户：「✓ 仪表盘已更新」

### 第六步：如果是 `/backup push`，同步到 GitHub

检查 `git remote -v`：

**已有 remote：**
```bash
git push
```

**没有 remote（第一次）：**
```bash
gh repo create <文件夹名> --private --source=. --remote=origin --push
```

告诉用户 GitHub 仓库地址，「✓ 已同步到云端」

---

## 最终回复格式

完成后用一段简洁的总结告诉用户：
```
✓ 存档完成
  备注：xxx
  时间：xxx
  仪表盘：已更新
  GitHub：已同步 / 仅本地
```

---

## 注意事项
- 仓库默认私密（private）
- 仪表盘路径固定在 iCloud：~/Library/Mobile Documents/com~apple~CloudDocs/claude-dashboard.html
- 全程用中文大白话说明
