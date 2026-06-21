# Configuration

Astro configuration lives in `astro.config.mjs` at the project root.

## Basic Configuration

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';

export default defineConfig({
  // Your configuration options here
});
```

## Common Options

### Site URL

```javascript
export default defineConfig({
  site: 'https://example.com',
  base: '/blog', // For subdirectory deployments
});
```

### Output Mode

```javascript
import node from '@astrojs/node';

export default defineConfig({
  output: 'static',    // Default - prerender all pages; opt out per route with `export const prerender = false`
  // output: 'server', // SSR all pages; opt in per route with `export const prerender = true`

  adapter: node({      // Required for server output / per-route SSR
    mode: 'standalone',
  }),
});
```

> **Note:** The only output modes are `'static'` (default) and `'server'`. There is no `'hybrid'` mode — its behavior is now the default `'static'`.

### Integrations

```javascript
import react from '@astrojs/react';
import mdx from '@astrojs/mdx';
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  integrations: [
    react(),
    mdx(),
    sitemap(),
  ],
});
```

For Tailwind, the `@astrojs/tailwind` integration is deprecated (legacy Tailwind 3 only). Use the official Tailwind Vite plugin for Tailwind 4 — run `npx astro add tailwind` (installs `@tailwindcss/vite`):

```javascript
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  vite: { plugins: [tailwindcss()] },
});
```

Then add `@import "tailwindcss";` to `src/styles/global.css` and import that file in your layout.

### Build Options

```javascript
export default defineConfig({
  build: {
    format: 'directory',  // /about/index.html (default)
    // format: 'file',    // /about.html

    inlineStylesheets: 'auto', // Inline small stylesheets
    // inlineStylesheets: 'always',
    // inlineStylesheets: 'never',

    assets: '_astro',     // Assets directory name
  },

  compressHTML: true,     // Minify HTML output

  outDir: './dist',       // Build output directory
  publicDir: './public',  // Static assets directory
});
```

### Dev Server

```javascript
export default defineConfig({
  server: {
    port: 4321,           // Default port
    host: true,           // Expose to network
    open: true,           // Open browser on start
  },

  devToolbar: {
    enabled: true,        // Show dev toolbar
  },
});
```

### Prefetching

```javascript
export default defineConfig({
  prefetch: {
    prefetchAll: true,              // Prefetch all links
    defaultStrategy: 'viewport',    // 'hover' | 'viewport' | 'load'
  },
});
```

## Vite Configuration

```javascript
export default defineConfig({
  vite: {
    plugins: [],
    resolve: {
      alias: {
        '@': '/src',
        '@components': '/src/components',
      },
    },
    css: {
      preprocessorOptions: {
        scss: {
          additionalData: `@import "@/styles/variables.scss";`,
        },
      },
    },
    build: {
      rollupOptions: {
        output: {
          manualChunks: {
            vendor: ['react', 'react-dom'],
          },
        },
      },
    },
  },
});
```

## Markdown Configuration

```javascript
import remarkToc from 'remark-toc';
import rehypeSlug from 'rehype-slug';

export default defineConfig({
  markdown: {
    syntaxHighlight: 'shiki',  // 'shiki' | 'prism' | false (default: { type: 'shiki', excludeLangs: ['math'] }; also accepts a full config object)

    shikiConfig: {
      theme: 'dracula',
      wrap: true,
    },

    remarkPlugins: [
      remarkToc,
      [remarkPlugin, { option: true }],  // With options
    ],

    rehypePlugins: [
      rehypeSlug,
    ],

    gfm: true,                // GitHub Flavored Markdown
    smartypants: true,        // Smart quotes
  },
});
```

## i18n Configuration

```javascript
export default defineConfig({
  i18n: {
    defaultLocale: 'en',
    locales: ['en', 'es', 'fr', 'de'],

    routing: {
      prefixDefaultLocale: false,  // /about vs /en/about
      redirectToDefaultLocale: true,
    },

    fallback: {
      es: 'en',  // Fallback to English for Spanish
    },
  },
});
```

## Image Configuration

```javascript
export default defineConfig({
  image: {
    // Allowed remote image domains
    domains: ['example.com', 'cdn.example.com'],

    // Allowed remote patterns
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '**.amazonaws.com',
      },
    ],

    // Image service configuration
    service: {
      entrypoint: 'astro/assets/services/sharp',
      config: {
        limitInputPixels: false,
      },
    },
  },
});
```

## Redirects

```javascript
export default defineConfig({
  redirects: {
    '/old-page': '/new-page',
    '/old-blog/[...slug]': '/blog/[...slug]',
    '/twitter': {
      status: 302,
      destination: 'https://twitter.com/astrodotbuild',
    },
  },
});
```

## TypeScript Configuration

### tsconfig.json

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
      "@layouts/*": ["src/layouts/*"]
    }
  }
}
```

