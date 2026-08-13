# 【Feature Platform】Agentic Workflow：Feature Monitoring → Diagnosis → Replacement → IDSP Handoff

- **Summary**

    收到 4 个核心需求：

    1. 支持 Feature Platform 从「找特征 / 理解特征」升级为围绕 IDSP 策略链路的 Feature Intelligence & Recovery Workflow：从特征监控异常发现，到诊断、解释、替代候选、IDSP 策略变更 proposal；
    2. 支持用户在 Feature Monitoring Dashboard 发现异常后进入 Feature Drilldown，由 Agent 自动聚合 metrics、trace、data source、strategy usage，定位异常原因和影响范围；
    3. 支持增强 Feature Card：从 Discovery Card 升级为 Diagnosis Card / Replacement Card，展示业务含义、当前健康状态、IDSP 使用情况、风险、替代候选和验证建议；
    4. 支持 Ask AI 查找替代、fallback、衍生或血缘相关特征，并生成 IDSP Strategy Change Proposal；Agent 只生成诊断、候选和 proposal，不直接修改策略，所有变更必须由人确认后进入 IDSP 配置、验证、发布和回滚链路。

    We have received 4 core requirements:

    1. Upgrade Feature Platform from “feature discovery / feature understanding” to a Feature Intelligence & Recovery Workflow around the IDSP strategy lifecycle: anomaly monitoring, diagnosis, explanation, replacement candidates, and IDSP strategy change proposal;
    2. Support Feature Drilldown from Feature Monitoring Dashboard. The Agent aggregates metrics, trace, data source, and strategy usage to identify the root cause and impacted scope;
    3. Enhance Feature Card from Discovery Card to Diagnosis Card / Replacement Card, showing business meaning, current health, IDSP usage, risks, replacement candidates, and validation suggestions;
    4. Support Ask AI to find substitute, fallback, derived, or lineage-related features and generate an IDSP Strategy Change Proposal. The Agent only diagnoses, generates candidates, and creates proposal. It must not directly modify strategies; every change must be human-confirmed before entering IDSP configuration, validation, rollout, and rollback flow.

- **Related Documents**

    > Current design input: Feature Platform Future Design Blueprint  
    > Current design input: Feature Platform Agentic Workflow - Feature Monitoring → Diagnosis → Replacement → IDSP Handoff  
    > Public wiki references: `wiki/collections/trident-strategy-platform/entities/feature-platform.md`; `wiki/collections/trident-strategy-platform/reports/platform-capability-map.md`; `wiki/collections/trident-strategy-platform/playbooks/how-to-read-metrics-trace-and-data-assets.md`

# Basic Info // 基础信息

> **Change Log // 变更记录**

| **Date // 日期** | **Description // 描述** | **修改人 // by** |
|---|---|---|
| 07/29/2026 | Initial PRD based on Agentic Workflow design input. | Yueming Gao |

> **Relevant Links // 相关链接**

|  | **Links // 链接** | **POC** |
|---|---|---|
| **Link to Meego Ticket** | TBD | PM |
| **Link to Legal Ticket** | N/A for P0, unless strategy handoff copy needs legal review | PM |
| **Link to Figma/Demo** | TBD | UED |
| **Link to Event Tracking** | TBD | DA |
| **Other Useful Links** | Public wiki references listed above | PM / RD |

# Background // 需求背景

## **What are we building? // 需求概述**

1. Problem 1: 当前 Feature Platform 如果只做 feature registry + RAG 问答，价值主要停留在“找特征、理解特征”。但在 IDSP 策略生产链路里，用户真正需要的是：当某个 feature 异常时，平台能帮助用户快速判断异常原因、影响哪些策略、是否需要替换或降级，以及如何进入 IDSP 变更链路。

    Problem 1: If Feature Platform only supports feature registry and RAG Q&A, its value stays at “find and understand features.” In the IDSP strategy production lifecycle, users need the platform to diagnose feature anomalies, identify impacted strategies, decide whether replacement or fallback is needed, and hand off the change to IDSP.

2. Problem 2: 当前 Feature Card 更多是静态解释卡片，缺少当前健康状态、策略使用、trace/metrics 证据和替代候选比较，因此不能支撑用户做恢复决策。

    Problem 2: Current Feature Card is mainly a static explanation card. It lacks current health status, strategy usage, trace/metrics evidence, and replacement candidate comparison, so it cannot support recovery decisions.

