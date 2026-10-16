# GCP Agentic KPI Data Model Refactoring Guide

## Purpose

This document provides the implementation specification for refactoring the GCP Agentic KPI project in BigQuery.

It is written for an engineering agent that will inspect and update the repository. The agent must use this document as the target architecture and must preserve existing business logic unless a change is explicitly identified, documented, tested, and approved.

The intended outcome is a KPI platform that:

- Establishes a governed single source of truth for KPI values and definitions.
- Supports more than 50 KPIs without a single unmanageable stored procedure.
- Serves visualisation tools and conversational analytics agents.
- Separates atomic facts, business definitions, targets, performance rules, and derived presentation logic.
- Is performant, auditable, scalable, and easy to extend.
- Preserves historical definitions and calculation traceability.
- Provides machine-readable metadata to reduce incorrect AI interpretation.

---

## Current assets

The current implementation contains two principal tables:

1. `dp_707_cons_cmm_adp_kpi_definitions`
2. `dp_707_cons_cmm_adp_mw_kpi_daily_metrics`

The definitions table contains valuable business and technical metadata, including KPI names, definitions, business use cases, calculation logic, source tables, ownership, applicability, grain, unit, thresholds, freshness, version, and lifecycle attributes.

The daily metrics table currently mixes:

- Metric identity and values.
- Descriptive KPI metadata.
- Targets.
- RAG status.
- Week-on-week, quarter-on-quarter, and year-on-year comparisons.
- Pipeline audit information.

This mixture duplicates metadata across fact rows, increases maintenance effort, widens the table, and causes the calculation stored procedure to grow as KPIs and comparison requirements are added.

---

## Required design principles

The repository changes must follow these principles:

1. **Store facts once.**
   Atomic KPI observations belong in a narrow fact table.

2. **Store business meaning once.**
   KPI definitions and semantic behaviour belong in governed metadata tables.

3. **Keep grain-defining dimensions in the fact.**
   Do not reduce the metric table to only `kpi_id` and `metric_value`. A row must still identify its business date and dimensional grain.

4. **Do not duplicate descriptive metadata in every metric row.**
   KPI name, display name, unit, owner, definition, thresholds, and calculation descriptions must not be repeated in the core metric fact.

5. **Separate actuals from targets.**
   Targets have their own scenarios, versions, effective periods, and ownership.

6. **Calculate presentation attributes in serving objects.**
   RAG status, direction labels, target status, and period-comparison labels should normally be derived in views or curated serving tables.

7. **Version business definitions.**
   Historical metric values must remain linked to the definition version under which they were calculated.

8. **Use modular SQL with a common output contract.**
   KPI calculations must be separated by business domain or reusable source base rather than maintained in one large procedure.

9. **Optimise source scans.**
   KPIs derived from the same source and grain should share a filtered and aggregated base query where practical.

10. **Treat metadata as both human-readable and machine-readable.**
    AI agents must not be expected to infer aggregation behaviour, polarity, allowed dimensions, or comparison policy solely from prose.

---

## Target architecture

```text
Golden BigQuery data products
            |
            v
Reusable domain/source base models
            |
            v
Modular KPI calculation models
            |
            v
Staging and data-quality validation
            |
            v
kpi_metric_value
     |              |               |
     v              v               v
kpi_definition  kpi_target  kpi_performance_rule
     \              |               /
      \             |              /
       v            v             v
         Governed serving views
                  |
          +-------+-------+
          |               |
          v               v
     BI tools         AI agents
```

---

## Target BigQuery objects

### Core tables

- `kpi_definition`
- `kpi_metric_value`
- `kpi_target`
- `kpi_performance_rule`

### Governance and relationship tables

- `kpi_source_lineage`
- `kpi_relationship`
- `kpi_dimension_policy`
- `kpi_brand_applicability`

### Serving objects

- `vw_kpi_catalog`
- `vw_kpi_metrics_enriched`
- `vw_kpi_period_comparison`
- `vw_kpi_data_quality_status`

### Pipeline and audit objects

- `stg_kpi_metric_value`
- `calculation_run_audit`
- `data_quality_result`

The repository agent may adapt physical names to existing naming standards, but it must preserve the logical separation described above.

---

# 1. KPI definition table

## Responsibility

