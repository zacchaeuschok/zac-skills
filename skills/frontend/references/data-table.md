# Data Table Patterns

## Usage

Most projects using this stack have a generic `<DataTable>` component built on TanStack Table v8. It handles rendering, sorting, and pagination. You define the columns; it does the rest.

```typescript
<DataTable
  columns={columns}
  data={items ?? []}
  isLoading={isLoading}
  initialSorting={[{ id: "created_at", desc: true }]}
/>
```

## Column definitions

Columns are typed as `ColumnDef<YourListItem>[]` and typically defined in the page file or a co-located columns file.

### Basic column

```typescript
import { ColumnDef } from "@tanstack/react-table";
import type { ProjectListItem } from "@/types/project";

const columns: ColumnDef<ProjectListItem>[] = [
  {
    accessorKey: "name",
    header: "Name",
  },
  {
    accessorKey: "owner",
    header: "Owner",
  },
];
```

### Sortable column

```typescript
import { ArrowUpDown } from "lucide-react";
import { Button } from "@/components/ui/button";

{
  accessorKey: "created_at",
  header: ({ column }) => (
    <Button
      variant="ghost"
      onClick={() => column.toggleSorting(column.getIsSorted() === "asc")}
    >
      Created
      <ArrowUpDown className="ml-2 h-4 w-4" />
    </Button>
  ),
  cell: ({ row }) => formatDate(row.getValue("created_at")),
}
```

### Custom cell renderer

```typescript
{
  accessorKey: "status",
  header: "Status",
  cell: ({ row }) => {
    const status = row.getValue("status") as string;
    return (
      <Badge variant={status === "active" ? "default" : "secondary"}>
        {status}
      </Badge>
    );
  },
}
```

### Action column

Action columns have `id` instead of `accessorKey` because they don't map to a data field:

```typescript
{
  id: "actions",
  header: "",
  cell: ({ row }) => {
    const item = row.original;
    return (
      <DropdownMenu>
        <DropdownMenuTrigger asChild>
          <Button variant="ghost" size="icon">
            <MoreHorizontal className="h-4 w-4" />
          </Button>
        </DropdownMenuTrigger>
        <DropdownMenuContent align="end">
          <DropdownMenuItem onClick={() => setEditingId(item.id)}>
            Edit
          </DropdownMenuItem>
          <DropdownMenuItem
            onClick={() => setDeleteTarget(item)}
            className="text-destructive"
          >
            Delete
          </DropdownMenuItem>
        </DropdownMenuContent>
      </DropdownMenu>
    );
  },
}
```

## Null handling in cells

Cell renderers must handle null/undefined gracefully. The table will call the renderer even when the value is null.

```typescript
cell: ({ row }) => {
  const date = row.getValue("last_activity_at") as string | null;
  return date ? formatDate(date) : <span className="text-muted-foreground">Never</span>;
}
```

## Initial sorting

Set a default sort so the table isn't randomly ordered on first load:

```typescript
<DataTable
  columns={columns}
  data={data}
  initialSorting={[{ id: "created_at", desc: true }]}
/>
```

## Pagination

The DataTable typically handles client-side pagination internally. If the project uses server-side pagination, the hook will accept page/limit params and return total count. Check the existing pattern before assuming client-side.

## Clickable rows

Some tables make entire rows clickable for navigation to a detail page:

```typescript
<DataTable
  columns={columns}
  data={data}
  onRowClick={(row) => navigate(`/projects/${row.original.id}`)}
/>
```

Check whether the project's DataTable supports this prop before using it. If not, put navigation in the action column or make the name cell a link.