3. Problem 3: Ask AI 如果只返回“推荐特征”，容易造成误用。对于异常恢复场景，Ask AI 应该返回候选、fit label、差异分析、验证计划和风险边界，而不是直接替用户做策略决策。

    Problem 3: If Ask AI only returns “recommended features,” users may misuse candidates. For recovery scenarios, Ask AI should return candidates, fit labels, difference analysis, validation plans, and risk boundaries instead of making strategy decisions for users.

4. Problem 4: Feature Platform 与 IDSP 之间需要建立从 read integration 到 proposal integration，再到 human-confirmed write handoff 的分层边界，避免 Agent 直接修改策略或绕过人工确认。

    Problem 4: Feature Platform and IDSP need a layered integration model: read integration, proposal integration, and human-confirmed write handoff. This prevents the Agent from directly modifying strategies or bypassing human review.

## **Goal // 目标 and key metrics**

1. P0 建立最小闭环：Anomaly → Feature Card → Replacement Candidates：

    - 覆盖 4 类监控指标：数据质量、特征服务、策略消费、业务影响；
    - 覆盖 3 种增强卡片模式：Discovery Card、Diagnosis Card、Replacement Card；
    - 覆盖 5 类替代候选等级：Full substitute、Partial substitute、Fallback only、Derived candidate、Not suitable；
    - 100% replacement analysis 必须包含 fit label、difference analysis、risk、validation plan；
    - 异常定位耗时目标：TBD baseline → target with @Platform Oncall / DA。

2. P1 建立 IDSP Proposal：

    - 100% IDSP Strategy Change Proposal 必须包含 impacted strategy、current feature、candidate feature、replacement type、condition change、expected impact、risk、validation plan、rollout plan、rollback plan；
    - 100% IDSP handoff 必须经过 human confirm，不允许 Agent 直接上线策略；
    - IDSP proposal 生成耗时目标：TBD baseline → target with @IDSP owner。

3. P2 建立 Lineage & Derived Feature：

    - 覆盖 8 类血缘关系：derived_from、same_source_as、sibling_of、used_by_strategy、replaces、fallback_for、conflicts_with、deprecated_by；
    - 对无可用替代的 case，支持生成 Derived Feature Proposal，并进入 New Feature Entry + Agent Evaluation；
    - 衍生特征 proposal 的 evaluation pass rate：TBD with @Feature Platform RD。

4. P3 建立 Agentic Recovery Loop：

    - Agent 自动检测异常、自动诊断、自动找候选、自动生成 proposal；
    - rollout 后自动监控效果，失败自动建议 rollback；
    - 所有策略变更仍需要 human confirm，0 自动绕过。

1. P0 builds the minimum loop: Anomaly → Feature Card → Replacement Candidates:

    - Cover 4 monitoring metric categories: data quality, feature service, strategy consumption, and business impact;
    - Cover 3 enhanced card modes: Discovery Card, Diagnosis Card, and Replacement Card;
    - Cover 5 replacement fit levels: Full substitute, Partial substitute, Fallback only, Derived candidate, and Not suitable;
    - 100% replacement analysis must include fit label, difference analysis, risk, and validation plan;
    - Anomaly diagnosis time target: TBD baseline → target with @Platform Oncall / DA.

2. P1 builds IDSP Proposal:

    - 100% IDSP Strategy Change Proposal must include impacted strategy, current feature, candidate feature, replacement type, condition change, expected impact, risk, validation plan, rollout plan, and rollback plan;
    - 100% IDSP handoff must be human-confirmed. The Agent must not directly launch strategy changes;
    - IDSP proposal generation time target: TBD baseline → target with @IDSP owner.

3. P2 builds Lineage & Derived Feature:

    - Cover 8 lineage relationships: derived_from, same_source_as, sibling_of, used_by_strategy, replaces, fallback_for, conflicts_with, deprecated_by;
    - For cases with no available replacement, support Derived Feature Proposal and enter New Feature Entry + Agent Evaluation;
    - Derived proposal evaluation pass rate: TBD with @Feature Platform RD.

4. P3 builds Agentic Recovery Loop:

    - The Agent detects anomalies, diagnoses, finds candidates, and generates proposals automatically;
    - After rollout, the Agent monitors impact and suggests rollback when needed;
    - Every strategy change still requires human confirmation. No auto-bypass.