`kpi_definition` is the canonical KPI contract and business glossary. It contains descriptive, semantic, governance, and version information. It must not contain metric observations.

## Recommended schema

```sql
CREATE TABLE IF NOT EXISTS `${PROJECT_ID}.${DATASET_ID}.kpi_definition` (
  kpi_id INT64 NOT NULL,
  kpi_code STRING NOT NULL,
  display_name STRING NOT NULL,

  definition STRING NOT NULL,
  business_use_case STRING,
  calculation_description STRING NOT NULL,

  service_code STRING,
  default_unit_code STRING NOT NULL,
  default_granularity STRING NOT NULL,
  aggregation_type STRING NOT NULL,
  performance_direction STRING NOT NULL,

  owner_group STRING,
  business_owner STRING,
  technical_owner STRING,
  approval_group STRING,

  freshness_sla_hours INT64,
  certification_status STRING,
  data_classification STRING,

  synonyms ARRAY<STRING>,
  allowed_dimensions ARRAY<STRING>,
  prohibited_aggregations ARRAY<STRING>,
  interpretation_guidance STRING,
  known_limitations STRING,

  effective_from DATE NOT NULL,
  effective_to DATE,
  definition_version INT64 NOT NULL,
  is_current BOOL NOT NULL,
  is_active BOOL NOT NULL,

  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL,

  PRIMARY KEY (kpi_id, definition_version) NOT ENFORCED
);
```

## Required semantics

### `kpi_id`

Stable numeric identifier. Never reuse an identifier for a different KPI.

### `kpi_code`

Stable machine-readable business key, for example:

```text
NEW_CUSTOMER_VOLUME
CUSTOMER_CHURN_RATE
NET_ADDS
```

Do not use a display label as an integration key.

### `aggregation_type`

Constrain to approved values such as:

```text
SUM
AVERAGE
WEIGHTED_AVERAGE
RATIO
LAST_VALUE
NON_ADDITIVE
```

This field tells BI tools and AI agents how values may be aggregated.

### `performance_direction`

Constrain to approved values such as:

```text
HIGHER_IS_BETTER
LOWER_IS_BETTER
TARGET_RANGE
INFORMATIONAL
```

This replaces KPI-specific polarity logic embedded throughout procedures.

### Definition versioning

A material business-definition or calculation change must create a new `definition_version`. Use:

- `effective_from`
- `effective_to`
- `definition_version`
- `is_current`
- `is_active`

Historical facts must retain the version used when they were calculated.

### Ownership

Prefer a durable `owner_group` or `approval_group` as the primary accountable entity. Named individuals may be retained as operational contacts, but they must not be the only ownership mechanism.

---

# 2. KPI metric fact table

## Responsibility

`kpi_metric_value` stores atomic KPI observations at an explicitly declared grain. It holds metric values, dimensional keys, definition version, freshness, and calculation audit fields.

It must not contain repeated descriptive metadata or target scenarios.

## Recommended schema

```sql
CREATE TABLE IF NOT EXISTS `${PROJECT_ID}.${DATASET_ID}.kpi_metric_value` (
  metric_date DATE NOT NULL,
  kpi_id INT64 NOT NULL,
  definition_version INT64 NOT NULL,

  brand_code STRING NOT NULL,
  service_code STRING,
  channel_code STRING,
  market_segment_code STRING,

  metric_value NUMERIC,
  numerator_value NUMERIC,
  denominator_value NUMERIC,

  data_as_of_date DATE,
  calculated_at TIMESTAMP NOT NULL,
  calculation_run_id STRING NOT NULL,

  PRIMARY KEY (
    metric_date,
    kpi_id,
    definition_version,
    brand_code,
    service_code,
    channel_code,
    market_segment_code
  ) NOT ENFORCED,

  FOREIGN KEY (kpi_id, definition_version)
    REFERENCES `${PROJECT_ID}.${DATASET_ID}.kpi_definition`
      (kpi_id, definition_version)
    NOT ENFORCED
)
PARTITION BY metric_date
CLUSTER BY kpi_id, brand_code, service_code;
```

## Grain

The expected grain is one row per unique combination of:

```text
metric_date
kpi_id
definition_version
brand_code
service_code
channel_code
market_segment_code
```

