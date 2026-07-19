# 054 - Procedural Memory Specification

## Purpose

Define the Procedural Memory subsystem that stores learned skills, successful strategies, and reusable plan templates that improve VOXY's effectiveness over time.

## Scope

- Skill and strategy storage
- Plan template management
- Success pattern recording
- Skill retrieval and matching
- Skill versioning
- Skill activation

## Responsibilities

- Store successful plans as templates
- Record effective strategies
- Match current situations to known skills
- Activate appropriate skills
- Track skill success rates
- Update skills based on outcomes

## Non-Goals

- Event storage (Episodic Memory)
- Fact storage (Semantic Memory)
- Real-time skill learning

## Architecture Position

Procedural Memory stores the "how to do" knowledge. It feeds into Planner for plan generation.

## Inputs

- Successful plans from Executor
- Strategy descriptions
- User feedback on plan quality
- Reflection suggestions

## Outputs

- Skill templates
- Strategy recommendations
- Plan templates
- Success rate statistics

## Interfaces

```rust
pub struct ProceduralMemory {
    pub fn store_skill(skill: Skill) -> Result<SkillId, MemoryError>;
    pub fn find_skills(context: &Context) -> Vec<Skill>;
    pub fn activate_skill(id: SkillId) -> Result<SkillInstance, MemoryError>;
    pub fn record_outcome(id: SkillId, success: bool);
    pub fn get_skill_stats(id: SkillId) -> SkillStats;
}
```

## Data Models

### Skill

| Field | Type | Description |
|-------|------|-------------|
| id | SkillId | Unique identifier |
| name | String | Skill name |
| description | String | Description |
| trigger_patterns | Vec<String> | Activation patterns |
| plan_template | Plan | Reusable plan |
| success_rate | f32 | Historical success rate |
| usage_count | u32 | Times used |
| created_at | DateTime | Creation time |
| last_used | DateTime | Last activation |

### SkillStats

| Field | Type | Description |
|-------|------|-------------|
| success_rate | f32 | Success percentage |
| average_duration_ms | u64 | Average execution time |
| usage_count | u32 | Total uses |
| last_outcome | bool | Last result |

## State Management

- Skills stored in SQLite
- Success rates updated dynamically
- Templates are immutable versions

## Threading Model

Skill operations run on blocking pool.

## IPC Requirements

- No IPC for procedural memory

## Event Definitions

| Event | Payload | Description |
|-------|---------|-------------|
| `procedural.skill_stored` | Skill | Skill stored |
| `procedural.skill_activated` | SkillId | Skill activated |
| `procedural.outcome_recorded` | (id, success) | Outcome recorded |

## Error Handling

| Code | Description | Recovery |
|------|-------------|----------|
| `PM001` | Skill not found | Return empty |
| `PM002` | Activation failed | Log, skip skill |

## Retry Strategy

- Store: 2 retries
- Activation: 1 retry

## Recovery Strategy

- Activation failure: Skip skill, continue

## Security Requirements

- Skills validated before storage
- No arbitrary code in skills

## Configuration

```json
{
  "procedural_memory": {
    "max_skills": 10000,
    "min_success_rate_for_activation": 0.5,
    "template_matching_threshold": 0.7,
    "auto_update_enabled": true
  }
}
```

## Performance Targets

| Metric | Target |
|--------|--------|
| Store latency | < 20ms |
| Skill matching | < 50ms |
| Activation | < 10ms |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Max skills | 10,000 |
| Template size | 50KB |

## Benchmarks

- `bench_store`: Storage speed
- `bench_matching`: Matching speed

## Testing Requirements

- Storage tests
- Matching accuracy tests
- Activation tests

## Logging Requirements

- Storage at DEBUG level
- Activation at INFO level

## Telemetry

- Skills stored
- Activation frequency
- Success rates

## Cross References

- [050_MEMORY_RUNTIME_SPEC.md](050_MEMORY_RUNTIME_SPEC.md)
- [042_PLANNER_SPEC.md](042_PLANNER_SPEC.md)
- [044_REFLECTION_SPEC.md](044_REFLECTION_SPEC.md)

## References

- Procedural Memory (Cognitive Psychology)
- Case-Based Reasoning
- Plan Library Management