# Requirement Overall

## Featurelist（分前后端）

| **Platform** | **Menu** | **Feature list** | **优先级** |
|---|---|---|---|
| **Feature Platform** | Feature Monitoring Dashboard<br>特征监控看板 | 1. FE：展示 monitored feature KPI、异常热力图、异常列表和 action 入口。<br>FE: Show monitored feature KPI, anomaly heatmap, anomaly list, and action entry.<br><br>2. Serverend：提供 anomaly summary、severity、impacted scope、metric breakdown。<br>Backend: Provide anomaly summary, severity, impacted scope, and metric breakdown. | P0 |
| **Feature Platform** | Feature Drilldown Page<br>异常下钻页 | 1. FE：点击异常 cell 后展示 anomaly summary、metric breakdown、data source status、strategy usage、trace samples、Ask AI Diagnosis。<br>FE: After users click an anomaly cell, show anomaly summary, metric breakdown, data source status, strategy usage, trace samples, and Ask AI Diagnosis.<br><br>2. Serverend：聚合 metrics / trace / data source / strategy usage，返回结构化诊断上下文。<br>Backend: Aggregate metrics, trace, data source, and strategy usage as structured diagnosis context. | P0 |
| **Feature Platform** | Enhanced Feature Card<br>增强特征卡片 | 1. FE：支持 Discovery Card、Diagnosis Card、Replacement Card 三种模式。<br>FE: Support Discovery Card, Diagnosis Card, and Replacement Card.<br><br>2. Serverend：返回 feature identity、business meaning、data source、current anomaly、IDSP usage、risk、replacement candidates。<br>Backend: Return feature identity, business meaning, data source, current anomaly, IDSP usage, risk, and replacement candidates. | P0 |
| **Feature Platform Ask AI** | Agent Diagnosis Panel<br>Agent 诊断面板 | 1. FE：支持 Explain anomaly、Find replacement feature、Inspect lineage、Generate IDSP proposal 等 action。<br>FE: Support actions including Explain anomaly, Find replacement feature, Inspect lineage, and Generate IDSP proposal.<br><br>2. Serverend：封装 Anomaly Diagnosis Skill、Feature Explanation Skill、Replacement Discovery Skill。<br>Backend: Implement Anomaly Diagnosis Skill, Feature Explanation Skill, and Replacement Discovery Skill. | P0 |
| **Feature Platform Ask AI** | Replacement Discovery<br>替代特征查找 | 1. Serverend：多路召回候选：语义相似、元数据相似、数据源相似、策略使用相似、血缘/衍生关系。<br>Backend: Retrieve candidates through semantic similarity, metadata similarity, data source similarity, strategy usage similarity, and lineage/derivation relation.<br><br>2. FE：按 fit label 展示候选：Full substitute、Partial substitute、Fallback only、Derived candidate、Not suitable。<br>FE: Render candidates by fit label: Full substitute, Partial substitute, Fallback only, Derived candidate, Not suitable. | P0 |
| **Feature Platform + IDSP** | IDSP Handoff Proposal<br>IDSP 策略变更建议 | 1. Serverend：读取 IDSP strategy usage，生成 Strategy Change Proposal。<br>Backend: Read IDSP strategy usage and generate Strategy Change Proposal.<br><br>2. FE：展示 impacted strategy、condition change、validation plan、rollout plan、rollback plan，并要求 human confirm 后进入 IDSP。<br>FE: Show impacted strategy, condition change, validation plan, rollout plan, rollback plan, and require human confirmation before entering IDSP. | P1 |
| **Feature Platform** | Lineage & Derived Feature Explorer<br>血缘与衍生特征探索 | 1. Serverend：建设 same_source_as、derived_from、sibling_of、used_by_strategy 等关系。<br>Backend: Build relationships such as same_source_as, derived_from, sibling_of, and used_by_strategy.<br><br>2. FE：无合适替代时生成 Derived Feature Proposal，并进入 New Feature Entry + Agent Evaluation。<br>FE: When no suitable replacement exists, generate Derived Feature Proposal and enter New Feature Entry + Agent Evaluation. | P2 |
| **Feature Platform + IDSP** | Agentic Recovery Loop<br>Agentic 恢复闭环 | 1. Serverend：自动检测异常、诊断、找候选、生成 proposal，rollout 后监控效果并建议 rollback。<br>Backend: Automatically detect anomaly, diagnose, find candidates, generate proposal, monitor after rollout, and suggest rollback.<br><br>2. FE：展示闭环状态机和人工确认入口。<br>FE: Show recovery state machine and human confirmation entry. | P3 |

