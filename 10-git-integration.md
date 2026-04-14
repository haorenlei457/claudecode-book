# 10 - Git集成

## 📋 模块介绍

Git集成让 Claude Code 成为完整的开发工作流工具，无需离开终端就能完成从编码到部署的全流程。本章将深入讲解Git操作、工作流管理和自动化。

---

## 🟢 入门级：Git操作基础

### 为什么需要Git集成？

```markdown
传统开发流程：
1. 写代码 → 2. 手动git add → 3. 手动git commit → 4. 手动git push

使用 Claude Code：
1. 写代码 → 2. "帮我提交" → 完成！

优势：
✅ 自动生成规范的提交信息
✅ 智能分析变更内容
✅ 自动处理分支管理
✅ 集成PR审查流程
```

### 基本Git命令

```bash
# 查看状态
claude> 查看git状态
claude> 当前有哪些变更？

# 查看历史
claude> 查看最近的提交历史
claude> 显示最近5个commit

# 创建分支
claude> 创建新分支 feature/user-auth
claude> 从main切出新分支 fix-login-bug

# 切换分支
claude> 切换到develop分支
claude> 切换到上一个分支

# 合并分支
claude> 合并feature分支到main
claude> 处理合并冲突

# 创建提交
claude> /commit 修复登录bug
claude> 提交所有变更，消息是"添加用户认证功能"

# 推送代码
claude> 推送到远程仓库
claude> 推送当前分支并创建PR
```

### 查看Git状态

```bash
$ claude
claude> 查看git状态

📊 Git 状态报告

当前分支：feature/user-auth
上游分支：origin/feature/user-auth

变更文件：
  修改： 3个文件
    - src/auth/login.ts
    - src/auth/types.ts
    - tests/auth.test.ts

  新增： 2个文件
    - src/auth/validator.ts
    - docs/auth.md

  删除： 1个文件
    - src/auth/old-auth.ts

未跟踪文件：
  - temp/debug.log

建议操作：
  1. 删除临时文件：git rm temp/debug.log
  2. 创建提交：/commit 实现用户认证功能
```

---

## 🟡 中级：Git工作流与自动化

### 智能提交流程

Claude Code 的提交不是简单的 `git commit`，而是智能流程：

```
智能提交流程：
┌──────────────┐
│ 分析变更      │  ← 查看git diff，理解修改内容
└──────┬───────┘
       ↓
┌──────────────┐
│ 检查规范      │  ← 运行lint、测试，确保代码质量
└──────┬───────┘
       ↓
┌──────────────┐
│ 生成消息      │  ← 根据变更生成规范的提交信息
└──────┬───────┘
       ↓
┌──────────────┐
│ 执行提交      │  ← git add + git commit
└──────┬───────┘
       ↓
┌──────────────┐
│ 后续操作      │  ← 可选：push、创建PR
└──────────────┘
```

### 提交信息规范

Claude Code 遵循 Conventional Commits 规范：

```
<type>(<scope>): <subject>

<body>

<footer>
```

**示例**：

```bash
# 功能提交
claude> /commit 添加用户登录功能

生成的提交：
feat(auth): 添加用户登录功能

- 实现了JWT认证
- 添加了登录表单验证
- 集成了第三方OAuth

Closes #123
```

```bash
# 修复提交
claude> /commit 修复登录失败的问题

生成的提交：
fix(auth): 修复登录验证逻辑错误

修复了token验证时的时区问题
导致某些时区的用户无法登录

Fixes #456
```

**类型说明**：

| 类型 | 含义 | 示例 |
|------|------|------|
| feat | 新功能 | feat: 添加用户注册 |
| fix | 修复bug | fix: 修复空指针异常 |
| docs | 文档更新 | docs: 更新API文档 |
| style | 代码格式 | style: 格式化代码 |
| refactor | 重构 | refactor: 优化数据库查询 |
| test | 测试 | test: 添加单元测试 |
| chore | 构建/工具 | chore: 更新依赖 |

### PR管理

#### 1. 创建PR

