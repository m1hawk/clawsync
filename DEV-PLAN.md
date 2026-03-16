# Clawsync 开发计划

> 让你与 AI 的关系更有温度

## 项目概述

Clawsync 是一个 **OpenClaw Skill 插件**，通过分析用户与 AI 的对话历史，识别情感化表达，计算同步率，并据此调整 AI 的回应风格。

## OpenClaw Skill 架构

作为 OpenClaw Skill，Clawsync 运行在 OpenClaw Agent 运行时内，具有以下特点：

- **Skill 位置**: `~/.openclaw/workspace/skills/clawsync/`
- **核心文件**: `SKILL.md` (定义 Skill 行为和响应风格)
- **数据存储**: `~/.openclaw/syncrate/` (同步率状态和历史)
- **运行时集成**: 自动加载，随 Agent 运行

## 技术栈

- **定义语言**: Markdown (SKILL.md)
- **配置格式**: JSON
- **运行环境**: OpenClaw Agent Runtime
- **依赖**: OpenClaw 内置工具 (`sessions_history`, `cron`, `delivery`)

## Skill 文件结构

```
~/.openclaw/workspace/skills/clawsync/
├── SKILL.md                 # Skill 定义文件 (核心)
├── config.json              # 默认配置
├── emotion-words.json       # 情感词库
└── styles/
    ├── warm.md              # 温暖向风格指南
    └── humorous.md          # 毒舌幽默向风格指南
```

## 运行时生成的文件

```
~/.openclaw/
├── syncrate/
│   ├── state.json           # 当前同步率状态
│   └── history.jsonl        # 同步率变化历史
└── workspace/
    └── SYNCRATE.md          # 当前同步率状态（供 Agent 读取）
```

---

## 开发阶段

### Phase 1: Skill 基础结构

**目标**: 创建 OpenClaw Skill 基础文件

**任务**:
- [ ] 创建 `SKILL.md` - Skill 定义文件
- [ ] 创建 `config.json` - 默认配置
- [ ] 创建 `emotion-words.json` - 情感词库
- [ ] 创建目录结构
  ```
  clawsync/
  ├── SKILL.md
  ├── config.json
  ├── emotion-words.json
  └── styles/
      ├── warm.md
      └── humorous.md
  ```

---

### Phase 2: SKILL.md 核心定义

**目标**: 编写 Skill 核心定义文件

**任务**:
- [ ] 定义 Skill 元数据 (name, description, metadata)
- [ ] 编写响应风格指南读取逻辑
- [ ] 定义 `/syncrate` 命令处理
- [ ] 定义 `/syncrate style <warm|humorous>` 命令处理
- [ ] 定义 `/syncrate history` 命令处理

---

### Phase 3: 情感词库

**目标**: 创建情感分析词库

**任务**:
- [ ] `emotion-words.json` - 多语言情感词库
  - `zh-CN.positive` - 中文正面情感词
  - `zh-CN.negative` - 中文负面情感词
  - `en.positive` - 英文正面情感词
  - `en.negative` - 英文负面情感词
  - `taskPatterns` - 任务模式排除词
  - `techPatterns` - 技术模式排除词

---

### Phase 4: 风格指南模板

**目标**: 创建两种性格风格的详细指南

**任务**:
- [ ] `styles/warm.md` - 温暖向风格指南
  - 各等级的回应风格
  - 语气、用词规范
  - 禁止事项
- [ ] `styles/humorous.md` - 毒舌幽默向风格指南
  - 各等级的回应风格
  - 吐槽风格规范
  - 傲娇表达方式

---

### Phase 5: Cron 每日分析任务

**目标**: 实现每日同步率计算

**任务**:
- [ ] 编写 Cron 任务逻辑 (在 SKILL.md 中定义)
  - 读取 `sessions_history` 最近一天的消息
  - 第一阶段：关键词筛选
  - 第二阶段：LLM 精确分析（仅对混合消息）
  - 计算同步率变化（受每日上限限制）
  - 应用衰减规则
  - 更新 `state.json` 和 `SYNCRATE.md`
  - 追加 `history.jsonl`
  - 发送每日摘要通知

---

### Phase 6: 首次安装流程

**目标**: 实现首次安装时的历史分析

**任务**:
- [ ] 检测是否首次安装 (`state.json` 是否存在)
- [ ] 读取最近 30 天 `sessions_history`
- [ ] 批量情感分析
- [ ] 计算初始同步率（无上限）
- [ ] 生成初始 `state.json` 和 `SYNCRATE.md`
- [ ] 发送欢迎通知

---

### Phase 7: 通知系统

**目标**: 实现通知推送

**任务**:
- [ ] 欢迎通知 (首次安装)
- [ ] 每日摘要通知 (Cron 任务后)
- [ ] 升级通知 (等级跨越阈值时)
- [ ] 使用 OpenClaw `delivery` 工具发送

---

### Phase 8: 配置系统

**目标**: 实现用户可配置选项

**任务**:
- [ ] `config.json` 默认配置
  - `levelUpSpeed` - 升级速度
  - `dailyMaxIncrease` - 每日最大增长
  - `dailyDecay` - 每日衰减值
  - `decayThresholdDays` - 衰减阈值天数
  - `personalityType` - 性格类型
  - `customLevels` - 自定义等级名称
- [ ] 配置读取和验证逻辑

---

### Phase 9: 状态管理

**目标**: 实现同步率状态持久化

**任务**:
- [ ] `state.json` 结构设计
  - 当前同步率
  - 当前等级
  - 当前风格
  - 最后更新日期
  - 连续无互动天数
- [ ] `history.jsonl` 结构设计
  - 日期、同步率变化、触发原因
- [ ] `SYNCRATE.md` 自动生成
  - 当前状态展示
  - 风格指南摘要

---

### Phase 10: 测试和文档

**目标**: 确保质量和完善文档

**任务**:
- [ ] 测试各命令功能
- [ ] 测试 Cron 任务执行
- [ ] 测试首次安装流程
- [ ] 测试风格切换
- [ ] 完善 SKILL.md 文档
- [ ] 编写 README.md

---

## 文件清单

| 文件 | 描述 | 优先级 |
|-----|-----|-------|
| `SKILL.md` | Skill 定义文件 (核心) | P0 |
| `config.json` | 默认配置 | P0 |
| `emotion-words.json` | 情感词库 | P0 |
| `styles/warm.md` | 温暖向风格指南 | P1 |
| `styles/humorous.md` | 毒舌幽默向风格指南 | P1 |
| `README.md` | 项目说明文档 | P2 |

---

## OpenClaw 工具依赖

| 工具 | 用途 |
|-----|-----|
| `sessions_history` | 读取对话历史进行分析 |
| `cron` | 每日定时执行同步率计算 |
| `delivery` | 发送通知消息 |
| `fs` | 读写状态文件 |
| `llm` | 对混合消息进行精确分析 |

---

## 测试覆盖目标

- 命令响应测试: `/syncrate`, `/syncrate style`, `/syncrate history`
- 情感分析测试: 关键词检测、LLM 分析
- 同步率计算测试: 每日上限、衰减、等级计算
- 首次安装测试: 历史分析、初始同步率计算
- 通知测试: 各类通知发送