## User Story Route

### P0 User Story：Anomaly → Feature Card → Replacement Candidates

用户在 Feature Monitoring Dashboard 看到某个 feature 异常，例如 EEA region 的 enum distribution shift 或 null_rate 升高。用户点击异常 cell 后进入 Feature Drilldown。系统展示异常时间、影响范围、metric breakdown、data source status、IDSP strategy usage 和 trace samples。用户点击 Ask AI Diagnosis，Agent 返回异常摘要、可能原因、证据和下一步操作。用户查看 Diagnosis Feature Card，确认该 feature 的业务含义、数据源、IDSP 使用情况和风险。用户点击 Find replacement feature，Ask AI 返回 replacement candidates，并按 fit label 展示候选差异和验证计划。

The user sees a feature anomaly on Feature Monitoring Dashboard, such as enum distribution shift or null_rate increase in EEA. After clicking the anomaly cell, the user enters Feature Drilldown. The system shows start time, impacted scope, metric breakdown, data source status, IDSP strategy usage, and trace samples. The user clicks Ask AI Diagnosis. The Agent returns diagnosis summary, root cause hypothesis, evidence, and next actions. The user reviews Diagnosis Feature Card to confirm business meaning, data source, IDSP usage, and risk. The user clicks Find replacement feature. Ask AI returns replacement candidates with fit labels, difference analysis, and validation plan.

### P1 User Story：Replacement Candidates → IDSP Strategy Change Proposal

用户选择一个 candidate feature 后，Agent 生成 IDSP Strategy Change Proposal。Proposal 必须说明 impacted strategies、current abnormal feature、candidate feature、replacement type、condition change、expected impact、risk、validation plan、rollout plan 和 rollback plan。用户确认后，系统进入 IDSP 配置链路；Agent 不直接修改或上线策略。

After selecting a candidate feature, the Agent generates an IDSP Strategy Change Proposal. The proposal must include impacted strategies, current abnormal feature, candidate feature, replacement type, condition change, expected impact, risk, validation plan, rollout plan, and rollback plan. After user confirmation, the system enters IDSP configuration flow. The Agent must not directly modify or launch strategies.

### P2 User Story：No Suitable Replacement → Derived Feature Proposal

如果没有可用替代特征，用户进入 Lineage & Derived Feature Explorer。Agent 基于同源、兄弟特征、上游/下游关系生成 Derived Feature Proposal，并说明 derivation logic、data source、usage scenario、expected benefit、risk、validation plan 和 IDSP usage。用户确认后进入 New Feature Entry + Agent Evaluation。

If no suitable replacement exists, the user enters Lineage & Derived Feature Explorer. The Agent generates a Derived Feature Proposal based on same-source, sibling, upstream, and downstream relationships, including derivation logic, data source, usage scenario, expected benefit, risk, validation plan, and IDSP usage. After user confirmation, the flow enters New Feature Entry + Agent Evaluation.

# Feature Details - Page

## Feature 1: Feature Monitoring Dashboard

| **Demo** | **Description** |  |
|---|---|---|
| TBD | 1. 页面目标：让用户快速发现 feature 健康问题。<br><br>2. 顶部 KPI：Total monitored features、Healthy features、Warning features、Critical features、Impacted strategies、Data source incidents。<br><br>3. 中间热力图：x-axis 可按 feature / business domain / data source 展示，y-axis 可按 region / entity / time bucket 展示，cell color 表示 health score / anomaly severity。<br><br>4. 右侧异常列表：feature code、anomaly type、severity、impacted region、impacted strategy count、start time、action。 | 1. Page goal: help users quickly discover feature health issues.<br><br>2. Top KPI: Total monitored features, Healthy features, Warning features, Critical features, Impacted strategies, Data source incidents.<br><br>3. Heatmap: x-axis can show feature / business domain / data source; y-axis can show region / entity / time bucket; cell color indicates health score / anomaly severity.<br><br>4. Anomaly list: feature code, anomaly type, severity, impacted region, impacted strategy count, start time, action. |

### Monitoring Metrics