Available presets:
- `astro/tsconfigs/base` - Minimal
- `astro/tsconfigs/strict` - Recommended
- `astro/tsconfigs/strictest` - Maximum strictness

### Type Declarations

```typescript
// src/env.d.ts
// Astro types — not necessary if you already have a tsconfig.json with the recommended `include`
/// <reference path="../.astro/types.d.ts" />

interface ImportMetaEnv {
  readonly PUBLIC_API_URL: string;
  readonly DATABASE_URL: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}

declare namespace App {
  interface Locals {
    user: {
      id: string;
      name: string;
    } | null;
  }
}
```

## Environment Variables

### .env Files

```bash
# .env
PUBLIC_API_URL=https://api.example.com
DATABASE_URL=postgresql://localhost/mydb
SECRET_KEY=abc123
```

### Access in Code

```astro
---
// Server-side (all variables)
const dbUrl = import.meta.env.DATABASE_URL;

// Client-side (only PUBLIC_ prefixed)
const apiUrl = import.meta.env.PUBLIC_API_URL;

// Built-in variables
const mode = import.meta.env.MODE;      // 'development' | 'production'
const prod = import.meta.env.PROD;      // boolean
const dev = import.meta.env.DEV;        // boolean
const site = import.meta.env.SITE;      // From astro.config site
const base = import.meta.env.BASE_URL;  // From astro.config base
---
```

### Type-Safe Environment

```javascript
// astro.config.mjs
import { defineConfig, envField } from 'astro/config';

export default defineConfig({
  env: {
    schema: {
      PUBLIC_API_URL: envField.string({
        context: 'client',
        access: 'public',
        default: 'https://api.example.com',
      }),
      DATABASE_URL: envField.string({
        context: 'server',
        access: 'secret',
      }),
      PORT: envField.number({
        context: 'server',
        access: 'public',
        default: 4321,
      }),
      FEATURE_FLAG: envField.boolean({
        context: 'client',
        access: 'public',
        default: false,
      }),
    },
  },
});
```

```astro
---
import { PUBLIC_API_URL, DATABASE_URL } from 'astro:env/server';
import { PUBLIC_API_URL } from 'astro:env/client';
---
```

## Session Configuration

```javascript
import { defineConfig, sessionDrivers } from 'astro/config';

export default defineConfig({
  session: {
    driver: sessionDrivers.redis({
      url: process.env.REDIS_URL,
    }),
    // Or use filesystem, memory, etc.
  },
});
```

## Experimental Features

```javascript
export default defineConfig({
  experimental: {
    contentIntellisense: true,
    clientPrerender: true,
  },
});
```

> **Note:** Server islands (`server:defer`) need no experimental flag.

## Full Example

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import react from '@astrojs/react';
import tailwindcss from '@tailwindcss/vite';
import mdx from '@astrojs/mdx';
import sitemap from '@astrojs/sitemap';
import vercel from '@astrojs/vercel';

export default defineConfig({
  site: 'https://mysite.com',

  output: 'static',  // default; opt routes into SSR with `export const prerender = false`
  adapter: vercel(), // only needed when some routes are server-rendered

  integrations: [
    react(),
    mdx(),
    sitemap(),
  ],

  prefetch: {
    prefetchAll: true,
  },

  i18n: {
    defaultLocale: 'en',
    locales: ['en', 'es'],
  },

  image: {
    domains: ['images.unsplash.com'],
  },

  markdown: {
    syntaxHighlight: 'shiki',
    shikiConfig: {
      theme: 'github-dark',
    },
  },

  vite: {
    plugins: [tailwindcss()],
    resolve: {
      alias: {
        '@': '/src',
      },
    },
  },
});
```

## Best Practices

1. **Start with strict TypeScript** - Better type safety
2. **Use path aliases** - Cleaner imports
3. **Configure image domains** - Security whitelist
4. **Set site URL** - Required for sitemaps and canonical URLs
5. **Use `output: 'static'` (the default)** - Opt individual routes out of prerendering with `export const prerender = false` for on-demand rendering; static + on-demand in one mode
6. **Enable prefetching** - Faster navigation
7. **Configure Markdown plugins** - Enhanced content
8. **Type your env variables** - Catch errors early
