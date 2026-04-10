---
name: deep-research-zh
description: Conduct systematic academic literature reviews in 6 phases, producing structured notes, a curated paper database, and a synthesized final report. Output is organized by phase for clarity.
argument-hint: [topic]
---

# 深度研究技能 (Deep Research Skill)

## 触发条件 (Trigger)

当用户想要执行以下操作时激活此技能：
- “研究一个话题”、“文献综述”、“查找关于...的论文”、“关于...的综述论文”
- “深入研究 [话题]”、“[话题] 的 SOTA（当前最高水平）是什么”
- 使用 `/research <topic>` 斜杠命令

## 概述 (Overview)

本技能通过 6 个阶段进行系统的学术文献综述，生成结构化笔记、精选论文数据库以及综合性的最终报告。输出按**阶段 (Phase)** 组织，以确保条理清晰。**所有的markdown，应该使用中文书写。**

**安装路径**: `~/.agents/skills/deep-research-zh/` —— 脚本、参考资料及此技能定义。
**输出路径**: 相对于当前工作目录的 `./deep-research-output/{slug}/`。

## 关键：严格的阶段顺序执行 (CRITICAL: Strict Sequential Phase Execution)

**你必须按照 1 → 2 → 3 → 4 → 5 → 6 的严格顺序执行所有 6 个阶段。严禁跳过任何阶段。**

这是本技能最重要的原则。违规行为包括：
- ❌ 从阶段 2 直接跳到阶段 5/6（跳过了“深度钻研”和“代码”阶段）
- ❌ 在完成阶段 3 的深度阅读之前编写综合综述或报告
- ❌ 仅根据搜索结果中的摘要/标题生成最终报告
- ❌ 合并或混淆阶段（例如，“将阶段 3-5 放在一起做”）

### 阶段准入协议 (Phase Gate Protocol)

在开始阶段 N+1 之前，你必须验证阶段 N 的**必需输出文件**是否已存在于磁盘上。如果文件不存在，则视为未完成该阶段。

| 阶段 | 准入要求：必需的输出文件 |
|-------|---------------------------|
| 1 → 2 | `phase1_frontier/frontier.md` 已存在且包含 ≥10 篇论文 |
| 2 → 3 | `phase2_survey/survey.md` 已存在且 `paper_db.jsonl` 包含 35-80 篇论文 |
| 3 → 4 | `phase3_deep_dive/selection.md` 和 `phase3_deep_dive/deep_dive.md` 已存在，且 deep_dive.md 包含 ≥8 篇论文的详细笔记 |
| 4 → 5 | `phase4_code/code_repos.md` 已存在且包含 ≥3 个代码仓库 |
| 5 → 6 | `phase5_synthesis/synthesis.md` 和 `phase5_synthesis/gaps.md` 已存在 |

**完成每个阶段后，请打印阶段完成检查点：**

### 为什么每个阶段都至关重要

- **阶段 3 (深度钻研)** 是你真正阅读论文的环节 —— 没有它，你的综合综述将流于表面，仅基于摘要。
- **阶段 4 (代码与工具)** 将研究落实到实际应用中 —— 没有它，你会错过开源生态系统。
- **阶段 5 (综合综述)** 需要阶段 3 的深度知识 —— 你无法综合分析你没读过的论文。
- **阶段 6 (报告)** 汇集了此前所有阶段的内容 —— 它应当引用阶段 3 笔记中的具体发现。

## 论文质量策略 (Paper Quality Policy)

**经同行评审的会议论文优先于 arXiv 预印本。** 许多 arXiv 论文未经过同行评审，可能包含未经验证的断言。

### 来源优先级 (由高到低)
1. **顶级期刊**
2. **同行评审期刊**
3. **研讨会论文**（门槛较低但仍经过评审）
4. **高引用的 arXiv 预印本**: 极有可能是高质量的，但仍属未验证
5. **近期的 arXiv 预印本**: 谨慎使用，需明确标注 `(preprint)` 状态

