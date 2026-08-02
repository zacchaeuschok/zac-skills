---
name: frontend
description: Build and review frontend features in React + TypeScript + Vite + Tailwind CSS + shadcn/ui + TanStack (Query, Table, Router) stacks. Covers both creation (scaffolding pages, hooks, types, data tables, forms) and review (checking for anti-patterns, missed conventions, stale state, leaky abstractions). Use whenever the user asks to "add a page", "create a component", "scaffold a feature", "wire up an endpoint", "add a data table", "build a form", "add CRUD for X", "review this component", "what's wrong with this page", "check this hook", "review the frontend", "is this idiomatic React", or any task involving building or evaluating UI in a React + TanStack + shadcn stack. Also triggers on mentions of React Query patterns, column definitions, mutation invalidation, shadcn components, or Tailwind layout questions in the context of feature work.
argument-hint: "[create|review] [target path or feature name]"
model: opus
effort: high
---

# /frontend

Two modes: **create** and **review**. Infer which one from context. If the user says "add", "build", "scaffold", "wire up", or "create" they want creation. If they say "review", "check", "what's wrong", "is this right", or paste code for feedback, they want review. When ambiguous, ask.

## Before you start

Read the project's CLAUDE.md (or equivalent) if one exists. It contains stack-specific conventions, component locations, and design principles that override the generic patterns here. The patterns in this skill are defaults - project-level instructions always win.

Then orient yourself in the codebase:

1. **Find the structure**: where do pages, hooks, types, and components live? (Common: `src/pages/`, `src/hooks/`, `src/types/`, `src/components/`)
2. **Find a reference example**: pick an existing page that does something similar to what the user is asking for. Read it. This is your template - match its conventions, not your assumptions.
3. **Find the API client**: how does this project make API calls? (Look for a `useApi` hook, an `api.ts` utility, or an axios instance.)
4. **Find the query keys**: is there a centralized query key factory? (Look for `query-keys.ts` or similar.) If so, use it. Never use raw string keys.

---

## Mode: Create

When building a new feature, you typically touch 4-5 files. The order matters because each file depends on the previous one. But not every feature needs all steps. A read-only detail page doesn't need forms. A settings panel doesn't need a data table. Only create what the feature actually requires.

### Step 1: Types

Create the TypeScript interfaces that match the API schema. Read `${CLAUDE_SKILL_DIR}/references/types.md` for the naming conventions and patterns.

Key rules:
- One file per resource in the types directory
- Separate interfaces for create requests, update requests, list items, and detail responses
- List items are leaner than detail responses (only what the table needs)
- JSON blobs are `Record<string, unknown>`, not `any`
- Nullable fields use `| null`, not optional (`?`)
- Match the API response shape exactly. Don't rename fields or reshape data in types.

### Step 2: Hook

Create a React Query hook that wraps all API operations for this resource. Read `${CLAUDE_SKILL_DIR}/references/hooks.md` for the full pattern.

Key rules:
- One hook per resource (e.g., `useProjects`, `useTasks`)
- The hook returns query data + mutation functions in a single object
- Every mutation's `onSuccess` must invalidate the relevant query keys
- Use the project's query key factory, not raw strings
- Polling (refetchInterval) is conditional: only poll when there's a reason to (e.g., a background task is running)
- Never put business logic in the hook. It fetches, mutates, and invalidates. That's it.

### Step 3: Page component

Create the page that renders the feature. Read `${CLAUDE_SKILL_DIR}/references/pages.md` for the structure.

