# Icons

- **Import icons directly from the project's configured icon library** (`lucide-react`, `@tabler/icons-react`, …). Check which one the project uses — never assume. Icons render fine in Server Components — no client boundary needed.
- **Pass icons as component objects, never string keys to a lookup map**:

```tsx
// Bad
const iconMap = { check: CheckIcon, alert: AlertIcon };
const StatusBadge = ({ icon }: { icon: string }) => { const Icon = iconMap[icon]; return <Icon />; };

// Good
import { CheckIcon } from 'lucide-react';
const StatusBadge = ({ icon: Icon }: { icon: React.ComponentType }) => <Icon />;
<StatusBadge icon={CheckIcon} />
```

- **No sizing classes on icons inside components** — `Button`, `DropdownMenuItem`, `Alert`, etc. size their icons via CSS. No `size-4`, no `mr-2`:

```tsx
// Bad
<Button><SearchIcon className="mr-2 size-4" /> Search</Button>

// Good — position with data-icon, sizing is the component's job
<Button><SearchIcon data-icon="inline-start" /> Search</Button>
<Button>Next <ArrowRightIcon data-icon="inline-end" /></Button>
```
