# Type Patterns

## File location

One file per resource: `src/types/task.ts`, `src/types/project.ts`, etc.

## Naming conventions

| Purpose | Pattern | Example |
|---------|---------|---------|
| Create request body | `Create{Resource}Request` | `CreateTaskRequest` |
| Update request body | `Update{Resource}Request` | `UpdateTaskRequest` |
| List item (table row) | `{Resource}ListItem` | `TaskListItem` |
| Detail response | `{Resource}DetailResponse` | `TaskDetailResponse` |
| Single response | `{Resource}Response` | `TaskResponse` |

## List items vs detail responses

List items contain only what the table needs to render a row. Detail responses contain the full resource with nested relations. This distinction matters because list endpoints return arrays - keeping the item shape lean avoids transferring fields nobody renders.

```typescript
// Lean - only what the table shows
export interface ProjectListItem {
  id: number;
  name: string;
  owner: string;
  is_archived: boolean;
  last_activity_at: string | null;
}

// Full - everything about the resource
export interface ProjectDetailResponse {
  id: number;
  name: string;
  owner: string;
  is_archived: boolean;
  last_activity_at: string | null;
  metadata: Record<string, unknown>;
  activity_history: ActivitySummary[];
  task_history: TaskSummary[];
  created_at: string;
  updated_at: string;
}
```

## Rules

**Nullable vs optional**: Use `| null` for fields the API returns as `null`. Use `?` only for fields the API omits entirely. These are different things.

```typescript
// The API always returns last_activity_at, but it might be null
last_activity_at: string | null;

// The API only includes metadata when requested via ?expand=metadata
metadata?: Record<string, unknown>;
```

**JSON blobs**: Use `Record<string, unknown>`, never `any`. If the shape is known, define it.

```typescript
// Unknown shape
metadata: Record<string, unknown>;

// Known shape
config: {
  max_retries: number;
  timeout_seconds: number;
  fallback_template_id: number | null;
};
```

**Dates**: Keep as `string` (ISO 8601). Don't convert to `Date` in the type - that's a runtime concern.

**Enums vs unions**: Prefer string unions. They're simpler and match the API's string values directly.

```typescript
// Prefer this
status: "pending" | "active" | "completed" | "failed";

// Over this
status: TaskStatus; // enum defined elsewhere
```

**Match the API exactly**: Don't rename `friendly_name` to `friendlyName`. Don't reshape nested objects into flat ones. The type is a contract with the API. If the API sends snake_case, the type uses snake_case.