```bash
claude> 创建PR

Claude会：
1. 推送当前分支
2. 分析变更内容
3. 生成PR标题和描述
4. 创建PR

生成的PR：
标题：[WIP] 实现用户认证功能

描述：
## 功能概述
实现了完整的用户认证系统

## 变更内容
- 添加JWT认证中间件
- 实现登录/注册API
- 添加前端登录页面
- 编写单元测试

## 测试
- [x] 单元测试通过
- [x] 集成测试通过
- [ ] 手动测试

## 截图
[登录页面截图]

## 相关Issue
Closes #123
```

#### 2. 审查PR

```bash
claude> 审查PR #456

Claude会自动：
1. 获取PR详情
2. 查看代码变更
3. 运行测试
4. 检查代码质量
5. 生成审查报告

审查报告：
## PR 审查报告 #456

### 概览
- 作者：@developer
- 分支：feature/auth → main
- 变更：+523/-89 行

### 代码质量 ✅
- 测试覆盖率：87% (要求>80%)
- ESLint：无错误
- TypeScript：无类型错误

### 发现的问题 ⚠️

#### 🔴 严重 (1)
1. **潜在的安全漏洞**
   - 位置：src/auth/login.ts:45
   - 问题：明文存储密码
   - 建议：使用bcrypt哈希

#### 🟡 一般 (2)
1. **缺少错误处理**
   - 位置：src/auth/token.ts:23
   - 建议：添加try-catch

2. **硬编码值**
   - 位置：src/config/auth.ts:12
   - 建议：移到环境变量

### 建议 ✅
1. 添加密码强度检查
2. 实现速率限制
3. 添加审计日志

### 结论
建议修复严重问题后合并
```

#### 3. 合并PR

```bash
claude> 合并PR #456

确认信息：
- PR #456: 实现用户认证功能
- 所有检查通过 ✅
- 已审查通过 ✅

合并方式：
1. Create merge commit (推荐)
2. Squash and merge
3. Rebase and merge

选择：1

执行合并...
✅ 合并成功！
```

### 高级Git工作流

#### 1. 功能分支工作流

```bash
# 开始新功能
claude> 开始功能：用户认证

Claude会自动：
1. 从main创建分支：feature/user-auth
2. 设置上游跟踪
3. 创建初始提交

# 开发过程中
claude> 提交进度

# 功能完成
claude> 完成功能

Claude会自动：
1. 最终提交
2. 推送分支
3. 创建PR
4. 请求审查
```

#### 2. 修复分支工作流

```bash
# 紧急修复
claude> 创建热修复：修复登录bug

Claude会自动：
1. 从main创建hotfix分支
2. 修复问题
3. 提交并推送
4. 创建PR到main和develop
```

#### 3. 发布工作流

```bash
# 准备发布
claude> 准备发布 v1.2.0

Claude会自动：
1. 创建release分支
2. 更新版本号
3. 更新CHANGELOG
4. 打标签
5. 创建发布PR
```

---

## 🔴 专家级：Git集成深度剖析

### Git操作封装

