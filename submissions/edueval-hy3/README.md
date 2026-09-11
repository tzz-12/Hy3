# EduEval · 基于 Hy3 的初中数学教学设计质量评估器

> 本目录是 **独立应用仓库** 的提交说明，代码与完整文档在：
> **https://github.com/tzz-12/edu-eval-hy3**
>
> 对应 issue：[#4 Build a vibe-coded application powered by Hy3](https://github.com/Tencent-Hunyuan/Hy3/issues/4)

## 项目是什么

EduEval 评估 **AI 生成的初中数学单课时教学设计** 的质量。它把「教得好」这种高度主观、
无唯一标准的质量，转化为一套**可操作、可复核、可抵御表面包装**的判定结果：

- 输入：一份教学设计文本（`.md` / `.txt` / `.docx` / `.pdf` / `.pptx`）+ 声明的年级、版本、课题、课时
- 输出：准入判定（PASS / FAIL / NE）+ 0~100 总分 + 10 个维度的分项评分、证据定位与改进建议
- 形态：**Web 应用**（对话式界面 + 报告 5 个标签页 + 内嵌图表），也可走 CLI

## Hy3 承担的角色

系统是「确定性层 + Hy3 语义层」的双层结构，划分原则是：**能被程序验证的绝不交给模型。**

| 层次 | 承担者 | 职责 |
|---|---|---|
| 确定性层（零 LLM） | 本地代码 | 多格式解析、本地知识库三层检索、sympy 公式恒等验算、概念→年级映射查表、章节结构检查、维度加权聚合与准入决议 |
| 语义层 | **Hy3 API** | 事实核验、教学设计判断、表达与安全审读、证据复核与仲裁 |
| 稳定性层 | **Hy3 API** | 双采样自一致：同一提示独立采样两次，两视角分歧超阈值时交由 Hy3 仲裁 |

Hy3 以 **5 类角色提示**（fact / design / expression_safety / review / arbitrate）承载 10 个维度的判定；
一次完整评测在双采样开启时产生约 8~20 次 Hy3 调用。

全程**只通过 API 调用混元模型**，不训练、不微调、不做本地推理部署。

> **裁判模型口径（如实说明）**：课题要求「全程通过 API 调用 Hy3」。TokenHub 的 `hy3`
> 服务在提交窗口内返回 `HTTP 402 / code 401008`（该服务的免费体验额度已耗尽，且账号未
> 开启后付费），而同一 Key 下 `hy4-preview` 可正常调用。因此演示报告与演示视频由
> **同属混元家族的 `hy4-preview`** 产出：评测提示、判分口径、代码路径完全一致，切换只差
> `.env` 里的 `HY3_MODEL` 一行；`hy3` 服务开通后付费后改回该行即得到严格的 Hy3 版本。
> 每份报告内嵌 `judge` 字段（模型名 / 端点主机 / 采样参数 / 耗时）可供直接核对，
> 详见仓库 README §8.2。

## 对照 issue 要求

| 要求 | 落实情况 |
|---|---|
| 全程通过 API 调用 Hy3，不训练 / 微调 / 本地部署 | 所有语义判定经 Hy3 API；密钥走环境变量，不硬编码 |
| 至少 1 个可交互前端 | Web 应用：对话式评测界面 + 报告 5 标签页（总览 / 维度详情 / 一致性 / 规则层 / 改进建议）+ 评测说明面板 |
| 至少跑通 2 个端到端 demo 流程 + ≤2 min 视频 | 3 条流程：① 好样本 PASS ② 公式错误被规则层 0.5 秒拦截 FAIL ③ 伪启发包装被维度 9 判低分；演示视频见独立仓库 [`docs/demo.mp4`](https://github.com/tzz-12/edu-eval-hy3/blob/main/docs/demo.mp4)（98 秒） |
| 项目开源，README 写明 Hy3 角色 | 仓库 public（MIT），README §1.1 专节说明 Hy3 的职责与不负责的部分 |
| README 记录 AI 协作范围 | README §10「AI 协作说明」按模块记录 WorkBuddy 协作范围 |

## 怎么跑

```bash
git clone https://github.com/tzz-12/edu-eval-hy3
cd edu-eval-hy3
cp .env.example .env      # 填 HY3_BASE_URL / HY3_API_KEY / HY3_MODEL=hy3
pip install -r requirements.txt

# 命令行
PYTHONPATH=src python -m edu_eval 教案.md --grade 八年级 --topic 二次函数

# Web 应用
bash scripts/run_demo.sh start 8000 --live
```

## 声明

本仓库为腾讯犀牛鸟开源实战「混元大语言模型」项目的个人 / 活动作品，**并非腾讯官方发布**，
不代表腾讯公司立场。本 PR 只提交项目说明与仓库链接，**不改动 Hy3 仓库自身的代码**。