| **Metric Category** | **Metrics** | **Description // 说明** |
|---|---|---|
| Data Quality | null_rate, missing_rate, invalid_value_rate, enum_unseen_value_rate, distribution_shift_score, freshness_delay, coverage_rate | Detect null, missing, invalid value, unknown enum, distribution shift, freshness delay, and coverage issue. |
| Feature Service | availability, latency, timeout_rate, error_rate, sync_status | Detect service availability, latency, timeout, error, and sync status issue. |
| Strategy Consumption | strategy_usage_count, hit_rate, condition_match_rate, decision_change_rate, fallback_rate | Detect whether the anomaly impacts IDSP strategy matching and decision result. |
| Business Impact | impacted_entity_count, impacted_region_count, impacted_strategy_count, business_metric_delta | Detect business impact scope and metric delta. |

## Feature 2: Feature Drilldown Page

| **Demo** | **Description** |  |
|---|---|---|
| TBD | 1. Header：展示 feature code、anomaly severity、anomaly type、impacted scope、start time。<br><br>2. Anomaly Summary：展示异常摘要、同比/环比变化、影响范围。<br><br>3. Metric Breakdown：支持按 region / entity / source / strategy 下钻。<br><br>4. Data Source Status：展示数据源是否延迟、是否同步失败、是否覆盖下降。<br><br>5. Strategy Usage：展示哪些 IDSP 策略使用该 feature、哪些策略受影响、当前配置条件是什么。<br><br>6. Trace Samples：展示异常样本、正常样本和差异解释。<br><br>7. Ask AI Diagnosis：提供 Explain anomaly、Find replacement feature、Generate IDSP proposal action。 | 1. Header: show feature code, anomaly severity, anomaly type, impacted scope, and start time.<br><br>2. Anomaly Summary: show anomaly summary, WoW/DoD delta, and impacted scope.<br><br>3. Metric Breakdown: support drilldown by region / entity / source / strategy.<br><br>4. Data Source Status: show source delay, sync failure, and coverage drop.<br><br>5. Strategy Usage: show which IDSP strategies use this feature, which are impacted, and the current conditions.<br><br>6. Trace Samples: show abnormal samples, normal samples, and difference explanation.<br><br>7. Ask AI Diagnosis: provide Explain anomaly, Find replacement feature, and Generate IDSP proposal actions. |

## Feature 3: Agent Diagnosis Panel

| **Demo** | **Description** |  |
|---|---|---|
| TBD | 1. 用户可以问：“这个 feature 为什么异常？”、“影响了哪些 IDSP 策略？”、“有没有同类 feature 可以替代？”、“能不能找一个 fallback feature？”、“有没有血缘相关 feature 可以衍生？”、“帮我生成一个 IDSP 替换方案。”<br><br>2. Agent 返回结构：Diagnosis Summary、Root Cause Hypothesis、Evidence、Next Actions。<br><br>3. Root Cause Hypothesis 包括：data source issue、feature logic issue、enum mapping issue、traffic mix shift、strategy config issue、monitoring false positive。<br><br>4. Evidence 必须引用 metrics、trace samples、data source status、strategy usage，不能只给自然语言判断。 | 1. Users can ask: “Why is this feature abnormal?”, “Which IDSP strategies are impacted?”, “Is there a similar replacement?”, “Can we find a fallback feature?”, “Can lineage-related features derive a new one?”, “Generate an IDSP replacement plan.”<br><br>2. Agent response structure: Diagnosis Summary, Root Cause Hypothesis, Evidence, Next Actions.<br><br>3. Root Cause Hypothesis includes data source issue, feature logic issue, enum mapping issue, traffic mix shift, strategy config issue, and monitoring false positive.<br><br>4. Evidence must cite metrics, trace samples, data source status, and strategy usage. Natural-language judgment alone is not enough. |

## Feature 4: Enhanced Feature Card

| **Card Type** | **Description // 描述** | **Fields // 字段** |
|---|---|---|
| Discovery Card | 用于找特征。<br>Used for feature discovery. | feature name/code, why matched, fit label, scope, data source, usage example, risk / limitation, output samples, View detail to confirm |
| Diagnosis Card | 用于解释异常。<br>Used for anomaly diagnosis. | Feature Identity, Business Meaning, Data Source, Current Anomaly, IDSP Usage, Risk, Trace / Metrics, Actions |
| Replacement Card | 用于替代候选比较。<br>Used for replacement candidate comparison. | Candidate Feature, Why matched, Difference, Fit label, Validation, Add to IDSP proposal |