Key rules:
- Pages are composition, not logic. They wire hooks to components.
- Local state manages UI concerns: which modal is open, which row is selected, what tab is active.
- Server state comes from the hook. Never duplicate it into local state.
- Loading and error states are handled. Show skeletons during load, toast on error.
- Column definitions for data tables live in the page file (they're page-specific presentation logic).

### Step 4: Data table (if applicable)

If the feature has a list view, use the project's DataTable component. Read `${CLAUDE_SKILL_DIR}/references/data-table.md` for column definitions, sorting, and pagination patterns.

Key rules:
- Column definitions are typed: `ColumnDef<YourListItem>[]`
- Sortable columns use a header with a sort toggle icon
- Action columns have `id: "actions"` (no `accessorKey`)
- Pass `initialSorting` for default sort order
- Cell renderers handle null/undefined gracefully

### Step 5: Forms (if applicable)

If the feature needs create/edit forms, read `${CLAUDE_SKILL_DIR}/references/forms.md`.

Key rules:
- Validate with Zod schemas that mirror the request types
- Use `safeParse`, not `parse` - handle the error branch with a toast
- Disable the submit button while the mutation is pending
- Clear/reset the form on success
- Show the mutation error via `toastError()` or the project's error utility

### Step 6: Routing

Add the route to the router config. Match the existing pattern:
- Nest under the appropriate layout
- Use the same guard components as sibling routes
- Path naming: plural for list pages (`/tasks`), singular with param for detail (`/tasks/:id`)

### Step 7: Navigation

Add a link in the sidebar or navigation. Match the icon style and label pattern of existing nav items.

### After creating

Run the dev server. Navigate to the new page. Verify:
- Data loads and displays
- Loading skeleton appears during fetch
- Empty state shows when there's no data
- Sorting works (if data table)
- Create/edit form submits and the list refreshes
- Error states toast correctly
- The page doesn't flash or re-render unnecessarily

---

## Mode: Review

When reviewing frontend code, work through these checks systematically. Read `${CLAUDE_SKILL_DIR}/references/review-checklist.md` for the full checklist with examples.

### 1. State management

- **Duplicated state**: Is server data copied into `useState`? It shouldn't be. Use the query data directly. The only exception is form fields being edited.
- **Stale closures**: Are event handlers or effects capturing stale values? Look for missing deps in `useEffect` and `useCallback`.
- **Derived state**: Is something in `useState` that could be computed from other state? Compute it inline or with `useMemo`.
- **Unnecessary re-renders**: Is state lifted too high? Does a parent re-render when only a child's state changed?

### 2. Data fetching

- **Missing invalidation**: Does every mutation invalidate the queries it affects? A create mutation that doesn't invalidate the list query leaves stale data on screen.
- **Raw query keys**: Are query keys raw strings instead of using the factory? This causes silent cache misses.
- **Missing error handling**: Do mutations have error handling? Unhandled promise rejections from `mutateAsync` crash silently.
- **Over-fetching**: Is the component fetching data it doesn't use? Are there queries that fire on every render because their key isn't stable?
- **Polling without stop condition**: Is `refetchInterval` always on, or does it stop when the condition is met?

### 3. Component design

- **God components**: Is a single component doing too much? Pages should compose smaller components, not contain hundreds of lines of JSX.
- **Prop drilling**: Are props passed through 3+ levels? Consider composition (children) or a context.
- **Inline functions in JSX**: Are new function instances created every render in event handlers that get passed to memoized children?
- **Missing loading/error/empty states**: Every data-dependent view needs all three.

### 4. TypeScript

- **`any` types**: Replace with proper interfaces or `unknown` + narrowing.
- **Type assertions (`as`)**: Each one is a lie to the compiler. Are they necessary, or is the underlying type wrong?
- **Missing return types on hooks**: Custom hooks should have explicit return types so consumers get autocomplete.
- **Inconsistent nullability**: Is the same field `null` in one place and `undefined` in another?

### 5. Styling and UI

- **Hardcoded values**: Colors, spacing, radii should use Tailwind tokens or CSS variables, not hex codes or pixel values.
- **Inconsistent patterns**: Does this component use different UI patterns than its siblings? (e.g., a custom modal when the rest of the app uses shadcn Dialog)
- **Debug data leakage**: Are raw IDs, JSON dumps, or internal state shown at the same visual weight as user-facing content? Debug info belongs in collapsible secondary panes.
- **Missing responsive behavior**: Does the layout break at common viewport widths?

### 6. Conventions

- **Import aliases**: Using `@/` prefix? Or relative paths? Match the project.
- **Error handling**: Using the project's error utility (e.g., `toastError`)? Or ad-hoc `console.error`?
- **Component file naming**: Does the file name match the project's convention? (kebab-case, PascalCase, etc.)
- **Query key discipline**: Using the factory? Invalidating by prefix where appropriate?

### Review output format

```
## Frontend Review: [Component/Feature Name]

### Issues (ranked by impact)
1. **[Category]**: [What's wrong] - [Why it matters] - [Fix]
2. ...

### Good patterns (worth keeping)
- [What's done well and why]

### Suggestions (non-blocking)
- [Nice-to-haves that aren't bugs]
```

---

## Reference files

These contain detailed patterns with code examples. Read them when you need specifics:

- `${CLAUDE_SKILL_DIR}/references/types.md` - Type naming conventions, interface patterns, nullability rules
- `${CLAUDE_SKILL_DIR}/references/hooks.md` - React Query hook structure, mutation patterns, polling, query key factories
- `${CLAUDE_SKILL_DIR}/references/pages.md` - Page composition, state management, modal patterns, layout
- `${CLAUDE_SKILL_DIR}/references/data-table.md` - Column definitions, sorting, pagination, action columns, cell renderers
- `${CLAUDE_SKILL_DIR}/references/forms.md` - Zod validation, form state, submit handling, error display
- `${CLAUDE_SKILL_DIR}/references/review-checklist.md` - Full review checklist with anti-pattern examples and fixes
