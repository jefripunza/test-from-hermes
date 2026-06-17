---
name: frontend-performance
description: "Frontend performance expert: Core Web Vitals (LCP, INP, CLS), SSR/SSG, hydration, bundle analysis, tree shaking. Fast initial load, excellent UX."
metadata:
  hermes:
    tags: [frontend, performance, web-vitals, lcp, cls, ssr, hydration, bundle]
---

# Frontend Performance — Core Web Vitals Optimization

Fast initial load. Excellent user experience. Optimize Core Web Vitals.

## Core Web Vitals

### LCP (Largest Contentful Paint)
Target: **<2.5s**

Causes of poor LCP:
- Slow server response time (TTFB)
- Render-blocking resources
- Large hero images
- Slow resource load time

Fixes:
- Server-side rendering (SSR) or static generation (SSG)
- Optimize TTFB — CDN, edge compute, caching
- Preload hero images: `<link rel="preload" as="image">`
- Optimize images — WebP/AVIF, responsive sizes, compression
- Remove unused CSS/JS from critical path
- Use resource hints: `preconnect`, `prefetch`, `preload`

### INP (Interaction to Next Paint)
Target: **<200ms**

Causes of poor INP:
- Long tasks on main thread (>50ms)
- Heavy JS execution on interaction
- Layout thrashing from forced reflows

Fixes:
- Break long tasks with `setTimeout` / `requestIdleCallback`
- Debounce/throttle event handlers
- Use `content-visibility: auto` for off-screen content
- Avoid forced reflows — batch DOM reads/writes
- Use Web Workers for heavy computation
- Code splitting — load interaction handler JS lazily

### CLS (Cumulative Layout Shift)
Target: **<0.1**

Causes of poor CLS:
- Images without dimensions
- Ads, embeds without reserved space
- Dynamic content injected above existing content
- FOIT/FOUT from web fonts

Fixes:
- Always set `width`/`height` on images and videos
- Use `aspect-ratio` CSS property
- Reserve space for ads/embeds
- Font `font-display: swap` + preload
- Don't inject content above existing content
- Use `<link rel="preload">` for above-fold assets

## Rendering Strategies

| Strategy | LCP | INP | CLS | SEO | Use Case |
|----------|-----|-----|-----|-----|----------|
| CSR | Poor | Good | Good | Poor | Admin panels |
| SSR | Good | Good | Good | Good | Content-heavy, dynamic |
| SSG | Great | Good | Great | Great | Marketing, blog, docs |
| ISR | Great | Good | Great | Great | Semi-dynamic content |
| Partial HY | Good | Good | Good | Good | Hybrid apps |

### SSR / Hydration Considerations

- **Avoid full hydration** — use partial hydration (islands)
- **Streaming SSR** — send HTML progressively, faster TTFB
- **Dehydrate state** — pass serialized store to client
- **Waterfall** — avoid client → server → client data chain

## Bundle Analysis

```bash
# Vite bundle analysis
npm install -D vite-bundle-analyzer
# vite.config.ts
import { bundleAnalyzer } from 'vite-bundle-analyzer'
plugins: [bundleAnalyzer()]
```

### Tree Shaking

- Use named imports, not default imports of entire libs
- Avoid barrel files (`index.ts` that re-export everything)
- Configure `sideEffects: false` in package.json
- Dynamic imports: `import('heavy-component')`

### Asset Optimization

- Images: WebP/AVIF, responsive `srcset`, lazy loading
- Fonts: woff2, subset, `font-display: swap`, preload key fonts
- JS/CSS: minify, brotli compression, CDN cache
- Icons: inline SVGs or sprite sheet vs icon font

## Goal

- Fast initial load (LCP < 2.5s)
- Excellent user experience (INP < 200ms, CLS < 0.1)
- **Measure before optimize**
