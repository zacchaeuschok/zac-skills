# Form Patterns

## Validation with Zod

Define a Zod schema that mirrors the request type. Use `safeParse` (not `parse`) so you control the error handling.

```typescript
import { z } from "zod";

const createProjectSchema = z.object({
  name: z.string().min(1, "Name is required"),
  owner: z.string().min(1, "Owner is required"),
  metadata: z.record(z.string(), z.unknown()).optional(),
});
```

## Simple form pattern

For forms where you control the state directly:

```typescript
export function ProjectCreateForm({ onSuccess }: { onSuccess?: () => void }) {
  const { createProject } = useProjects();
  const [name, setName] = useState("");
  const [owner, setOwner] = useState("");

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    const result = createProjectSchema.safeParse({ name, owner });
    if (!result.success) {
      toast.error(result.error.issues[0]?.message ?? "Validation failed");
      return;
    }

    try {
      await createProject.mutateAsync(result.data);
      toast.success("Project created");
      setName("");
      setOwner("");
      onSuccess?.();
    } catch (err) {
      toastError("Failed to create project", err);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div className="space-y-2">
        <Label htmlFor="name">Name</Label>
        <Input
          id="name"
          value={name}
          onChange={(e) => setName(e.target.value)}
        />
      </div>
      <div className="space-y-2">
        <Label htmlFor="owner">Owner</Label>
        <Input
          id="owner"
          value={owner}
          onChange={(e) => setOwner(e.target.value)}
        />
      </div>
      <Button type="submit" disabled={createProject.isPending}>
        {createProject.isPending ? "Creating..." : "Create Project"}
      </Button>
    </form>
  );
}
```

## React Hook Form pattern

For complex forms with many fields, use react-hook-form with Zod resolver:

```typescript
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";

export function TaskEditForm({ task }: { task: TaskDetailResponse }) {
  const { updateTask } = useTasks();

  const form = useForm<z.infer<typeof updateTaskSchema>>({
    resolver: zodResolver(updateTaskSchema),
    defaultValues: {
      friendly_name: task.friendly_name,
      description: task.description ?? "",
    },
  });

  const onSubmit = async (data: z.infer<typeof updateTaskSchema>) => {
    try {
      await updateTask.mutateAsync({ id: task.id, ...data });
      toast.success("Task updated");
    } catch (err) {
      toastError("Failed to update task", err);
    }
  };

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField
          control={form.control}
          name="friendly_name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Name</FormLabel>
              <FormControl>
                <Input {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <Button type="submit" disabled={updateTask.isPending}>
          Save
        </Button>
      </form>
    </Form>
  );
}
```

## JSON editor forms

For resources with freeform JSON (configs, trees, metadata), use the project's JSON editor component:

```typescript
const [jsonValue, setJsonValue] = useState(JSON.stringify(initial, null, 2));

const handleSubmit = async () => {
  let parsed: unknown;
  try {
    parsed = JSON.parse(jsonValue);
  } catch {
    toast.error("Invalid JSON");
    return;
  }

  const result = schema.safeParse(parsed);
  if (!result.success) {
    toast.error(`Invalid: ${result.error.issues[0]?.message}`);
    return;
  }

  // submit result.data
};

<JsonEditor value={jsonValue} onChange={setJsonValue} height="400px" />
```

## Form in a dialog

Wrap the form in a shadcn Dialog. The dialog controls open/close; the form handles submit:

```typescript
<Dialog open={open} onOpenChange={onOpenChange}>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>Create Project</DialogTitle>
    </DialogHeader>
    <ProjectCreateForm onSuccess={() => onOpenChange(false)} />
  </DialogContent>
</Dialog>
```

## Error display

Use the project's error utility. If the project has `toastError(prefix, err)`, use it. If not, follow this pattern:

```typescript
catch (err) {
  const message = err instanceof Error ? err.message : "Unknown error";
  toast.error(`Failed to create project: ${message}`);
}
```

Never `console.error` in place of showing the user what happened. The user can't see the console.
