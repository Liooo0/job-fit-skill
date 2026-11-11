# job-fit-skill — 简历 × JD 岗位匹配评分（Agent Skill）

[![CI](https://github.com/Liooo0/job-fit-skill/actions/workflows/ci.yml/badge.svg)](https://github.com/Liooo0/job-fit-skill/actions/workflows/ci.yml)

把简历和 JD 贴给会用的 agent（如 Claude Code），它调**确定性规则引擎**给出可解释的五维匹配报告：
总分 + 分档、技能命中、相关缺口、风险提示，再附面试追问建议。

**零 LLM 打分**：分数由纯 Python 规则引擎算出——同一份 JD 昨天今天分数一致，每个分数能解释到具体命中词。
装完完全离线，简历和 JD 不发给任何第三方服务。

来自 [Job Hunter](https://github.com/Liooo0/job-hunter)（AI 求职工作流套件的评分引擎，`pip` 包 `jobhunter-engine`）。

## 试用（60 秒）

```bash
pip install "jobhunter-engine @ git+https://github.com/Liooo0/job-hunter.git@v1.0.0"
```

然后把本仓库 `SKILL.md` + 一份简历 + 一个 JD 交给你的 agent；或者跳过 agent 直接跑：

```bash
# 用示例配置（由 example/resume-sample.md 抽取而来）
jobhunter-score "AI应用工程师" \
  --desc-file example/jd-sample.txt \
  --salary "15-25K" --city 深圳 \
  --config example/config.sample.json
```

实际输出（Python 3.11 实测，`example/expected-output.txt` 同文）：

```
┌─ 岗位匹配报告 ──────────────────────────────────
│ 总分: 88/100 → 🚀 强烈推荐 (strong_apply)
├─ 五维明细 ───────────────────────────────────
│ 技术匹配 █████████░  95 ×30% →  28.5   技能命中: Python/FastAPI/RAG/Agent/知识库/工作流
│ 职业方向 █████████░  95 ×30% →  28.5   目标岗位命中: AI应用工程师
│ 经验匹配 ███████░░░  70 ×15% →  10.5   无明显经验表述
│ 文化匹配 ███████░░░  75 ×15% →  11.2   无文化风险/加分信号(默认75)
│ 地点通勤 █████████░  90 ×10% →   9.0   深圳=primary城市
├─ 技能命中: Python、FastAPI、RAG、Agent、知识库、工作流
├─ 相关缺口: 大模型
└─ 备注: target_roles 来自 config | 关键词层得分=60(目标岗位:AI应用工程师、技能匹配:Python/FastAPI/RAG) | 加权原始值=87.70
```

agent 在这份报告上补三件事：结论一句话、缺口是"可改"还是"硬伤"、以及每个缺口的面试追问。

## 分档规则

| 总分 | 结论 |
|---|---|
| ≥75 | 🚀 强烈建议投（strong_apply） |
| 60-74 | ✅ 值得投（apply） |
| 45-59 | 🤔 观望（consider） |
| <45 | ⏭️ 建议跳过（skip） |

## 五维权重

技术匹配 30% · 职业方向 30% · 经验匹配 15% · 文化匹配 15% · 地点通勤 10%
（权重与词表可在 job-hunter 的 config 里定制，见其 README）

## 它在套件里的位置

| 项目 | 角色 |
|---|---|
| [job-fit-skill](https://github.com/Liooo0/job-fit-skill)（本仓库） | 最轻入口：贴简历+JD → 匹配分，免登录免配置 |
| [Job Hunter](https://github.com/Liooo0/job-hunter) | 完整工作流：规则过滤 → 打分 → 拟真投递 → 失败追踪 |
| [boss-zhipin-helper](https://github.com/Liooo0/boss-zhipin-helper) | Chrome 扩展：JD 即时提炼 / AI 回复 |
| [jobintel-dashboard](https://github.com/Liooo0/jobintel-dashboard) | 数据看板：4.7 万条岗位判定记录只读复盘 |

## 边界

- 只评分和解读，不做任何自动投递、不接触招聘平台
- 打分是关键词层的确定性计算，不构成对个人能力的评价
- 示例简历/JD 均为合成数据，不含真人信息