```typescript
class GitOperations {
  private cwd: string;

  constructor(cwd: string) {
    this.cwd = cwd;
  }

  async status(): Promise<GitStatus> {
    const result = await this.exec('git status --porcelain=v1');
    return this.parseStatus(result);
  }

  async add(files?: string[]): Promise<void> {
    if (files && files.length > 0) {
      await this.exec(`git add ${files.join(' ')}`);
    } else {
      await this.exec('git add -A');
    }
  }

  async commit(message: string, options?: CommitOptions): Promise<void> {
    const args = ['git', 'commit', '-m', message];

    if (options?.amend) {
      args.push('--amend');
    }

    if (options?.noVerify) {
      args.push('--no-verify');
    }

    await this.exec(args.join(' '));
  }

  async push(remote?: string, branch?: string): Promise<void> {
    const args = ['git', 'push'];

    if (remote) {
      args.push(remote);
      if (branch) {
        args.push(branch);
      }
    }

    await this.exec(args.join(' '));
  }

  async createBranch(name: string, from?: string): Promise<void> {
    const args = ['git', 'checkout', '-b', name];

    if (from) {
      args.push(from);
    }

    await this.exec(args.join(' '));
  }

  async diff(ref?: string): Promise<FileDiff[]> {
    const cmd = ref
      ? `git diff ${ref} --stat`
      : 'git diff --cached --stat';

    const result = await this.exec(cmd);
    return this.parseDiffStat(result);
  }

  async log(options?: LogOptions): Promise<CommitInfo[]> {
    const format = '%H|%an|%ae|%ad|%s';
    const cmd = `git log --pretty=format:"${format}" -${options?.limit || 10}`;

    const result = await this.exec(cmd);
    return this.parseLog(result);
  }

  private async exec(command: string): Promise<string> {
    return new Promise((resolve, reject) => {
      exec(command, { cwd: this.cwd }, (error, stdout, stderr) => {
        if (error) {
          reject(new Error(stderr || error.message));
        } else {
          resolve(stdout.trim());
        }
      });
    });
  }

  private parseStatus(output: string): GitStatus {
    const lines = output.split('\n').filter(Boolean);
    const status: GitStatus = {
      staged: [],
      unstaged: [],
      untracked: []
    };

    for (const line of lines) {
      const staged = line[0];
      const unstaged = line[1];
      const file = line.slice(3);

      if (staged !== ' ' && staged !== '?') {
        status.staged.push({ status: staged, file });
      }

      if (unstaged !== ' ') {
        status.unstaged.push({ status: unstaged, file });
      }

      if (staged === '?') {
        status.untracked.push(file);
      }
    }

    return status;
  }
}
```

### 智能提交消息生成

```typescript
class CommitMessageGenerator {
  async generate(diff: FileDiff[]): Promise<string> {
    // 1. 分析变更类型
    const changeTypes = this.analyzeChangeTypes(diff);

    // 2. 确定提交类型
    const type = this.determineType(changeTypes);

    // 3. 确定影响范围
    const scope = this.determineScope(diff);

    // 4. 生成主题
    const subject = this.generateSubject(diff, changeTypes);

    // 5. 生成详细描述
    const body = this.generateBody(diff);

    // 6. 组装提交信息
    return this.formatMessage(type, scope, subject, body);
  }

  private analyzeChangeTypes(diff: FileDiff[]): ChangeType[] {
    const types: ChangeType[] = [];

    for (const file of diff) {
      if (this.isNewFeature(file)) {
        types.push('feature');
      } else if (this.isBugFix(file)) {
        types.push('fix');
      } else if (this.isRefactoring(file)) {
        types.push('refactor');
      } else if (this.isDocumentation(file)) {
        types.push('docs');
      } else if (this.isTest(file)) {
        types.push('test');
      }
    }

    return types;
  }

  private determineType(types: ChangeType[]): CommitType {
    const priority: CommitType[] = ['feat', 'fix', 'refactor', 'docs', 'test', 'chore'];

    for (const type of priority) {
      if (types.includes(type as ChangeType)) {
        return type;
      }
    }

    return 'chore';
  }

  private generateSubject(diff: FileDiff[], types: ChangeType[]): string {
    // 基于变更内容生成描述性主题
    const actions = {
      feature: '添加',
      fix: '修复',
      refactor: '重构',
      docs: '更新文档',
      test: '添加测试',
      chore: '更新'
    };

    const action = actions[this.determineType(types)];
    const target = this.describeChanges(diff);

    return `${action}${target}`;
  }

  private formatMessage(
    type: CommitType,
    scope: string,
    subject: string,
    body: string
  ): string {
    let message = type;

    if (scope) {
      message += `(${scope})`;
    }

    message += `: ${subject}`;

    if (body) {
      message += `\n\n${body}`;
    }

    return message;
  }
}
```

### PR自动化

