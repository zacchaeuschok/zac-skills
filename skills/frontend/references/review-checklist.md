# Frontend Review Checklist

Work through these checks in order. Each section has anti-patterns with examples and fixes.

---

## 1. State management

### Duplicated server state

**Anti-pattern**: Copying query data into useState, then manually keeping them in sync.

```typescript
// Bad
const { data: projects } = useQuery({ queryKey: ["projects"], queryFn: fetchProjects });
const [tableData, setTableData] = useState<Project[]>([]);

useEffect(() => {
  if (projects) setTableData(projects);
}, [projects]);
```

```typescript
// Good - use query data directly
const { data: projects } = useQuery({ queryKey: ["projects"], queryFn: fetchProjects });
// pass projects directly to the table
```

**Exception**: Form fields being edited. You need local state for a form because the user is modifying a copy.

### Derived state in useState

**Anti-pattern**: Storing something that could be computed.

```typescript
// Bad
const [filteredProjects, setFilteredProjects] = useState<Project[]>([]);
useEffect(() => {
  setFilteredProjects(projects?.filter(p => p.is_archived === false) ?? []);
}, [projects]);
```

```typescript
// Good - compute inline
const activeProjects = projects?.filter(p => !p.is_archived) ?? [];
// Or useMemo if the computation is expensive
const activeProjects = useMemo(
  () => projects?.filter(p => !p.is_archived) ?? [],
  [projects]
);
```

### State lifted too high

If a piece of state is only used by one child, it should live in that child, not the parent. Each state change in the parent re-renders all children.

---

## 2. Data fetching

### Missing mutation invalidation

**Anti-pattern**: Mutation succeeds but the list shows stale data because the query wasn't invalidated.

```typescript
// Bad - no invalidation
const deleteMutation = useMutation({
  mutationFn: (id: number) => api.delete(`/projects/${id}`),
});
```

```typescript
// Good
const deleteMutation = useMutation({
  mutationFn: (id: number) => api.delete(`/projects/${id}`),
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: queryKeys.projects() });
  },
});
```

### Raw query keys

**Anti-pattern**: Using string literals instead of the query key factory.

```typescript
// Bad - typo-prone, can't rename, can't prefix-invalidate
useQuery({ queryKey: ["projects"], ... });
queryClient.invalidateQueries({ queryKey: ["proejcts"] }); // silent bug
```

```typescript
// Good
useQuery({ queryKey: queryKeys.projects(), ... });
queryClient.invalidateQueries({ queryKey: queryKeys.projects() });
```

### Unhandled mutateAsync errors

`mutateAsync` throws on failure. If you `await` it without a try-catch, the error is unhandled.

```typescript
// Bad - unhandled rejection
const handleDelete = async () => {
  await deleteMutation.mutateAsync(id);
  toast.success("Deleted");
};
```

```typescript
// Good
const handleDelete = async () => {
  try {
    await deleteMutation.mutateAsync(id);
    toast.success("Deleted");
  } catch (err) {
    toastError("Failed to delete", err);
  }
};
```

**Alternative**: Use `mutate` (not `mutateAsync`) with `onSuccess`/`onError` callbacks. Then errors are handled by the mutation config, not the call site.

### Unstable query keys

If a query key includes an object, make sure the object reference is stable. Recreating it on every render means a new query on every render.

```typescript
// Bad - new object reference every render
useQuery({
  queryKey: ["activities", { status: filter, page }],
  ...
});
```

```typescript
// Good - use the factory which returns stable tuples
useQuery({
  queryKey: queryKeys.activities({ status: filter, page }),
  ...
});
```

---

## 3. Component design

### God components

If a single component file is over 300 lines, it's probably doing too much. Look for natural boundaries:

- The table and its column definitions can be separate from the page
- Forms are their own component
- Modals/dialogs are their own component
- Filter bars are their own component

### Missing loading state

Every view that depends on async data must handle the loading case. Showing nothing (or worse, showing a flash of "no results") while data loads is a poor experience.

```typescript
// Bad - "No results" flashes before data arrives
if (!projects?.length) return <p>No projects found</p>;

// Good - distinguish loading from empty
if (isLoading) return <Skeleton className="h-64 w-full" />;
if (!projects?.length) return <EmptyState message="No projects yet" />;
```

### Prop drilling

If props pass through 3+ component levels untouched, the intermediate components are just couriers. Options:

1. **Composition**: Pass the consumer as `children` to skip intermediate levels
2. **Context**: If many components need the same data, create a context
3. **Hook**: If the data comes from React Query, just call the hook again in the consumer (React Query deduplicates)

---

## 4. TypeScript

### Gratuitous `any`

Each `any` disables type checking for everything it touches. Replace with `unknown` + narrowing, or define the actual type.

```typescript
// Bad
const handleData = (data: any) => { ... };

// Good
const handleData = (data: unknown) => {
  if (!isProjectResponse(data)) throw new Error("Unexpected response shape");
  // data is now typed
};
```

### Unnecessary type assertions

`as` tells the compiler "trust me." If you need it often, the upstream type is probably wrong. Fix the type, don't patch the consumers.

```typescript
// Suspicious - why doesn't the type already know this?
const name = row.getValue("name") as string;

// Better - type the column definition so the cell type is inferred
accessorKey: "name" as const,
```

---

## 5. Styling

### Hardcoded values

```typescript
// Bad
<div style={{ color: "#3b82f6", padding: "12px" }}>

// Good - use tokens
<div className="text-primary p-3">
```

### Inconsistent component usage

If the project uses shadcn Dialog, don't build a custom modal with a div and backdrop. If the project uses shadcn Button, don't use a raw `<button>` with inline styles.

### Debug data at primary visual weight

Raw JSON, internal IDs, and diagnostic state should not compete visually with user-facing content. Put them in collapsible sections, secondary panes, or behind a "Show details" toggle.

---

## 6. Conventions

### Import style

Check whether the project uses `@/` aliases or relative imports. Use whatever the project uses. Don't mix.

### Error handling

Check for a `toastError` utility or similar. Use it instead of writing ad-hoc error formatting in every catch block.

### File naming

Match the project's convention. If hooks are `use-kebab-case.ts`, don't create `useProjects.ts`.