If a dimension is not applicable, the project must use one consistent convention. Do not mix nulls, blanks, `ALL`, and `UNKNOWN` without a defined semantic policy.

## Date and audit semantics

Separate the following concepts:

- `metric_date`: business date represented by the KPI.
- `data_as_of_date`: latest source-data date incorporated into the result.
- `calculated_at`: timestamp at which the value was calculated.
- `calculation_run_id`: identifier linking the row to an auditable pipeline run.

Do not use pipeline execution date as a substitute for metric business date.

## Numeric type

Use `NUMERIC` for counts, currency, rates, targets, and comparison calculations unless a documented precision or range requirement requires another type.

## Numerator and denominator

Populate `numerator_value` and `denominator_value` for rates and ratios where possible. This improves reconciliation, explainability, and weighted aggregation. They may be null for metrics where they are not applicable.

---

# 3. Target table

## Responsibility

`kpi_target` stores target values independently from actual metric values. It supports multiple target scenarios and versions.

## Recommended schema

```sql
CREATE TABLE IF NOT EXISTS `${PROJECT_ID}.${DATASET_ID}.kpi_target` (
  target_date DATE NOT NULL,
  kpi_id INT64 NOT NULL,
  brand_code STRING NOT NULL,
  service_code STRING,

  target_scenario STRING NOT NULL,
  target_value NUMERIC NOT NULL,

  effective_from DATE NOT NULL,
  effective_to DATE,
  target_version INT64 NOT NULL,
  is_current BOOL NOT NULL,

  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
)
PARTITION BY target_date
CLUSTER BY kpi_id, brand_code, target_scenario;
```

Example target scenarios include:

```text
QXP
MTP
BUDGET
FORECAST
STRETCH
```

Do not store only one `tgt_value` and `tgt_type` on each actual metric row.

---

# 4. Performance rule table

## Responsibility

`kpi_performance_rule` contains effective-dated rules for converting actual or target-relative performance into RAG status.

## Recommended schema

```sql
CREATE TABLE IF NOT EXISTS `${PROJECT_ID}.${DATASET_ID}.kpi_performance_rule` (
  kpi_id INT64 NOT NULL,
  rule_version INT64 NOT NULL,

  comparison_basis STRING NOT NULL,
  performance_direction STRING NOT NULL,
  amber_boundary NUMERIC,
  red_boundary NUMERIC,

  effective_from DATE NOT NULL,
  effective_to DATE,
  is_current BOOL NOT NULL,

  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL,

  PRIMARY KEY (kpi_id, rule_version) NOT ENFORCED
);
```

Approved `comparison_basis` values may include:

```text
TARGET_RATIO
ABSOLUTE_VALUE
PERIOD_CHANGE
PERCENTAGE_POINT_CHANGE
```

Do not assume every KPI can be classified using `actual_value / target_value`.

---

# 5. Normalise multi-value and relationship metadata

## Source lineage

Replace a delimited `source_tables` string with:

```sql
CREATE TABLE IF NOT EXISTS `${PROJECT_ID}.${DATASET_ID}.kpi_source_lineage` (
  kpi_id INT64 NOT NULL,
  definition_version INT64 NOT NULL,
  source_project STRING NOT NULL,
  source_dataset STRING NOT NULL,
  source_table STRING NOT NULL,
  source_column STRING,
  lineage_role STRING NOT NULL
);
```

Suggested `lineage_role` values:

```text
PRIMARY_SOURCE
FILTER_SOURCE
LOOKUP_SOURCE
TARGET_SOURCE
```

## KPI relationships

Replace a single or delimited `related_kpi_ids` field with:

```sql
CREATE TABLE IF NOT EXISTS `${PROJECT_ID}.${DATASET_ID}.kpi_relationship` (
  from_kpi_id INT64 NOT NULL,
  to_kpi_id INT64 NOT NULL,
  relationship_type STRING NOT NULL,
  relationship_description STRING
);
```

Suggested relationship types:

```text
COMPONENT_OF
DRIVER_OF
CORRELATED_WITH
REPLACES
SUPERSEDES
```

## Brand applicability

If applicability requires governance or effective dating, use a bridge table rather than a delimited string:

```sql
CREATE TABLE IF NOT EXISTS `${PROJECT_ID}.${DATASET_ID}.kpi_brand_applicability` (
  kpi_id INT64 NOT NULL,
  definition_version INT64 NOT NULL,
  brand_code STRING NOT NULL,
  effective_from DATE NOT NULL,
  effective_to DATE
);
```

---

# 6. Serving views

## `vw_kpi_catalog`

Expose currently approved and active KPI definitions for discovery by people, BI tools, and AI agents.

The view should include:

- KPI identifiers and labels.
- Definition and business purpose.
- Aggregation type.
- Performance direction.
- Unit and default grain.
- Ownership and certification.
- Synonyms.
- Allowed dimensions.
- Prohibited aggregations.
- Interpretation guidance.
- Known limitations.
- Freshness SLA.

Exclude expired definition versions unless the consumer explicitly queries a history object.

## `vw_kpi_metrics_enriched`

Create a consumer-friendly view that joins facts, definitions, targets, and performance rules.

Illustrative implementation:

```sql
CREATE OR REPLACE VIEW `${PROJECT_ID}.${DATASET_ID}.vw_kpi_metrics_enriched` AS
SELECT
  m.metric_date,
  m.kpi_id,
  m.definition_version,
  d.kpi_code,
  d.display_name,
  d.definition,
  d.business_use_case,

  m.brand_code,
  m.service_code,
  m.channel_code,
  m.market_segment_code,

  m.metric_value,
  m.numerator_value,
  m.denominator_value,
  d.default_unit_code AS unit_code,
  d.aggregation_type,
  d.performance_direction,

  m.data_as_of_date,
  m.calculated_at,
  m.calculation_run_id,

  t.target_scenario,
  t.target_value,
  SAFE_DIVIDE(m.metric_value, t.target_value) AS target_attainment_ratio,

  CASE
    WHEN t.target_value IS NULL THEN 'NOT_APPLICABLE'
    WHEN d.performance_direction = 'HIGHER_IS_BETTER'
      AND m.metric_value >= t.target_value
      THEN 'ON_OR_ABOVE_TARGET'
    WHEN d.performance_direction = 'LOWER_IS_BETTER'
      AND m.metric_value <= t.target_value
      THEN 'ON_OR_ABOVE_TARGET'
    ELSE 'BELOW_TARGET'
  END AS target_status

FROM `${PROJECT_ID}.${DATASET_ID}.kpi_metric_value` AS m
JOIN `${PROJECT_ID}.${DATASET_ID}.kpi_definition` AS d
  ON m.kpi_id = d.kpi_id
 AND m.definition_version = d.definition_version
LEFT JOIN `${PROJECT_ID}.${DATASET_ID}.kpi_target` AS t
  ON m.metric_date = t.target_date
 AND m.kpi_id = t.kpi_id
 AND m.brand_code = t.brand_code
 AND COALESCE(m.service_code, '') = COALESCE(t.service_code, '');
```

The final implementation must also apply the applicable effective-dated target and performance-rule version. The agent must not copy the illustrative join without checking date-range and scenario requirements.

---

# 7. Period comparisons

## Required change

Remove the following types of fields from the atomic fact table:

```text
wow_value
wow_direction
wow_status
qoq_value
qoq_direction
qoq_status
yoy_value
yoy_direction
yoy_status
```

Create a long-format comparison view or curated comparison table with fields such as:

```text
metric_date
kpi_id
definition_version
brand_code
service_code
comparison_type
comparison_date
current_value
comparison_value
absolute_change
percentage_change
movement_direction
business_movement_status
```

Example rows:

```text
2026-09-24 | 101 | 3 | EE | MOBILE | WOW | 2026-09-17 | 1200 | 1150 | 50
2026-09-24 | 101 | 3 | EE | MOBILE | YOY | 2025-09-24 | 1200 | 1080 | 120
```

This structure must support additional comparison types without adding columns to the fact table.

## Comparison policy

Do not automatically equate:

- Quarter-on-quarter with a fixed 90-day offset.
- Year-on-year with a fixed 365-day offset.

Each comparison must follow an explicitly governed policy, for example:

```text
SAME_CALENDAR_DATE
SAME_FISCAL_PERIOD
SAME_TRADING_WEEK
SAME_WEEKDAY
PREVIOUS_COMPLETE_QUARTER
ROLLING_PERIOD
```