### 何时使用 arXiv 论文
- 作为同行评审工作的**补充**证据
- 针对尚未在会议发表的**极近期**结果（< 3个月）
- 尚无同行评审版本存在时 —— 在引用中注明 `(preprint)`
- 用于**综述/评论**类论文（即便没有同行评审，这类论文也很有参考价值）

## Search Tools (by priority)

### 1. search_openalex.py (primary — free, no apis)
**Location**: `~/.agents/skills/deep-research-zh/scripts/search_openalex.py`
```bash
python ~/.agents/skills/literature-search/scripts/search_openalex.py \
  --query "QUERY" --max-results 20 --year-range 2022-2026 \
  --min-citations 5 -o results_openalex.jsonl
```

### 2. search_semantic_scholar.py (supplementary — citation data + broader coverage)
**Location**: `~/.agents/skills/deep-research-zh/scripts/search_semantic_scholar.py`
Supports `--peer-reviewed-only` and `--top-conferences` filters.

### 3. search_arxiv.py (supplementary — latest preprints)
**Location**: `~/.agents/skills/deep-research-zh/scripts/search_arxiv.py`
For searching recent papers not yet published at conferences. Mark citations with `(preprint)`.

### Other Scripts
| Script | Location | Key Flags |
|--------|----------|-----------|
| `download_papers.py` | `~/.agents/skills/deep-research-zh/scripts/` | `--jsonl`, `--output-dir`, `--max-downloads`, `--sort-by-citations` |
| `extract_pdf.py` | `~/.agents/skills/deep-research-zh/scripts/` | `--pdf`, `--pdf-dir`, `--output-dir`, `--sections-only` |
| `paper_db.py` | `~/.agents/skills/deep-research-zh/scripts/` | subcommands: `merge`, `search`, `filter`, `tag`, `stats`, `add`, `export` |
| `bibtex_manager.py` | `~/.agents/skills/deep-research-zh/scripts/` | `--jsonl`, `--output`, `--keys-only` |
| `compile_report.py` | `~/.agents/skills/deep-research-zh/scripts/` | `--topic-dir` |

### WebFetch Mode (no Bash)
1. **Paper discovery**: `WebSearch` + `WebFetch` to query Semantic Scholar/arXiv APIs
2. **Paper reading**: `WebFetch` on ar5iv HTML or `Read` tool on downloaded PDFs
3. **Writing**: `Write` tool for JSONL, notes, report files

## 6 阶段工作流 (6-Phase Workflow)

### 阶段 1：前沿探索 (Frontier)
检索**最新的**会议论文集和预印本，以了解当前趋势。
1. 编写 `phase1_frontier/paper_finder_config.yaml`，目标定在最近 1-2 年
2. 使用 WebSearch 查找最新的录用论文列表
3. 识别趋势方向和关键突破
→ 输出: `phase1_frontier/frontier.md`, `phase1_frontier/search_results/`

### 阶段 2：全景调研 (Survey)
在更广泛的时间范围内构建全面的研究版图。过滤后目标为 **35-80 篇论文**。
1. 编写 `phase2_survey/paper_finder_config.yaml`，覆盖 2023-2026
2. 运行 Openalex + Semantic Scholar + arXiv 搜索
3. 合并所有结果：`python .../paper_db.py merge`
4. 过滤出 35-80 篇最相关的论文：`python .../paper_db.py filter --min-score 0.80 --max-papers 70`
5. 按主题聚类，编写调研笔记
→ 输出: `phase2_survey/survey.md`, `phase2_survey/search_results/`, `paper_db.jsonl`

### 阶段 3：深度钻研 (Deep Dive) ⚠️ 严禁跳过

**此阶段是强制性的。** 你必须完整阅读 8-15 篇论文，而不仅仅是摘要。