### Replacement Candidate Fit Label

| **Fit Label** | **Meaning // 含义** | **IDSP Handoff // 是否可推给 IDSP** |
|---|---|---|
| Full substitute | 业务含义、entity、数据源、覆盖范围高度一致。 | 可以生成替换 proposal。 |
| Partial substitute | 只覆盖部分场景。 | 只能做部分策略替换。 |
| Fallback only | 不适合长期替代，但可临时降级。 | 可生成 fallback proposal。 |
| Derived candidate | 需要基于现有 feature 派生。 | 进入 new feature proposal。 |
| Not suitable | 风险过高或口径不一致。 | 不推送。 |

## Feature 5: Lineage & Derived Feature Explorer

| **Demo** | **Description** |  |
|---|---|---|
| TBD | 1. 该模块作为未来新增能力设计，不声称当前 Feature Platform 已有完整血缘 schema。<br><br>2. 页面需要回答：这个 feature 的上游数据源是什么？是否有同源 feature？是否有同 entity、同 business domain 的相邻 feature？是否能通过已有 feature 派生一个新 feature？派生 feature 是否能被 IDSP 消费？<br><br>3. 如果没有合适替代特征，Agent 生成 Derived Feature Proposal，并进入 New Feature Entry + Agent Evaluation。 | 1. This is designed as a future new capability. We should not claim that Feature Platform already has a complete lineage schema today.<br><br>2. The page answers: What is the upstream data source? Are there same-source features? Are there sibling features with the same entity and business domain? Can a new feature be derived from existing ones? Can the derived feature be consumed by IDSP?<br><br>3. If no suitable replacement exists, the Agent generates a Derived Feature Proposal and enters New Feature Entry + Agent Evaluation. |

# Feature Details - no Page

## Backend Logic 1: Replacement Discovery

Replacement discovery uses multi-route retrieval and ranking, not pure vector similarity.

替代特征推荐不是单纯向量相似，而是多路召回 + 排序。

| **Route // 召回来源** | **Fields // 依据** | **Purpose // 作用** |
|---|---|---|
| Semantic Similarity | feature meaning, search summary, aliases, common questions, usage example | Find features with similar business meaning. |
| Metadata Similarity | same entity, same feature type, same data type, same business domain, same config mode | Filter out unusable candidates. |
| Data Source Similarity | same source, same upstream table / stream, same freshness, same enum dictionary | Find same-source replacement or same-source fallback. |
| Strategy Usage Similarity | same strategy type, same business domain, same IDSP config pattern | Find features that are replaceable in strategy configuration. |
| Lineage / Derivation | upstream sibling, downstream derived, same parent source, can derive | Find derivation options. |

Suggested product ranking logic:

```text
replacement_score =
  semantic_similarity * 0.25
+ entity_match * 0.15
+ data_source_match * 0.15
+ region_coverage_match * 0.15
+ strategy_usage_match * 0.15
+ health_score * 0.10
+ risk_penalty * -0.10
```

This is a product ranking suggestion, not a hard implementation requirement.

## Backend Logic 2: Agent Skill Split

| **Skill** | **Input** | **Output** |
|---|---|---|
| Anomaly Diagnosis Skill | feature code, anomaly metric, time window, region/entity, heatmap cell context | anomaly summary, impacted scope, root cause hypothesis, evidence, next actions |
| Feature Explanation Skill | feature code | feature meaning, data source, usage scenario, risk, IDSP usage, output samples |
| Replacement Discovery Skill | abnormal feature, scenario, region, entity, IDSP strategy context | candidate replacement features, fit label, difference analysis, validation plan |
| Lineage Derivation Skill | abnormal feature, no suitable replacement reason | upstream/downstream related features, same-source features, derived feature proposal, risk and validation plan |
| IDSP Proposal Skill | impacted strategy, abnormal feature, candidate replacement, validation evidence | IDSP change proposal, replacement logic, fallback logic, rollout plan, rollback plan |

## Backend Logic 3: IDSP Integration

| **Integration Layer** | **Capability** | **Description // 说明** |
|---|---|---|
| Read Integration | Read IDSP strategy usage | Feature Platform needs to know which strategies use the feature, which condition uses it, strategy status, hit rate, recent publish/rollback, and trace samples. |
| Proposal Integration | Generate IDSP Strategy Change Proposal | Proposal types include Replace Feature, Add Fallback, Adjust Threshold, Disable Condition, Add Derived Feature, Need Human Review. |
| Write Integration | Human-confirmed IDSP handoff | Agent proposal → PM/RD review → IDSP strategy change → offline validation / shadow test → experiment / gray rollout → monitoring → full rollout or rollback. |