Record the comparison policy in machine-readable metadata.

---

# 8. Stored procedure and SQL modularisation

## Required repository structure

Adapt paths to the repository's conventions while preserving modular separation.

```text
sql/
  definitions/
    create_kpi_definition.sql
    seed_kpi_definitions.sql

  facts/
    create_kpi_metric_value.sql
    merge_kpi_metric_value.sql

  targets/
    create_kpi_target.sql

  rules/
    create_kpi_performance_rule.sql

  kpis/
    customer/
      calculate_new_customer_volume.sql
      calculate_churn_rate.sql
    revenue/
      calculate_arpu.sql
      calculate_total_revenue.sql
    experience/
      calculate_complaint_rate.sql

  common/
    build_customer_daily_base.sql
    build_revenue_daily_base.sql
    calculate_period_comparisons.sql
    attach_targets.sql

  serving/
    create_vw_kpi_catalog.sql
    create_vw_kpi_metrics_enriched.sql
    create_vw_kpi_period_comparison.sql
    create_vw_kpi_data_quality_status.sql

  orchestration/
    load_daily_kpis.sql

  tests/
    test_metric_uniqueness.sql
    test_definition_referential_integrity.sql
    test_definition_version_overlap.sql
    test_valid_metric_ranges.sql
    test_inactive_kpi_loads.sql
    test_grain_conformance.sql
```

## Common KPI output contract

Every KPI calculation module must emit the same column contract and compatible types:

```sql
SELECT
  metric_date,
  kpi_id,
  definition_version,
  brand_code,
  service_code,
  channel_code,
  market_segment_code,
  metric_value,
  numerator_value,
  denominator_value,
  data_as_of_date
FROM ...
```

The orchestration layer must:

1. Execute or compose the KPI modules.
2. Write results to `stg_kpi_metric_value`.
3. Run data-quality checks.
4. Fail safely if blocking checks fail.
5. `MERGE` validated rows into `kpi_metric_value`.
6. Record the run in `calculation_run_audit`.

## Reusable source bases

Where several KPIs share a source and grain, scan and aggregate that source once where practical.

Illustrative pattern:

```sql
WITH customer_daily_base AS (
  SELECT
    activity_date,
    brand_code,
    service_code,
    COUNTIF(is_new_customer) AS new_customer_volume,
    COUNTIF(is_churned_customer) AS churn_volume,
    COUNT(*) AS active_customer_base
  FROM `${PROJECT_ID}.${GOLDEN_DATASET}.customer_activity`
  WHERE activity_date = p_metric_date
  GROUP BY 1, 2, 3
)

SELECT
  activity_date AS metric_date,
  101 AS kpi_id,
  1 AS definition_version,
  brand_code,
  service_code,
  CAST(NULL AS STRING) AS channel_code,
  CAST(NULL AS STRING) AS market_segment_code,
  CAST(new_customer_volume AS NUMERIC) AS metric_value,
  CAST(NULL AS NUMERIC) AS numerator_value,
  CAST(NULL AS NUMERIC) AS denominator_value,
  activity_date AS data_as_of_date
FROM customer_daily_base

UNION ALL

SELECT
  activity_date AS metric_date,
  102 AS kpi_id,
  1 AS definition_version,
  brand_code,
  service_code,
  CAST(NULL AS STRING) AS channel_code,
  CAST(NULL AS STRING) AS market_segment_code,
  SAFE_DIVIDE(churn_volume, active_customer_base) AS metric_value,
  CAST(churn_volume AS NUMERIC) AS numerator_value,
  CAST(active_customer_base AS NUMERIC) AS denominator_value,
  activity_date AS data_as_of_date
FROM customer_daily_base;
```

## Avoid ungoverned dynamic SQL

Do not store arbitrary executable SQL expressions in metadata and execute them dynamically as the primary calculation framework.

Metadata may describe the calculation and control approved parameters, but production SQL calculations should remain version-controlled, reviewable, testable, and statically analysable wherever practical.

---

# 9. Current-to-target field mapping

## Move from the metric fact