1. 从 `paper_db.jsonl` 中选择 8-15 篇论文并说明理由 → 写入 `phase3_deep_dive/selection.md`
2. 下载 PDF：`python download_papers.py --max-downloads 15`
3. 针对每一篇选定的论文，阅读全文（通过 `Read` 读取 PDF 或通过 `WebFetch` 在 ar5iv 阅读 HTML）
4. 为每篇论文编写详细的结构化笔记：问题、贡献、方法论、实验、局限性、关联性
5. 汇总所有笔记 → `phase3_deep_dive/deep_dive.md`

**阶段 3 准入要求**: `deep_dive.md` 必须包含 ≥8 篇论文的详细笔记，且每篇笔记的方法论和实验部分必须填充完整。仅包含摘要的总结不符合要求。

→ 输出: `phase3_deep_dive/selection.md`, `phase3_deep_dive/deep_dive.md`, `phase3_deep_dive/papers/`

### 阶段 4：代码与工具 (Code & Tools) ⚠️ 严禁跳过

**此阶段是强制性的。** 你必须调研开源生态系统。

1. 从阶段 3 阅读的论文中提取 GitHub 链接
2. 使用 WebSearch 搜索实现代码："site:github.com {方法名称}", "site:paperswithcode.com {话题}"
3. 记录找到的每个仓库：URL、Star 数、语言、最后更新时间、文档质量
4. 搜索相关的基准测试 (Benchmarks) 和数据集
5. 写入 → `phase4_code/code_repos.md`（必须包含 ≥3 个仓库）
6. 如果论文没有公开研究代码，则贴论文对应的url

→ 输出: `phase4_code/code_repos.md`

### 阶段 5：综合综述 (Synthesis)
跨论文分析。**赋予同行评审发现更高的权重**。
此阶段必须建立在阶段 3 的详细笔记和阶段 4 的代码版图之上。
包含分类法 (Taxonomy)、对比表、空白点分析 (Gap Analysis)。

→ 输出: `phase5_synthesis/synthesis.md`, `phase5_synthesis/gaps.md`

### 阶段 6：报告汇编 (Compilation)
汇集之前所有阶段的产出，生成最终报告。
**在开始前**：验证所有阶段的输出文件是否齐全。如果缺失，返回补全。

→ 输出: `phase6_report/report.md`, `phase6_report/references.bib`

## Output Directory

```
output/{topic-slug}/
├── paper_db.jsonl                    # Master database (accumulated)
├── phase1_frontier/
│   ├── paper_finder_config.yaml
│   ├── search_results/
│   └── frontier.md
├── phase2_survey/
│   ├── paper_finder_config.yaml
│   ├── search_results/
│   └── survey.md
├── phase3_deep_dive/
│   ├── papers/
│   ├── selection.md
│   └── deep_dive.md
├── phase4_code/
│   └── code_repos.md
├── phase5_synthesis/
│   ├── synthesis.md
│   └── gaps.md
└── phase6_report/
    ├── report.md
    └── references.bib
```

## Key Conventions

- **Paper IDs**: Use `arxiv_id` when available, otherwise Semantic Scholar `paperId`
- **Citations**: `[@key]` format, key = firstAuthorYearWord (e.g., `[@vaswani2017attention]`)
- **JSONL schema**: title, authors, abstract, year, venue, venue_normalized, **peer_reviewed**, citationCount, paperId, arxiv_id, pdf_url, tags, source
- **Preprint marking**: Always note `(preprint)` when citing non-peer-reviewed work
- **Incremental saves**: Each phase writes to disk immediately
- **Paper count**: Target 35-80 papers in final paper_db.jsonl (use `paper_db.py filter`)

## References

- `~/.agents/skills/deep-research-zh/references/workflow-phases.md` — Detailed 6-phase methodology
- `~/.agents/skills/deep-research-zh/references/note-format.md` — Note templates, BibTeX format, report structure
- `~/.agents/skills/deep-research-zh/references/api-reference.md` — arXiv, Semantic Scholar, ar5iv API guide

## Related 
- See also: [novelty-assessment](./references/assessment-prompts.md), [survey-generation](./references/survey-prompts.md)
