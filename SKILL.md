---
name: job-fit-score
description: 岗位匹配打分。用户贴出简历和 JD（或给文件路径），想知道"这个岗位值不值得投 / 我和 JD 匹配多少分 / 帮我评估这个岗位"时使用。输出可解释的五维匹配报告（技术/方向/经验/文化/地点）+ 缺口与风险 + 面试追问建议。触发词：匹配打分、岗位评估、值不值得投、job fit。
origin: https://github.com/Liooo0/job-fit-skill
---

# Job Fit Score — 简历 × JD 岗位匹配评分

确定性规则引擎打分（零 LLM、零 token、可复现），语言模型只负责解读和追问建议。
引擎来自 [job-hunter](https://github.com/Liooo0/job-hunter)（`jobhunter-engine` pip 包）。

## Step 0：确认引擎可用

```bash
jobhunter-score --help 2>/dev/null || pip install "jobhunter-engine @ git+https://github.com/Liooo0/job-hunter.git@v1.0.0"
```

装完完全离线运行，不会把简历或 JD 发给任何服务。

## Step 1：拿到两样输入

- 简历（粘贴文本或文件路径，.md / .txt / .pdf 均可；PDF 只取文本）
- JD（粘贴文本或文件路径），尽量带上：岗位标题、薪资区间、城市

缺 JD 时提示用户补；缺简历时可用 `example/resume-sample.md` 的结构引导用户粘贴关键段。

## Step 2：从简历抽画像（写临时 config）

从简历抽 8-15 个技能词 + 1-3 个目标岗位方向，写入临时 JSON：

- 技能词：硬技能（语言/框架/工具/领域词，如 Python、FastAPI、RAG、Agent、知识库）
- 跳过：软技能（沟通能力等）、公司名、学校名
- 目标岗位：用户明确说的方向，没说就从简历最近经历推 1-3 个

```json
{ "skills": ["Python", "FastAPI", "RAG", "Agent"], "target_roles": ["AI应用工程师"] }
```

参考 `example/config.sample.json`。

## Step 3：跑引擎

```bash
jobhunter-score "岗位标题原样" \
  --desc-file /tmp/jd.txt \
  --salary "15-25K" --city 深圳 \
  --config /tmp/jobfit-config.json
```

- JD 正文先写到临时文件再用 `--desc-file`（长 JD 别塞命令行参数）
- 城市取 JD 里的工作地；薪资原样传（如 "15-25K"）

## Step 4：解读报告（语言模型在这步才出场）

向用户呈现引擎输出后，补三层人话：

1. **结论**：总分 + 分档（≥75 强烈建议投 / 60-74 值得投 / 45-59 观望 / <45 建议跳过），一句话说清为什么
2. **差距**：逐条解释「相关缺口」和「风险提示」——是关键词没对上（改简历可解）还是硬性不符（改不了）
3. **面试追问**：针对每条缺口，生成 1-2 个面试官最可能追问的问题 + 一句应对思路

## 红线

- 只评分和解读，**绝不自动投递**、不联系任何平台
- 简历内容只用于生成本地临时 config，不写入对话以外的任何地方
- 分数是关键词层的确定性计算，不是对用户能力的评价——措辞上别让用户误会