```text
Current field       Target location
------------------  --------------------------------
kpi_name            kpi_definition.kpi_code/display_name
service             retain service_code in fact only if it is part of row grain
kpi_unit            kpi_definition.default_unit_code
granularity         kpi_definition.default_granularity
threshold_red       kpi_performance_rule
threshold_amber     kpi_performance_rule
tgt_value           kpi_target
tgt_type            kpi_target.target_scenario
status              enriched serving view
wow_*               period comparison object
qoq_*               period comparison object
yoy_*               period comparison object
tgt_status           enriched serving view
```

## Keep in or add to the metric fact

```text
metric_date
kpi_id
definition_version
brand_code
service_code when part of the grain
channel_code when part of the grain
market_segment_code when part of the grain
metric_value
numerator_value
denominator_value
data_as_of_date
calculated_at
calculation_run_id
```

## Rename or replace

```text
run_date          -> metric_date plus calculated_at
data_refresh_date -> data_as_of_date
inserted_at       -> calculated_at as TIMESTAMP
owner             -> business_owner, supported by owner_group
kpi_name          -> stable kpi_code plus changeable display_name
```

---

# 10. Data-quality requirements

BigQuery key constraints are declared as `NOT ENFORCED`; therefore, the pipeline must validate integrity explicitly.

## Metric uniqueness

```sql
SELECT
  metric_date,
  kpi_id,
  definition_version,
  brand_code,
  service_code,
  channel_code,
  market_segment_code,
  COUNT(*) AS row_count
FROM `${PROJECT_ID}.${DATASET_ID}.kpi_metric_value`
GROUP BY ALL
HAVING COUNT(*) > 1;
```

Expected result: zero rows.

## Definition referential integrity

```sql
SELECT DISTINCT
  m.kpi_id,
  m.definition_version
FROM `${PROJECT_ID}.${DATASET_ID}.kpi_metric_value` AS m
LEFT JOIN `${PROJECT_ID}.${DATASET_ID}.kpi_definition` AS d
  ON m.kpi_id = d.kpi_id
 AND m.definition_version = d.definition_version
WHERE d.kpi_id IS NULL;
```

Expected result: zero rows.

## Definition version overlap

For each `kpi_id`, definition effective-date ranges must not overlap.

## Required semantic checks

Implement tests for:

- Invalid negative counts where negatives are not permitted.
- Percentages outside the agreed valid range.
- Missing denominator for ratio KPIs.
- Zero denominator handling.
- Invalid numerator/denominator relationships where relevant.
- `data_as_of_date` later than `metric_date` unless explicitly allowed.
- New values produced for inactive KPIs.
- Certified KPIs without an owner or definition.
- Invalid target scenarios.
- More than one current definition per KPI.
- More than one current target version for the same grain and scenario.
- Missing or invalid dimensional grain.
- Unexpected schema drift from the common KPI output contract.

## Blocking versus warning checks

Classify each check:

```text
BLOCKING
WARNING
INFORMATIONAL
```

Blocking failures must prevent the production merge. Warning and informational findings must be recorded in `data_quality_result` and exposed through `vw_kpi_data_quality_status`.

---

# 11. AI-agent readiness requirements

The KPI catalog must provide structured metadata for conversational analytics.

At minimum, include:

```text
kpi_code
synonyms
definition
business_use_case
aggregation_type
performance_direction
allowed_dimensions
prohibited_aggregations
default_granularity
comparison_policy
unit_code
certification_status
data_classification
interpretation_guidance
known_limitations
business_owner
owner_group
```

Example:

```text
kpi_code: NEW_CUSTOMER_VOLUME
synonyms: ["new customers", "gross adds", "customer acquisitions"]
aggregation_type: SUM
performance_direction: HIGHER_IS_BETTER
allowed_dimensions: ["brand", "service", "channel"]
prohibited_aggregations: ["AVERAGE"]
interpretation_guidance: >
  This KPI counts qualifying customer acquisitions. It must not be
  interpreted as net additions because customer losses are excluded.
```

The agent-facing data layer must make it difficult to:

- Sum non-additive metrics.
- Average ratios incorrectly.
- Compare incompatible grains.
- Treat gross additions as net additions.
- Apply higher-is-better rules to lower-is-better KPIs.
- Use expired definitions without clear historical context.
- Present uncertified KPIs as governed values.

---

# 12. Performance requirements

## Partitioning