## Backend Logic 4: State Machine

| **State Machine** | **Flow** |
|---|---|
| Anomaly | DETECTED → DIAGNOSING → ROOT_CAUSE_HYPOTHESIS → REPLACEMENT_SEARCHING → PROPOSAL_GENERATED → HUMAN_REVIEW → IDSP_DRAFT_CREATED → VALIDATING → ROLLED_OUT / ROLLED_BACK / CLOSED |
| Replacement | CANDIDATE_FOUND → FIT_ANALYZED → VALIDATION_REQUIRED → APPROVED_FOR_PROPOSAL → PUSHED_TO_IDSP |
| Derived Feature | DERIVATION_NEEDED → DERIVED_PROPOSAL_GENERATED → EVALUATION_REQUIRED → FEATURE_CREATED → INDEXED → AVAILABLE_FOR_IDSP |

## Backend Logic 5: Strategy Change Proposal Contract

| **Field** | **Description // 说明** |
|---|---|
| impacted_strategy | 受影响策略 |
| current_feature | 当前异常 feature |
| anomaly_summary | 异常摘要 |
| candidate_feature | 替代候选 |
| replacement_type | full replacement / partial replacement / fallback / derived feature |
| condition_change | IDSP 条件变更建议 |
| expected_impact | 预期影响 |
| risk | 风险 |
| validation_plan | 验证方案 |
| rollout_plan | 灰度 / 实验方案 |
| rollback_plan | 回滚方案 |
| human_confirmation | 确认人、确认时间、进入 IDSP 配置链路的入口 |

## Error / Guardrail Framework

### Case 1: LOW_CONFIDENCE_REPLACEMENT

| **Code** | **Meaning** | **Trigger** | **UX** |
|---|---|---|---|
| LOW_CONFIDENCE_REPLACEMENT | 替代候选分数低或证据不足。 | top replacement score below threshold, or evidence missing. | Candidate card is greyed out; show “Need more validation”; require validation plan before proposal. |

### Case 2: SOURCE_SHARED_RISK

| **Code** | **Meaning** | **Trigger** | **UX** |
|---|---|---|---|
| SOURCE_SHARED_RISK | 替代候选依赖同一个异常 source。 | candidate source equals abnormal feature source, and source incident exists. | Inline warning on Replacement Card; candidate cannot be labeled Full substitute unless risk is explicitly acknowledged. |

### Case 3: IDSP_USAGE_CONTEXT_MISSING

| **Code** | **Meaning** | **Trigger** | **UX** |
|---|---|---|---|
| IDSP_USAGE_CONTEXT_MISSING | 缺少 IDSP 策略使用上下文。 | impacted_strategy or condition_change cannot be resolved. | Block IDSP proposal generation; show missing usage context and ask user to manually review in IDSP. |

### Case 4: VALIDATION_PLAN_MISSING

| **Code** | **Meaning** | **Trigger** | **UX** |
|---|---|---|---|
| VALIDATION_PLAN_MISSING | 缺少验证计划。 | proposal has no offline validation, shadow test, experiment, or monitoring metric. | Disable handoff action until validation plan is added. |

### Case 5: ROLLBACK_PLAN_MISSING

| **Code** | **Meaning** | **Trigger** | **UX** |
|---|---|---|---|
| ROLLBACK_PLAN_MISSING | 缺少回滚方案。 | proposal has no rollback condition, action, owner, or monitoring metric. | Disable handoff action until rollback plan is added. |

### Case 6: HUMAN_CONFIRM_REQUIRED

| **Code** | **Meaning** | **Trigger** | **UX** |
|---|---|---|---|
| HUMAN_CONFIRM_REQUIRED | Agent 不能直接修改或上线策略。 | user attempts to push proposal without review / confirmation. | Show confirmation modal; require PM/RD owner confirmation before entering IDSP configuration flow. |

## Guardrails

### Replacement Recommendation Guardrails

