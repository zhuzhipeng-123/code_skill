# GitHub Operations Skill

> 帮助用户完成 Git 代码提交和 GitHub PR 创建的技能

## 用户信息

- **GitHub 用户名**: zhuzhipeng-123
- **认证方式**: gh CLI (已配置)
- **Git 用户**: zzp <15237518575@163.com>

---

## 核心能力

### 1. 提交代码到仓库

**标准流程**:

```bash
# 1. 检查当前状态
git status

# 2. 查看变更内容
git diff

# 3. 添加变更文件
git add <files>  # 或 git add . 添加所有

# 4. 提交（必须写清楚 commit message）
git commit -m "type: description"
# type: feat(新功能) | fix(修复) | docs(文档) | refactor(重构) | test(测试) | chore(杂项)

# 5. 推送到远程
git push origin <branch-name>
```

**Commit Message 规范**:
- `feat: 添加用户登录功能`
- `fix: 修复登录页面样式问题`
- `docs: 更新 README 安装说明`
- `refactor: 重构数据获取逻辑`

---

### 2. 创建 Pull Request (PR)

**使用 gh CLI 创建 PR**:

```bash
# 1. 确保分支已推送到远程
git push -u origin <feature-branch>

# 2. 创建 PR
gh pr create --title "标题" --body "描述内容"

# 3. 或者使用交互式创建
gh pr create
```

**PR 标题格式**:
- `[功能] 添加用户登录`
- `[修复] 登录页面样式问题`
- `[优化] 重构数据获取逻辑`

**PR 描述模板**:
```markdown
## 变更内容
- 添加了 xxx 功能
- 修复了 xxx 问题

## 测试
- [ ] 已本地测试
- [ ] 已检查代码风格

## 相关 Issue
Closes #123  # 如果有关联的 issue
```

---

### 3. 常用 Git 操作

```bash
# 查看提交历史
git log --oneline -10

# 查看远程仓库
git remote -v

# 创建并切换分支
git checkout -b <new-branch>

# 切换分支
git checkout <branch-name>

# 拉取最新代码
git pull origin <branch-name>

# 撤销未暂存的修改
git checkout -- <file>

# 撤销已暂存但未提交的修改
git reset HEAD <file>
```

---

### 4. 常见问题处理

#### 问题: push 被拒绝 (rejected)

```bash
# 原因: 远程有新提交
# 解决: 先拉取再推送
git pull --rebase origin <branch-name>
git push origin <branch-name>
```

#### 问题: 合并冲突

```bash
# 1. 查看冲突文件
git status

# 2. 手动编辑冲突文件，解决冲突标记
# <<<<<<< HEAD
# 本地修改
# =======
# 远程修改
# >>>>>>> branch-name

# 3. 标记冲突已解决
git add <resolved-file>
git commit -m "fix: 解决合并冲突"
```

#### 问题: 想要撤销最后一次提交

```bash
# 保留修改
git reset --soft HEAD~1

# 丢弃修改（危险！）
git reset --hard HEAD~1
```

#### 问题: 想要修改最后一次 commit message

```bash
git commit --amend -m "新的 commit message"
# 注意: 如果已经 push 了，需要 force push
git push --force-with-lease origin <branch-name>
```

---

## gh CLI 快捷命令

```bash
# 查看 PR 列表
gh pr list

# 查看 PR 详情
gh pr view <pr-number>

# 合并 PR
gh pr merge <pr-number>

# 查看 Issue 列表
gh issue list

# 创建 Issue
gh issue create --title "标题" --body "内容"

# 在浏览器打开仓库
gh repo view --web
```

---

## 操作检查清单

### 提交代码前:
- [ ] 确认在正确的分支
- [ ] 检查 `git status` 了解变更
- [ ] 检查 `git diff` 确认变更内容
- [ ] 写清晰的 commit message

### 创建 PR 前:
- [ ] 确保代码已 push 到远程
- [ ] 写清晰的 PR 标题和描述
- [ ] 确认目标分支（通常是 main/master）

### 遇到问题时:
- [ ] 先读错误信息
- [ ] 检查网络连接
- [ ] 检查 gh auth status
- [ ] 使用 `git status` 了解当前状态

---

## 安全提示

⚠️ **永远不要**:
- 把 token 写进代码文件
- 在公开场合分享 token
- 使用 `--force` 推送到 main/master 分支

✅ **推荐做法**:
- 定期更换 token
- 使用有意义的分支名
- commit message 要清晰描述变更