Partition the metric fact by `metric_date` when normal consumption filters by reporting period.

Partition targets by `target_date` where target queries are date-oriented.

All production queries against partitioned tables should include partition filters unless a documented full-history operation is required.

## Clustering

Start with:

```text
kpi_id
brand_code
service_code
```

Adjust clustering only after reviewing actual query patterns and job statistics. Do not add many speculative clustering columns.

## Projection

Serving queries and models must select only required columns. Avoid unnecessary `SELECT *` in production transformations.

## Join strategy

Filter and aggregate large golden-layer sources before joining where possible. Reuse source-base models for KPIs with common lineage and grain.

## Materialisation

Use logical views for stable semantic joins initially. Consider materialising expensive, repeatedly queried outputs only after measuring usage, cost, freshness requirements, and compatibility with the selected materialisation method.

---

# 13. Migration strategy

The agent must not perform a destructive in-place replacement without a controlled migration.

## Phase 1: Introduce target structures

1. Create the new definition, metric, target, rule, audit, and validation objects.
2. Add `metric_date` and separate it from calculation timestamps.
3. Add definition versioning and effective dates.
4. Create the common KPI output contract.
5. Create the initial enriched serving view.

## Phase 2: Modularise calculations

1. Inventory all existing KPIs and their source tables.
2. Group KPIs by domain, source, grain, and refresh dependency.
3. Extract reusable source bases.
4. Move each KPI calculation into a module.
5. Reconcile new module outputs against the existing production output.
6. Add automated schema and data-quality tests.

## Phase 3: Migrate targets and comparisons

1. Move target values and scenarios into `kpi_target`.
2. Implement effective-dated target joins.
3. Move RAG logic into governed rules and serving logic.
4. Replace wide comparison fields with the long-format comparison object.
5. Document the comparison policy for every KPI.

## Phase 4: Cut over consumers

1. Validate row counts and values across an agreed historical period.
2. Validate dashboard queries against the enriched view.
3. Validate representative AI-agent questions against the KPI catalog and enriched metrics view.
4. Run old and new outputs in parallel during the agreed migration window.
5. Update downstream references.
6. Deprecate old objects only after validation and stakeholder approval.

---

# 14. Repository-agent execution instructions

The implementation agent must complete the following steps in order.

## Step 1: Inspect before editing

Identify and document:

- Existing DDL files.
- Stored procedures and scheduled queries.
- Terraform or deployment configuration.
- KPI calculation branches and repeated source scans.
- Current primary grain for each KPI.
- Existing tests and CI/CD checks.
- Downstream views and dashboards referenced in the repository.
- Existing naming, formatting, linting, and deployment conventions.

Do not assume file names or paths from this guide exactly match the repository.

## Step 2: Produce an impact inventory

Before changing logic, create a repository document containing:

- Current object.
- Target object.
- Change type.
- Affected KPIs.
- Expected compatibility impact.
- Migration dependency.
- Validation method.

## Step 3: Add schemas and contracts

Create target DDL and the shared KPI output contract. Use repository variables and established deployment patterns rather than hard-coded project identifiers.

## Step 4: Refactor incrementally

Refactor one business domain or small KPI group at a time. Do not rewrite all KPIs in one unreviewable change.

## Step 5: Add tests with each module

Every migrated KPI must include:

- Grain/uniqueness validation.
- Referential-integrity validation.
- Null and valid-range tests.
- Reconciliation against the legacy result.
- Source freshness checks where applicable.
- A documented aggregation type and performance direction.

## Step 6: Preserve lineage and traceability

Every fact row must be traceable to:

- KPI definition version.
- Calculation run.
- Source data date.
- Version-controlled SQL module.

## Step 7: Update documentation

At minimum, update:

- Data model documentation.
- KPI onboarding instructions.
- Definition versioning process.
- Target maintenance process.
- Comparison policy.
- Ownership and approval process.
- Runbook and rollback instructions.

---

# 15. KPI onboarding template

Use the following checklist for each new KPI.