| **Risk // 风险** | **Rule // 规则** |
|---|---|
| 错误替代 | 不输出“直接推荐”，只输出候选和 fit label。 |
| 口径不一致 | 必须展示差异：entity、region、data source、logic、enum。 |
| 低置信误导 | 低置信候选置灰。 |
| 数据源同故障 | 替代候选不能依赖同一个异常 source，除非明确说明风险。 |
| 不适合策略 | 必须检查 IDSP usage context。 |
| 无验证上线 | 必须生成 validation plan。 |
| Agent 自动改策略 | 禁止，必须 human confirm。 |

### IDSP Handoff Guardrails

| **Risk // 风险** | **Rule // 规则** |
|---|---|
| 直接上线 | Agent 只能创建 proposal / IDSP change entry，不直接上线。 |
| 影响范围不清 | Proposal 必须列出 impacted strategy。 |
| 无回滚 | Proposal 必须包含 rollback plan。 |
| 无监控 | Proposal 必须包含 rollout monitoring metrics。 |
| 无证据 | Proposal 必须引用 metrics / trace / Feature Card evidence。 |
| 衍生特征不成熟 | Derived feature 必须先过 evaluation 和 indexing。 |

# MVP & Roadmap

| **Phase** | **Scope // 范围** | **Not in Scope // 不做** |
|---|---|---|
| P0: Anomaly → Feature Card → Replacement Candidates | Feature Monitoring 异常列表 / 热力图；点击异常进入 Drilldown；展示增强 Feature Card；Ask AI 找替代候选；生成 Replacement Analysis；手动复制或跳转 IDSP。 | 不自动创建 IDSP change；不自动派生 feature；不自动上线；不做全量血缘图谱。 |
| P1: IDSP Proposal | 读取 IDSP strategy usage；展示 impacted strategies；生成 IDSP Strategy Change Proposal；支持用户确认后进入 IDSP change flow；增加 validation plan 和 rollback plan。 | 不跳过 owner review；不自动 rollout。 |
| P2: Lineage & Derived Feature | 建设 feature lineage explorer；支持 same-source / derived-from / sibling feature；无替代时生成 derived feature proposal；进入 New Feature Entry + Agent Evaluation。 | 不声称现阶段已有完整血缘 schema。 |
| P3: Agentic Recovery Loop | Agent 自动检测异常、自动诊断、自动找候选、自动生成 proposal；人确认后进入 IDSP；rollout 后自动监控效果，失败自动建议 rollback。 | Agent 仍不直接修改策略或绕过人工确认。 |

# Appendix // 附录

## End-to-end Example

Case：某 room-level enum feature 在 EEA 异常。

1. Monitoring heatmap 显示：feature = `room_type_feature`，region = EEA，anomaly = enum distribution shift，severity = high。
2. 用户点击异常 cell。
3. Drilldown 显示：001 占比从 40% 降到 5%，unknown value 占比升高，影响 12 个 IDSP 策略，数据源最近有延迟。
4. Agent Diagnosis 判断可能原因：enum mapping 变更、upstream source 延迟、EEA traffic mix shift。
5. Feature Card 展示 `room_type_feature` 的业务含义、output samples、data source、IDSP usage。
6. 用户点击 Find replacement。
7. Ask AI 返回候选：
    - `room_category_v2`: likely substitute, same entity, better EEA coverage, enum mapping 不完全一致，需要验证；
    - `room_vertical_tag`: partial substitute, 可用于部分策略，不适合 enforcement strict condition；
    - `room_content_model_score`: fallback only, 非枚举，但可以作为临时风险判断。
8. 用户选择 `room_category_v2`。
9. Agent 生成 IDSP Proposal：impacted strategies = 12；replace `room_type_feature` with `room_category_v2` only for EEA；run shadow test for 3 days；monitor hit rate and decision change；rollback if delta > threshold。
10. PM/RD 确认后进入 IDSP configuration flow。

## Product Positioning

方案最该强调的 4 个升级：

1. 从“找特征”升级为“特征恢复链路”：异常发现 → 诊断 → 替代 → IDSP 恢复。
2. 从 Feature Card 升级为 Decision Card：解释为什么异常、影响哪里、能不能替代、怎么替代、替代风险是什么。
3. 从 RAG 问答升级为 Agent Skills：diagnosis、explanation、replacement、lineage、IDSP proposal。
4. 从平台内闭环升级为 IDSP 策略闭环：Feature health issue → Feature intelligence → IDSP strategy proposal → validation → rollout / rollback。