```typescript
class PRAutomation {
  async createPR(options: PRCreationOptions): Promise<PRInfo> {
    // 1. 推送分支
    await this.git.push('origin', options.branch);

    // 2. 生成PR内容
    const title = await this.generateTitle(options);
    const body = await this.generateBody(options);

    // 3. 创建PR
    const pr = await this.github.createPullRequest({
      owner: options.owner,
      repo: options.repo,
      title,
      body,
      head: options.branch,
      base: options.base
    });

    // 4. 添加标签
    if (options.labels) {
      await this.github.addLabels(pr.number, options.labels);
    }

    // 5. 请求审查
    if (options.reviewers) {
      await this.github.requestReview(pr.number, options.reviewers);
    }

    return pr;
  }

  private async generateTitle(options: PRCreationOptions): Promise<string> {
    // 分析分支名和commit信息
    const commits = await this.git.log({ limit: 5 });

    if (options.branch.startsWith('feature/')) {
      const feature = options.branch.replace('feature/', '');
      return `feat: ${this.humanize(feature)}`;
    }

    if (options.branch.startsWith('fix/')) {
      const fix = options.branch.replace('fix/', '');
      return `fix: ${this.humanize(fix)}`;
    }

    // 基于最近的commit
    return commits[0]?.subject || 'Update';
  }

  private async generateBody(options: PRCreationOptions): Promise<string> {
    const sections: string[] = [];

    // 变更概述
    const diff = await this.git.diff(`${options.base}...${options.branch}`);
    sections.push('## 变更概述');
    sections.push(this.summarizeChanges(diff));

    // 详细变更
    sections.push('## 详细变更');
    sections.push(this.formatChanges(diff));

    // 测试情况
    sections.push('## 测试');
    sections.push(await this.checkTestStatus());

    // 检查清单
    sections.push('## 检查清单');
    sections.push(this.generateChecklist());

    return sections.join('\n\n');
  }
}
```

---

## 📚 实战案例：自动化发布流程

### 需求
- 🚀 自动化版本发布
- 📝 自动生成CHANGELOG
- 🏷️ 自动打标签
- 📦 自动创建GitHub Release

### 实现

```bash
#!/bin/bash
# release.sh - 自动化发布脚本

VERSION=$1
TYPE=$2  # major, minor, patch

# 1. 验证输入
if [ -z "$VERSION" ]; then
    echo "❌ 请提供版本号"
    echo "用法: ./release.sh 1.2.0 minor"
    exit 1
fi

# 2. 检查工作目录是否干净
if [ -n "$(git status --porcelain)" ]; then
    echo "❌ 工作目录不干净，请先提交变更"
    exit 1
fi

# 3. 创建发布分支
echo "🌿 创建发布分支..."
git checkout -b "release/v${VERSION}"

# 4. 更新版本号
echo "🔢 更新版本号..."
npm version "$VERSION" --no-git-tag-version

# 5. 更新CHANGELOG
echo "📝 更新CHANGELOG..."
npx conventional-changelog -p angular -i CHANGELOG.md -s

# 6. 提交变更
echo "💾 提交变更..."
git add .
git commit -m "chore(release): ${VERSION}"

# 7. 合并到main
echo "🔀 合并到main..."
git checkout main
git merge "release/v${VERSION}" --no-ff -m "Merge release ${VERSION}"

# 8. 打标签
echo "🏷️  打标签..."
git tag -a "v${VERSION}" -m "Release ${VERSION}"

# 9. 推送到远程
echo "🚀 推送到远程..."
git push origin main
git push origin "v${VERSION}"

# 10. 创建GitHub Release
echo "📦 创建GitHub Release..."
gh release create "v${VERSION}" \
    --title "Release ${VERSION}" \
    --notes-file CHANGELOG.md \
    --latest

# 11. 清理
echo "🧹 清理..."
git branch -D "release/v${VERSION}"

echo "✅ 发布完成！版本: ${VERSION}"
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 掌握基本Git命令
- ✅ 理解Git集成优势
- ✅ 学会查看状态和历史

### 中级要点
- ✅ 掌握智能提交流程
- ✅ 理解提交信息规范
- ✅ 学会PR管理
- ✅ 掌握Git工作流

### 专家级要点
- ✅ 深入Git操作封装
- ✅ 掌握智能消息生成
- ✅ 理解PR自动化
- ✅ 掌握发布流程

---

**下一步：** 学习 [11 - 终端交互](./11-terminal-interaction.md) 🚀
