# Vue Desktop

Experimental Open Web Desktop starter using Vue 3 and Vite. **Open About** creates an engine window that can be moved, snapped, resized and closed.

```sh
cd template
pnpm install
pnpm dev
```

Use `pnpm build` for static output. Vue components read published core snapshots through `shallowRef` and call the core pointer APIs. The current separate Vue adapter re-exports Nuxt APIs, so this starter binds directly to the core.

`template/` is self-contained and runs with project scripts. It is a local experiment.