```markdown
## KPI: <display name>

- KPI code:
- KPI ID:
- Definition version:
- Business definition:
- Business use case:
- Business owner:
- Owner group:
- Technical owner:
- Calculation description:
- Golden source tables:
- Metric date definition:
- Data-as-of-date definition:
- Grain:
- Allowed dimensions:
- Unit:
- Aggregation type:
- Performance direction:
- Numerator definition:
- Denominator definition:
- Target scenarios:
- Performance-rule basis:
- Comparison policy:
- Freshness SLA:
- Valid range:
- Known limitations:
- Certification status:
- Reconciliation query:
- Data-quality tests:
```

A KPI must not be promoted as certified until all mandatory items are completed and approved.

---

# 16. Acceptance criteria

The refactoring is complete only when all applicable criteria below are met.

## Data model

- [ ] The atomic metric fact contains no repeated KPI description, unit, owner, threshold, or target metadata.
- [ ] The fact grain is documented and enforced through data-quality tests.
- [ ] `metric_date`, `data_as_of_date`, and `calculated_at` have distinct documented meanings.
- [ ] Historical facts are linked to the correct definition version.
- [ ] Targets support multiple scenarios and versions.
- [ ] Performance rules are governed and effective-dated.
- [ ] Multi-value and relationship metadata does not depend on parsing inconsistent delimited strings.

## SQL architecture

- [ ] KPI calculations implement a common output contract.
- [ ] The monolithic procedure is decomposed into reviewable modules.
- [ ] Shared source scans are reused where appropriate.
- [ ] Production SQL does not depend on arbitrary ungoverned dynamic expressions stored in metadata.
- [ ] Orchestration validates staging data before merging into the fact.

## Data quality

- [ ] Fact uniqueness checks return zero failures.
- [ ] Definition referential-integrity checks return zero failures.
- [ ] Definition effective periods do not overlap.
- [ ] Required semantic range and denominator checks are implemented.
- [ ] Blocking failures prevent publication.
- [ ] Data-quality outcomes are auditable by calculation run.

## Serving layer

- [ ] `vw_kpi_catalog` exposes only the intended active/current definitions by default.
- [ ] `vw_kpi_metrics_enriched` provides consumer-friendly facts and definitions.
- [ ] Period comparisons are available without widening the atomic fact.
- [ ] Target and RAG outputs are polarity-aware.
- [ ] Consumer views do not silently join to multiple current definitions, targets, or rules.

## AI readiness

- [ ] Each KPI defines aggregation behaviour.
- [ ] Each KPI defines performance direction.
- [ ] Allowed dimensions and prohibited aggregations are machine-readable.
- [ ] Synonyms and interpretation guidance are available.
- [ ] Known limitations and certification status are exposed.
- [ ] Representative agent questions have been tested for correct KPI choice, grain, aggregation, and explanation.

## Migration and operations

- [ ] Existing KPI results have been reconciled against the new implementation.
- [ ] Downstream dependencies are inventoried and updated.
- [ ] A rollback approach is documented.
- [ ] Deprecated objects have an agreed retirement path.
- [ ] Repository documentation and runbooks are updated.

---

# 17. Out of scope unless separately approved

The implementation agent must not introduce the following without explicit approval:

- Changes to agreed KPI business definitions.
- Changes to source systems or golden-layer ownership.
- Destructive deletion of legacy tables during initial migration.
- A new orchestration platform.
- A new BI semantic-layer product.
- Automatic approval or certification of KPIs.
- Arbitrary execution of SQL stored as metadata.
- Materialisation of every serving view without measured justification.

---

# 18. Expected deliverables

The repository update should contain:

1. New or revised BigQuery DDL.
2. Modular KPI SQL calculations.
3. Shared source-base transformations where appropriate.
4. Orchestration that stages, validates, and merges KPI results.
5. Target and performance-rule structures.
6. KPI catalog, enriched metrics, comparison, and quality views.
7. Automated data-quality and reconciliation tests.
8. Definition and target seed or migration scripts.
9. An impact assessment and migration mapping.
10. Updated architecture, onboarding, runbook, and rollback documentation.
11. A concise change summary describing design decisions, assumptions, unresolved items, and test evidence.

---

## Final implementation rule

The metric fact should be thin, but not reduced to only a KPI identifier and value.

Keep the complete identifying grain and operational traceability in the fact. Store definitions, ownership, units, semantic behaviour, targets, thresholds, lineage, and relationships in governed objects. Present a simple enriched interface to BI tools and AI agents through stable serving views.
