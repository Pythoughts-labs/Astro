---
description: Rules for TypeScript configuration in Astro projects
globs:
  - "tsconfig.json"
  - "src/env.d.ts"
  - "**/*.ts"
  - "**/*.astro"
---

# Astro TypeScript Rules

## tsconfig.json

```json
{
  "extends": "astro/tsconfigs/strict",
  "include": [".astro/types.d.ts", "**/*"],
  "exclude": ["dist"],
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@layouts/*": ["src/layouts/*"],
      "@content/*": ["src/content/*"]
    }
  }
}
```

## MUST DO

- Use `astro/tsconfigs/strict` or `strictest` preset
- Set `include` to pick up `.astro/types.d.ts` and `exclude` `dist` to avoid checking built files
- Define path aliases for cleaner imports
- Type Props interface in all components
- Type locals in `src/env.d.ts`
- Type environment variables
- Run `astro check` for type checking (`astro build` does NOT catch type errors); use `"build": "astro check && astro build"`

## MUST NOT DO

- Use `any` type - use `unknown` and narrow
- Skip typing Props interface
- Ignore TypeScript errors
- Use `astro/tsconfigs/base` in production projects

## Component Props

```astro
---
interface Props {
  title: string;
  description?: string;
  tags: string[];
  variant?: 'primary' | 'secondary';
}

const {
  title,
  description = 'Default description',
  tags,
  variant = 'primary',
} = Astro.props;
---
```

## Type Imports

Use explicit `import type` for type-only imports:

```typescript
import type { Foo } from './x';
```

`verbatimModuleSyntax: true` is enabled by default in all Astro presets and will flag imports that should use `import type`.

## Type Utilities (astro/types)

Astro provides built-in utility types under the `astro/types` entrypoint:

```typescript
import type { HTMLAttributes, ComponentProps, HTMLTag, Polymorphic } from 'astro/types';

// Mirror a native element's attributes
type AnchorProps = HTMLAttributes<'a'>;

// Reference another component's props
type ButtonProps = ComponentProps<typeof Button>;

// Polymorphic `as`-prop components
type Props<Tag extends HTMLTag> = Polymorphic<{ as: Tag }>;
```

Infer `Astro.params` / `Astro.props` types for dynamic routes:

```typescript
import type {
  InferGetStaticParamsType,
  InferGetStaticPropsType,
  GetStaticPaths,
} from 'astro';

export const getStaticPaths = (async () => {
  // ...
}) satisfies GetStaticPaths;

type Params = InferGetStaticParamsType<typeof getStaticPaths>;
type Props = InferGetStaticPropsType<typeof getStaticPaths>;
```

## Environment Variables

```typescript
// src/env.d.ts
// Astro types, not necessary if you already have a `tsconfig.json`
/// <reference path="../.astro/types.d.ts" />

interface ImportMetaEnv {
  readonly PUBLIC_API_URL: string;
  readonly DATABASE_URL: string;
  readonly SECRET_KEY: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

## Locals Typing

```typescript
// src/env.d.ts
declare namespace App {
  interface Locals {
    user: {
      id: string;
      email: string;
      name: string;
    } | null;
    theme: 'light' | 'dark';
    requestId: string;
  }
}
```

## Content Collection Types

```typescript
// src/content.config.ts
import { defineCollection, reference } from 'astro:content';
import { glob } from 'astro/loaders';
import { z } from 'astro/zod';

const blog = defineCollection({
  loader: glob({ pattern: '**/*.md', base: './src/data/blog' }),
  schema: z.object({
    title: z.string(),
    author: reference('authors'),
  }),
});

export const collections = { blog };

// Types are auto-generated:
// import type { CollectionEntry } from 'astro:content';
// type BlogPost = CollectionEntry<'blog'>;
```

> Config lives in `src/content.config.ts` and collections use a `loader` (Content Layer API). Entries are keyed by `id` (the old `slug` field was removed).

## API Route Types

```typescript
// src/pages/api/example.ts
import type { APIRoute, APIContext } from 'astro';

export const GET: APIRoute = async (context: APIContext) => {
  const { params, request, cookies, locals, url } = context;

  return new Response(JSON.stringify({ ok: true }), {
    headers: { 'Content-Type': 'application/json' },
  });
};
```

## Path Aliases Usage

```astro
---
// With aliases
import Layout from '@layouts/Layout.astro';
import Card from '@components/Card.astro';
import { getCollection } from 'astro:content';

// Instead of
import Layout from '../../../layouts/Layout.astro';
---
```
