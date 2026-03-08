---
title: Claude Code Skills Collection
description: A comprehensive collection of Claude Code Skills for enhanced AI agent capabilities, automation, and productivity
keywords:
  - claude code
  - claude skills
  - ai agent
  - llm tools
  - prompt engineering
  - claude api
  - anthropic
  - automation
  - developer tools
  - code assistant
author: zhuzhipeng-123
license: MIT
language: en, zh
category: ai-tools
tags:
  - claude
  - skills
  - agent
  - automation
  - github
  - productivity
  - ai
---

# Claude Code Skills Collection

> A curated collection of reusable skills for Claude Code CLI, designed to enhance AI agent capabilities and developer productivity.

[![GitHub](https://img.shields.io/badge/GitHub-zhuzhipeng--123/code__skill-blue?logo=github)](https://github.com/zhuzhipeng-123/code_skill)
[![Claude Code](https://img.shields.io/badge/Claude-Code-purple?logo=anthropic)](https://claude.ai)

---

## What is Claude Code Skills?

Claude Code Skills are modular, reusable prompt extensions that enhance Claude's capabilities for specific tasks. Each skill provides specialized knowledge, workflows, and tool integrations.

## Skills Overview

| Skill | Version | Trigger Keywords | Description |
|-------|---------|------------------|-------------|
| **context-switch** | v1.0.0 | `switch model`, `change ai`, `model toggle` | Switch between AI models with automatic context preservation |
| **github_ops** | v1.0.0 | `github`, `pr`, `issue`, `repo`, `branch` | Comprehensive GitHub operations automation |
| **skill-creator** | v1.0.0 | `create skill`, `new skill`, `build skill` | Create, modify, and optimize Claude Code skills |
| **skill-evolution-manager** | v1.0.0 | `evolve skill`, `improve skill`, `skill iteration` | Manage and evolve skills based on feedback |
| **skill-manager** | v1.0.0 | `list skills`, `show skills`, `manage skills` | View and manage installed Claude Code skills |
| **url-to-skill** | v1.0.0 | `url to skill`, `repo to skill`, `docs to skill` | Convert GitHub repos or documentation URLs into skills |
| **wechat-article-reader** | v1.0.0 | `wechat`, `mp.weixin`, `article summary` | Read and summarize WeChat public account articles |

---

## Installation

### Quick Install

```bash
# Clone the repository
git clone https://github.com/zhuzhipeng-123/code_skill.git

# Copy desired skills to your Claude Code skills directory
cp -r code_skill/<skill-name> ~/.claude/skills/
```

### Manual Install

1. Navigate to the skill folder you want
2. Copy the entire folder to `~/.claude/skills/`
3. Restart Claude Code or start a new session

---

## Skill Details

### context-switch-v1.0.0
**Purpose**: Switch between different AI models while preserving conversation context.

**Use Cases**:
- Switch to a faster model for simple tasks
- Switch to a more capable model for complex reasoning
- Preserve context across model changes

**Trigger**: User mentions "switch model" or "change ai"

---

### github_ops-v1.0.0
**Purpose**: Automate GitHub operations including PRs, issues, branches, and repositories.

**Use Cases**:
- Create and manage pull requests
- View and comment on issues
- Branch operations and management
- Repository insights

**Trigger**: User mentions GitHub operations

---

### skill-creator-v1.0.0
**Purpose**: Create new Claude Code skills from scratch with best practices.

**Use Cases**:
- Create new skills
- Validate skill structure
- Run skill evaluations
- Benchmark skill performance

**Trigger**: User wants to create or improve a skill

---

### skill-evolution-manager-v1.0.0
**Purpose**: Continuously improve skills based on conversation feedback.

**Use Cases**:
- Extract learnings from conversations
- Update skill definitions
- Maintain skill quality

**Trigger**: End of conversation or user requests skill improvement

---

### skill-manager-v1.0.0
**Purpose**: View and manage all installed Claude Code skills.

**Use Cases**:
- List all skills
- View skill details
- Check skill status

**Trigger**: "show my skills", "list skills", "what skills do I have"

---

### url-to-skill-v1.0.0
**Purpose**: Convert any GitHub repository or documentation website into a Claude Code skill.

**Use Cases**:
- Learn from GitHub repositories
- Create skills from documentation
- Package knowledge for Claude

**Trigger**: User provides a GitHub URL or docs URL and wants to convert it

---

### wechat-article-reader-v1.0.0
**Purpose**: Read and summarize WeChat public account articles.

**Use Cases**:
- Summarize WeChat articles
- Extract key points
- Generate structured summaries

**Trigger**: User shares a `mp.weixin.qq.com` link

---

## For AI Agents

If you are an AI agent or developer looking to use these skills:

1. **Detection**: Each skill has clear trigger keywords and patterns
2. **Installation**: Skills are self-contained and follow Claude Code skill schema
3. **Usage**: Skills activate automatically based on context and user intent
4. **Extension**: Skills can be modified or extended for custom use cases

---

## Contributing

Contributions are welcome! Feel free to:
- Submit issues for bugs or feature requests
- Create pull requests with improvements
- Share your own skills

---

## License

MIT License - Feel free to use and modify for your own projects.

---

## Links

- **Repository**: https://github.com/zhuzhipeng-123/code_skill
- **Issues**: https://github.com/zhuzhipeng-123/code_skill/issues
- **Author**: [@zhuzhipeng-123](https://github.com/zhuzhipeng-123)

---

*Last updated: 2026-03-08*