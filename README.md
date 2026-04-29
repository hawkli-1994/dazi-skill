# 搭子.skill

> *别滑了。你的 AI 已经很了解你，让它帮你找对的人。*

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://python.org)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![AgentSkills](https://img.shields.io/badge/AgentSkills-Standard-green)](https://agentskills.io)

&nbsp;

AI 原生社交匹配。不需要 App，不需要滑动，不需要填表。
告诉你的 AI 你想认识什么样的人，它帮你找到、帮你推荐理由、帮你发请求。
**双向确认后才交换联系方式。**

[安装](#安装) · [使用](#使用) · [效果示例](#效果示例) · [网站](https://dazi.network)

---

## 安装

告诉你的 AI：

> 帮我安装 dazi skill https://raw.githubusercontent.com/wfnuser/dazi-skill/main/SKILL.md

### Claude Code

```bash
claude install-skill https://raw.githubusercontent.com/wfnuser/dazi-skill/main/SKILL.md
```

### 依赖

- Python 3.10+（用于身份签名）
- 联网（调用 dazi-network API）
- 不需要 GPU，不需要本地模型，不需要 Docker

---

## 使用

在 Claude Code 中输入 `/dazi`，或者直接说：

```
帮我找搭子
```

首次使用会自动注册（30 秒），之后直接描述你想认识什么样的人就行。

### 你可以说

| 说法 | 效果 |
|------|------|
| "帮我找个聊得来的搭子" | 搜索匹配 |
| "想认识做独立开发的人" | 按兴趣搜索 |
| "有人想认识我吗" | 查看收件箱 |
| "更新一下我的资料" | 修改 profile |

---

## 效果示例

**注册（30 秒）**

```
❯ /dazi

  昵称？
  微扰

  出生年份、性别、常驻城市？
  1994, M, 北京

  联系方式？（匹配成功后才展示给对方）
  wechat: wfnuser

  用 3 句话描述自己：
  1. 在五道口开过一家剧本杀店，惨遭合伙人卷款跑路
  2. 机缘巧合下 26 Fall 可能要去读 PhD
  3. 集齐过BAT的工牌，选择去做自由的独立开发者

  ✓ 注册成功！
```

**找搭子**

```
❯ 想认识喜欢户外运动的小伙伴

  1. 李新宇 — 7/10
     - "infp"
     - "喜欢做视频"
     - "喜欢爬山"
     推荐理由：都喜欢户外运动，他爱爬山你可以一起。
     INFP 安静但真诚，适合不需要社交压力的运动搭子。

  想认识谁？
```

**双向匹配**

```
❯ 有人想认识我吗

  Deadpool 想认识你
    - "博士在读，最近在做AI Agent Infra"
    - "喜欢LOL"
    - "想做点有意义的事情"
    对方说："你在探索 AI 原生社交，我在做 AI Agent Infra，
    感觉我们在技术和方向上会很聊得来。"
    AI 分析：你们都在 AI Agent 方向探索，互补性很强。
    → 接受 / 拒绝？

  接受

  ✓ 匹配成功！对方联系方式：email: wadepan.cs@foxmail.com
```

---

## 工作原理

```
你说 "想认识做独立开发的人"
          ↓
    服务端 AI 解析意图 → 提取筛选条件 + 选择匹配维度
          ↓
    5 维 embedding 向量排序（性格/兴趣/价值观/生活方式/综合）
          ↓
    返回 top 30 候选人（昵称 + 标签）
          ↓
    你本地的 AI 精排 + 生成推荐理由
          ↓
    你挑人 → 发送请求 → 双向确认 → 交换联系方式
```

---

## 隐私

- 联系方式**加密存储**，只在双向匹配后解密展示
- 出生年份存为数字，展示给对方时只显示年龄段（如 "30s"）
- 你的 3 个标签会原样展示给潜在匹配 — 写你愿意公开的内容
- 服务端分析你的标签用于改善匹配，但**分析结果不会展示给其他用户**
- 随时可以删除账号和所有数据

---

## 写在最后

社交 App 让你滑 1000 个人，匹配 10 个，聊 3 个，见 1 个，然后发现不合适。

你的 AI 每天跟你聊 8 小时，它比任何算法都懂你是什么样的人。
让它帮你找搭子，不比你自己滑靠谱？

Apache-2.0 License © [微扰](https://www.xiaohongshu.com/user/profile/5b0d752e11be104d5db639f3)
