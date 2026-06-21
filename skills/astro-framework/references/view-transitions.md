# View Transitions

Astro's View Transitions provide smooth navigation between pages without full page reloads.

## Setup

```astro
---
// src/layouts/Layout.astro
import { ClientRouter } from 'astro:transitions';
---

<!doctype html>
<html>
  <head>
    <ClientRouter />
  </head>
  <body>
    <slot />
  </body>
</html>
```

## How It Works

With `<ClientRouter />` enabled:
1. User clicks a link
2. Astro intercepts the navigation
3. New page content is fetched
4. Transitions animate between old and new content
5. Browser history is updated

## Built-in Animations

### fade (Default)

```astro
---
import { fade } from 'astro:transitions';
---

<div transition:animate={fade({ duration: '0.4s' })}>
  Content fades in and out
</div>
```

### slide

```astro
---
import { slide } from 'astro:transitions';
---

<div transition:animate={slide({ duration: '0.3s' })}>
  Content slides in from the side
</div>
```

### initial

Prevents animation on first page load:

```astro
<div transition:animate="initial">
  No animation on initial load
</div>
```

### none

Disables transition for an element:

```astro
<div transition:animate="none">
  Instant swap, no animation
</div>
```

## Transition Directives

### transition:name

Link elements across pages for morphing:

```astro
<!-- Page 1: List -->
<img
  src={post.image}
  transition:name={`hero-${post.slug}`}
/>

<!-- Page 2: Detail -->
<img
  src={post.data.image}
  transition:name={`hero-${post.slug}`}
/>
```

Elements with matching `transition:name` will morph into each other. Astro already auto-assigns a shared `view-transition-name` to elements it infers as matching pairs (by element type and DOM location), so `transition:name` is only needed to override that automatic matching.

### transition:animate

Control animation behavior:

```astro
<header transition:animate="none">
  <!-- Static header, no animation -->
</header>

<main transition:animate={slide({ duration: '0.5s' })}>
  <!-- Slides in -->
</main>
```

### transition:persist

Keep element state across navigations:

```astro
<!-- Video keeps playing during navigation -->
<video transition:persist autoplay>
  <source src="video.mp4" />
</video>

<!-- Form keeps user input -->
<form transition:persist>
  <input type="text" />
</form>

<!-- React component maintains state -->
<Counter client:load transition:persist />
```

With unique ID:

```astro
<audio transition:persist="player" controls>
  <source src="song.mp3" />
</audio>
```

As a convenient shorthand, `transition:persist` can take a transition name as its value. The name both persists the element and serves as the matching identifier across components.

By default, a persisted island keeps its state but re-renders with new props on navigation. Add `transition:persist-props` alongside `transition:persist` to keep the island's existing props (no re-render with new values) in addition to its state.

## Custom Animations

### Define with keyframes

```astro
---
import { slide } from 'astro:transitions';
---

<style>
  @keyframes slideInFromLeft {
    from { transform: translateX(-100%); opacity: 0; }
    to { transform: translateX(0); opacity: 1; }
  }

  @keyframes slideOutToRight {
    from { transform: translateX(0); opacity: 1; }
    to { transform: translateX(100%); opacity: 0; }
  }
</style>

<div
  transition:animate={{
    forwards: {
      old: { name: 'slideOutToRight', duration: '0.3s', easing: 'ease-in' },
      new: { name: 'slideInFromLeft', duration: '0.3s', easing: 'ease-out' },
    },
    backwards: {
      old: { name: 'slideOutToRight', duration: '0.3s', easing: 'ease-in' },
      new: { name: 'slideInFromLeft', duration: '0.3s', easing: 'ease-out' },
    },
  }}
>
  Custom sliding content
</div>
```

### Reusable Animation

```typescript
// src/transitions/custom.ts
export const customSlide = {
  forwards: {
    old: { name: 'slideOut', duration: '0.3s', easing: 'ease-in', fillMode: 'forwards' },
    new: { name: 'slideIn', duration: '0.3s', easing: 'ease-out', fillMode: 'backwards' },
  },
  backwards: {
    old: { name: 'slideOut', duration: '0.3s', easing: 'ease-in', fillMode: 'forwards' },
    new: { name: 'slideIn', duration: '0.3s', easing: 'ease-out', fillMode: 'backwards' },
  },
};
```

```astro
---
import { customSlide } from '../transitions/custom';
---

<div transition:animate={customSlide}>
  Uses custom animation
</div>
```

## Navigation Controls

### Programmatic Navigation

