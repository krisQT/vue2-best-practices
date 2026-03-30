# Vue2-best-practices 使用指南

本文档详细说明如何在项目中使用 Vue2-best-practices skill。

## 目录

- [方法一：本地开发模式（推荐）](#方法一本地开发模式推荐)
- [方法二：从 GitHub 安装](#方法二从-github-安装)
- [方法三：Claude Code Marketplace](#方法三claude-code-marketplace)
- [使用技巧](#使用技巧)
- [验证安装](#验证安装)
- [常见问题](#常见问题)

---

## 方法一：本地开发模式（推荐）

适用于：
- 修改或测试 skill 内容
- 离线使用
- 开发新的 skill

### 步骤 1：克隆项目到本地

```bash
# 克隆到本地
git clone https://github.com/krisQT/vue2-best-practices.git

# 进入目录
cd vue2-best-practices
```

### 步骤 2：在目标项目中添加 Skill

在你的 Vue 2 项目根目录：

```bash
# 方式 A：使用 npm 安装 skills CLI（如果需要）
npm install -g @skill CLI

# 方式 B：直接引用本地路径（推荐用于开发）
# 在项目根目录创建 .skills 配置
mkdir -p .skills
ln -s /path/to/vue2-best-practices/skills .skills/vue2-best-practices
```

### 步骤 3：在 Trae IDE 中使用

1. **打开你的 Vue 2 项目**

2. **触发 Skill**
   ```
   在 Trae 的对话中输入：
   
   Use vue2 skill, <你的需求>
   
   例如：
   Use vue2 skill, 帮我创建一个用户列表组件
   Use vue2 skill, 如何解决 Vue 2 响应式不生效的问题
   ```

3. **自动识别（无需前缀）**
   Trae 会根据关键词自动匹配 skill，例如：
   - 提到 "Vue 2"、"Options API" → 触发 vue2-best-practices
   - 提到 "Vue Router" → 触发 vue2-router-best-practices
   - 提到 "Vuex" → 触发 vue2-vuex-best-practices

---

## 方法二：从 GitHub 安装

适用于：
- 使用发布的稳定版本
- 不需要修改 skill 内容

### 步骤 1：使用 npx 安装

```bash
npx skills add krisQT/vue2-best-practices
```

### 步骤 2：验证安装

```bash
# 检查已安装的 skills
npx skills list

# 应该看到类似输出：
# ✓ vue2-best-practices
# ✓ vue2-components-best-practices
# ✓ vue2-router-best-practices
# ... (9 个 skills)
```

### 步骤 3：在 Trae 中使用

安装后，直接在对话中使用：

```
帮我用 Vue 2 创建一个 Todo 应用，包含以下功能：
1. 添加待办事项
2. 标记完成
3. 删除待办
```

---

## 方法三：Claude Code Marketplace

适用于：
- Claude Code 用户
- 团队共享配置

### 安装步骤

```bash
# 添加 marketplace
/plugin marketplace add krisQT/vue2-best-practices

# 安装所有 skills
/plugin install vue2-skills-bundle@vue2-best-practices

# 或安装单个 skill
/plugin install vue2-best-practices@vue2-best-practices

# 安装多个 skills
/plugin install vue2-best-practices@vue2-best-practices vue2-router-best-practices@vue2-best-practices
```

---

## 使用技巧

### 1. 明确指定 Skill

获得最准确的结果：

```
Use vue2 router skill, 如何实现路由守卫验证登录状态
```

### 2. 组合使用多个 Skills

```
Use vue2 skill, Use vue2 router skill, 
创建一个带用户认证的 Vue 2 应用
```

### 3. 描述具体问题

```
Use vue2 skill, 我的数组修改后 UI 没有更新，怎么回事？
```

### 4. 请求最佳实践

```
Use vue2 skill, 推荐一些 Vue 2 组件设计最佳实践
```

---

## 验证安装

### 测试 Skill 是否生效

尝试以下对话：

**测试 1：基础识别**
```
Create a Vue 2 component with props and events
```
应该自动使用 vue2-components-best-practices

**测试 2：明确触发**
```
Use vue2 skill, 什么是 Vue 2 的响应式系统限制？
```

**测试 3：版本问题**
```
Use vue2 skill, .sync 修饰符在哪个版本可用？
```

---

## 常见问题

### Q1: 如何确认 skill 已正确安装？

检查项目目录是否有 `.skills` 文件夹：

```bash
ls -la .skills/  # 应该看到 skills 子目录
```

### Q2: 如何更新到最新版本？

```bash
# 如果使用 GitHub 安装
npx skills update krisQT/vue2-best-practices

# 如果使用本地开发模式
cd vue2-best-practices
git pull origin main
```

### Q3: 如何禁用某个 skill？

根据你使用的 CLI 不同，命令可能不同：
- 查看 CLI 帮助：`npx skills --help`
- 或删除 `.skills` 中的对应目录

### Q4: skill 之间有冲突吗？

不会。所有 skill 都是独立的内容，但：
- `vue2-best-practices` 是最基础的，包含通用模式
- 其他 skill 是专项的（路由、Vuex 等）
- AI 会根据上下文选择最相关的 skill

### Q5: 支持 TypeScript 吗？

是的！文档中包含了 TypeScript 示例：
- Props 类型定义
- Route Meta 类型
- 组件类型

---

## Skill 快速参考

| Skill | 触发关键词 | 用途 |
|-------|----------|------|
| vue2-best-practices | Vue 2, Options API | 核心概念和最佳实践 |
| vue2-options-api | this 上下文, lifecycle | Options API 深度指南 |
| vue2-components | props, events, slots | 组件设计和通信 |
| vue2-router | Vue Router, navigation | 路由和导航守卫 |
| vue2-vuex | Vuex, state | 状态管理 |
| vue2-mixins | mixins | 混入模式 |
| vue2-directives | custom directive | 自定义指令 |
| vue2-testing | Jest, testing | 测试实践 |
| vue2-debug-guides | debug, error | 调试和问题排查 |

---

## 下一步

- 查看 [README.md](README.md) 了解所有可用的 skills
- 查看 [AGENTS.md](AGENTS.md) 了解 AI Agent 使用指南
- 查看 [示例](./skills/vue2-best-practices/SKILL.md) 了解实际使用效果
