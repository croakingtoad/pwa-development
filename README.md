# pwa-development

A [Claude Code skill](https://code.claude.com/docs/en/skills) for Progressive Web App development.

Covers service workers, caching strategies, offline support, web app manifests, push notifications, installation prompts, and platform-specific guidance for both Android and iOS.

## Installation

Copy the `skills/pwa-development/` directory into your project's `.claude/skills/` folder:

```
.claude/
  skills/
    pwa-development/
      SKILL.md
      reference/
        manifest-advanced.md
        workbox-and-caching.md
        mobile-native-ux.md
        platform-quirks.md
        push-notifications.md
        framework-integration.md
        testing-and-debugging.md
        anti-patterns.md
```

The skill activates automatically when you work with service worker files, Workbox configs, or web app manifests. You can also invoke it by asking Claude to "make a PWA", "add offline support", or "create a service worker".

## What's Included

**SKILL.md** (always loaded) contains the essentials: diagnostic states P0--P6 for assessing where an app stands, a caching strategy decision table, minimal manifest and service worker templates, install prompt code, and a launch checklist.

**Reference files** (loaded on demand) go deeper:

| File | Coverage |
|------|----------|
| `manifest-advanced.md` | Full manifest with screenshots, shortcuts, share\_target, icon size matrix |
| `workbox-and-caching.md` | Workbox setup, per-strategy code, background sync, offline detection |
| `mobile-native-ux.md` | Safe areas, touch targets, pull-to-refresh, native feel, desktop PWA |
| `platform-quirks.md` | iOS Safari and Android Chrome specifics, side by side |
| `push-notifications.md` | Permission, subscribe, push event, notification click |
| `framework-integration.md` | Next.js, CRA, Vite, Webpack, Nuxt, SvelteKit setup |
| `testing-and-debugging.md` | Lighthouse, DevTools checklist, performance targets |
| `anti-patterns.md` | Named anti-patterns and common mistakes |

## Sources

This skill consolidates content from three community PWA skills:

- **[jwynia/agent-skills](https://github.com/jwynia/agent-skills)** -- Diagnostic P0--P6 operating model, anti-patterns, debugging checklist
- **[sebastiaanwouters/dotagents](https://github.com/sebastiaanwouters/dotagents)** (`skills/pwa/`) -- Essential HTML head, safe-area handling, touch targets, display-mode detection, iOS status bar, testing snippets
- **[alinaqi/maggy](https://github.com/alinaqi/maggy)** (MIT) -- Three Pillars structure, full manifest examples, Workbox config, push notifications, background sync, framework guides, common mistakes table

Content was extracted, deduplicated, merged where overlapping, and rebalanced for cross-platform Android/iOS coverage. New material (marked with `<!-- NEW -->` comments in the source files) was added to fill gaps -- primarily Android-specific guidance, desktop PWA support, and additional framework configs. See [SOURCES.md](SOURCES.md) for detailed attribution and license information.

## License

This skill is released under the [MIT License](LICENSE). The alinaqi/maggy source is MIT-licensed. The jwynia/agent-skills and sebastiaanwouters/dotagents sources have no license specified in their repositories.