```astro
<script>
  import { navigate } from 'astro:transitions/client';

  // Navigate programmatically
  navigate('/about');

  // With history options: 'push' | 'replace' | 'auto'
  // 'push' creates a new entry, 'replace' updates the current URL
  // without a new entry, and 'auto' (default) attempts history.pushState
  // but keeps the current URL if the transition fails.
  navigate('/dashboard', { history: 'replace' });
</script>
```

### Forcing Full-Page Navigation

Add `data-astro-reload` to an `<a>` or `<form>` to opt out of client-side routing and perform a normal full-page browser navigation for that link/form.

```astro
<a href="/external" data-astro-reload>
  Full page navigation
</a>

<form data-astro-reload>
  Full-page navigation on submit
</form>
```

## Lifecycle Events

```astro
<script>
  document.addEventListener('astro:before-preparation', (event) => {
    // Before fetching new page
    console.log('Navigating to:', event.to);
  });

  document.addEventListener('astro:after-preparation', (event) => {
    // After fetching, before swap
  });

  document.addEventListener('astro:before-swap', (event) => {
    // Right before DOM swap
    // Can customize swap behavior
    event.swap = () => {
      // Custom swap logic
    };
  });

  document.addEventListener('astro:after-swap', (event) => {
    // After DOM swap, before animations
    // Good for re-initializing scripts
  });

  document.addEventListener('astro:page-load', (event) => {
    // Page fully loaded, animations complete
    // Runs on initial load and every navigation
  });
</script>
```

## Script Re-execution

Scripts with `data-astro-rerun` execute on every navigation:

```astro
<script data-astro-rerun>
  // This runs on initial load AND every navigation
  console.log('Page changed!');
  initializeComponent();
</script>
```

`data-astro-rerun` applies to inline scripts. Bundled module scripts (and any script Astro processes as a module) are only ever executed once and will not re-run on navigation; use the `astro:page-load` event to re-initialize logic that must run on every navigation.

## Form Handling

Forms work with view transitions:

```astro
<form method="POST" action="/api/submit">
  <input type="text" name="email" />
  <button type="submit">Subscribe</button>
</form>

<!-- Disable transitions for form -->
<form data-astro-reload method="POST">
  <!-- Full page reload on submit -->
</form>
```

## Fallback Behavior

Browsers without View Transitions API get:
- Full page navigation (no JavaScript errors)
- Graceful degradation

Control fallback behavior with the `fallback` prop on `<ClientRouter />` (`'animate'` is the default):

```astro
<ClientRouter fallback="animate" /> <!-- or "swap" | "none" -->
```

`ClientRouter` includes automatic support for `prefers-reduced-motion`: it disables all view transition animations when that media query is set, with no manual handling needed.

Check support:

```astro
<script>
  if (document.startViewTransition) {
    console.log('View Transitions supported!');
  }
</script>
```

## Configuration

### Disable for Specific Links

```astro
<!-- Skip view transitions -->
<a href="/page" data-astro-reload>Full Reload</a>
```

### Prefetching

```javascript
// astro.config.mjs
export default defineConfig({
  prefetch: {
    prefetchAll: true, // Prefetch all links on hover
    defaultStrategy: 'viewport', // or 'hover', 'load'
  },
});
```

```astro
<!-- Manual prefetch control -->
<a href="/page" data-astro-prefetch="hover">Prefetch on hover</a>
<a href="/page" data-astro-prefetch="viewport">Prefetch when visible</a>
<a href="/page" data-astro-prefetch="load">Prefetch immediately</a>
<a href="/page" data-astro-prefetch="false">Never prefetch</a>
```

## Common Patterns

### Persistent Navigation

```astro
<!-- Navigation stays in place -->
<nav transition:persist>
  <a href="/">Home</a>
  <a href="/about">About</a>
</nav>

<!-- Main content transitions -->
<main transition:animate={fade()}>
  <slot />
</main>
```

### Image Gallery

```astro
<!-- List page -->
{images.map((img) => (
  <a href={`/gallery/${img.id}`}>
    <img
      src={img.thumb}
      transition:name={`image-${img.id}`}
    />
  </a>
))}

<!-- Detail page -->
<img
  src={image.full}
  transition:name={`image-${image.id}`}
/>
```

## Best Practices

1. **Use transition:name for connected elements** - Creates smooth morphing
2. **Keep heavy components persistent** - Video players, iframes
3. **Re-run initialization scripts** - Use `data-astro-rerun`
4. **Handle loading states** - Show indicators during fetching
5. **Test without JavaScript** - Ensure graceful fallback
6. **Use prefetching wisely** - Balance speed vs bandwidth
7. **Avoid animating too many elements** - Can impact performance
