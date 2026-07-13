# Project & Package Structure

## No barrel files — anywhere

- Never create an `index.ts` whose job is re-exporting. Consumers import the defining file directly:

```ts
// Good
import { useUploadFile } from '@feature/upload/hooks/use-upload-file';
import { Dropzone } from '@feature/upload/components/dropzone';

// Bad
import { useUploadFile, Dropzone } from '@feature/upload';
```

Direct imports keep tree-shaking exact, keep server/client boundaries visible, and make "where is this defined" a one-jump question.

## Feature packages (monorepo)

- **When a feature package is warranted:** the feature is shared by 2+ apps, or genuinely versionable/deployable on its own. **When not:** single-app features — a feature-colocated route folder is the default (see [nextjs.md](nextjs.md)); don't manufacture a package for one consumer.
- Slice by responsibility with a consistent directory taxonomy — `components/`, `hooks/`, `actions/`, `services/`, `types/`, `constants/`, `lib/` — not a generic dump. Add dirs only when a file exists to live there.
- **`package.json` exposes wildcard per-file subpath exports** — every file individually importable, no barrels:

```jsonc
"exports": {
  "./components/*": { "types": "./dist/src/components/*.d.ts", "default": "./src/components/*.tsx" },
  "./hooks/*":      { "types": "./dist/src/hooks/*.d.ts",      "default": "./src/hooks/*.ts" },
  "./services/*":   { "types": "./dist/src/services/*.d.ts",   "default": "./src/services/*.ts" },
  "./constants/*":  { "types": "./dist/src/constants/*.d.ts",  "default": "./src/constants/*.ts" }
}
```

- Wildcard patterns resolve exactly one extension per directory — keep `hooks/`, `services/`, `actions/`, `constants/`, `lib/` pure `.ts`; JSX lives only in `components/` (`*.tsx`). A hook that wants JSX gets split: logic hook in `hooks/`, markup in `components/`.
- `"type": "module"`, `"sideEffects": false`. Scoped package names; internal deps `workspace:*`, shared-versioned externals via `catalog:`.
- Extensible systems (dynamic forms, plugin-style renderers) use a **registry pattern**: plain definition objects, a Map-backed registry, a factory to instantiate, and a renderer that maps type → component. New capability = new registry entry, not new wiring. **When:** 3+ interchangeable variants, or external code must add capabilities. **When not:** two known cases — a plain lookup map or `switch` is simpler and stays greppable.

## App repos (Next.js)

- `src/` layout: `app/`, `components/`, `lib/`, `hooks/`, `data/`, `constants/`, `services/`, `store/`, `providers/`, `styles/`, `types/`, `env.ts`.
- Feature-colocated route folders own their `components/`, `lib/`, `store/`, `hooks/`; only shared code is hoisted (see [nextjs.md](nextjs.md)).
