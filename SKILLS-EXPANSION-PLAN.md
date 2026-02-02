# Skills Catalog Expansion Plan

Generated: 2026-01-30

## 1. Taxonomy of Uno Platform Docs Topics (from sitemap)

1,253 URLs clustered into 22 developer-focused categories:

| # | Category | URLs | Description |
|---|----------|------|-------------|
| 1 | Getting Started / Project Setup | ~41 | Templates, wizard, solution structure |
| 2 | IDE Setup / AI Agent Tooling | ~20 | VS2022, Rider, VS Code, Claude, Cursor |
| 3 | WinUI Controls - Implementation Ref | ~165 | Per-control implementation status |
| 4 | Controls - Usage Guides | ~20 | ListView, CommandBar, NavigationView, WebView |
| 5 | MVUX Architecture | ~40 | Feeds, States, ListFeeds, Commands, Selection |
| 6 | Navigation and Routing | ~45 | Routes, Regions, TabBar, Navigator, Dialogs |
| 7 | Styling, Theming, Material Design | ~55 | Material theme, colors, lightweight styling |
| 8 | Uno Toolkit Controls / Helpers | ~65 | AutoLayout, SafeArea, NavigationBar, helpers |
| 9 | C# Markup (Code-first UI) | ~30 | Fluent C# API, binding, styles, templates |
| 10 | Figma Design-to-Code | ~95 | Plugin, components, designer workflow |
| 11 | Platform Features / WinRT APIs | ~100 | Storage, geolocation, clipboard, sensors |
| 12 | Extensions (Auth, HTTP, Config, DI) | ~50 | MSAL, OIDC, Kiota, Refit, configuration |
| 13 | WebAssembly / PWA | ~40 | Bootstrapper, PWA, debugging, hosting |
| 14 | Samples and Sample Apps | ~180 | ChatGPT, Finance, TubePlayer, etc. |
| 15 | Workshops and Tutorials | ~55 | Counter app, Calculator, TubePlayer workshop |
| 16 | Uno Chefs (Reference Architecture) | ~50 | Production patterns, MVUX, navigation, toolkit |
| 17 | Migration and Upgrading | ~30 | Uno 6, .NET 9, Single Project, WPF/Silverlight |
| 18 | Troubleshooting | ~20 | Common issues (Android, Wasm, VS2022, builds) |
| 19 | Publishing and Deployment | ~15 | Android, iOS, Wasm, Desktop publishing |
| 20 | Hot Design / Uno Studio | ~20 | Hot Reload, Hot Design, properties, toolbox |
| 21 | Contributing / Internals | ~40 | Building Uno, runtime tests, code style |
| 22 | MAUI Embedding / Third-Party | ~25 | DevExpress, Telerik, Syncfusion, Esri |

---

## 2. Coverage Matrix: Topic to Existing Skills to Gaps

| Taxonomy Topic | uno-platform-agent | winui-xaml | Coverage Level |
|----------------|-------------------|------------|----------------|
| 1. Getting Started | ref-01, ref-02 | - | PARTIAL (setup only, no wizard/template selection) |
| 2. IDE Setup / AI Tooling | - | - | NONE |
| 3. WinUI Controls Impl | ref-08 (custom only) | ref-01, ref-04, ref-12 | WEAK (layout/collections/input, not control API ref) |
| 4. Controls Usage Guides | - | ref-04 (ListView only) | WEAK |
| 5. MVUX Architecture | ref-03 (overview) | - | PARTIAL (basics, not advanced patterns) |
| 6. Navigation/Routing | ref-04 | ref-11 | GOOD |
| 7. Styling/Theming/Material | ref-05 | ref-09 | GOOD |
| 8. Uno Toolkit Controls | ref-05 (AutoLayout only) | - | WEAK (65 URLs, 1 control covered) |
| 9. C# Markup | - | - | NONE |
| 10. Figma Design-to-Code | - | - | NONE |
| 11. Platform Features/WinRT | ref-06 (directives only) | - | WEAK (compilation, not API usage) |
| 12. Extensions (Auth/HTTP/DI) | ref-07 (HTTP overview) | - | PARTIAL (HTTP only, no Auth/Config/DI depth) |
| 13. WebAssembly / PWA | ref-06 (brief mention) | - | NONE |
| 14. Samples | - | - | N/A (educational, not skill material) |
| 15. Workshops/Tutorials | - | - | N/A (educational, not skill material) |
| 16. Uno Chefs | - | - | NONE (reference architecture) |
| 17. Migration/Upgrading | - | - | NONE |
| 18. Troubleshooting | - | - | NONE |
| 19. Publishing/Deployment | - | - | NONE |
| 20. Hot Design/Studio | - | - | NONE |
| 21. Contributing/Internals | - | - | N/A (contributor-only) |
| 22. MAUI Embedding | - | - | NONE |

### Coverage Summary

- **GOOD (2)**: Navigation, Styling/Theming
- **PARTIAL (3)**: Getting Started, MVUX, Extensions
- **WEAK (4)**: WinUI Controls, Controls Usage, Toolkit, Platform Features
- **NONE (8)**: IDE/AI Tooling, C# Markup, Figma, WebAssembly, Migration, Troubleshooting, Publishing, Hot Design, MAUI Embedding
- **N/A (3)**: Samples, Workshops, Contributing

---

## 3. Proposed Skills Backlog (Prioritized)

Scoring: Impact (1-5) x Frequency (1-5) x Inverse Effort (1-5, where 5=easy to create)

### P0 - Must Have

| Skill | Impact | Freq | Effort | Score | Covers Topics |
|-------|--------|------|--------|-------|---------------|
| **uno-extensions-services** | 5 | 5 | 3 | 75 | 12 (Auth, HTTP, Config, DI, Logging) |
| **uno-toolkit-controls** | 5 | 5 | 3 | 75 | 8 (AutoLayout, SafeArea, NavigationBar, TabBar, helpers) |
| **uno-wasm-pwa** | 4 | 4 | 4 | 64 | 13 (Wasm bootstrap, PWA, debugging, hosting) |
| **uno-csharp-markup** | 4 | 3 | 4 | 48 | 9 (Fluent C# API, binding, styles, templates) |
| **uno-migration-troubleshoot** | 5 | 3 | 3 | 45 | 17+18 (Migration paths, common issues, build fixes) |

### P1 - Should Have

| Skill | Impact | Freq | Effort | Score | Covers Topics |
|-------|--------|------|--------|-------|---------------|
| **uno-platform-apis** | 4 | 3 | 3 | 36 | 11 (WinRT APIs, sensors, storage, geolocation) |
| **uno-publishing** | 5 | 2 | 4 | 40 | 19 (Store publishing, desktop, web deployment) |
| **uno-hot-design** | 3 | 4 | 4 | 48 | 20 (Hot Reload, Hot Design, Uno Studio tooling) |

### P2 - Nice to Have

| Skill | Impact | Freq | Effort | Score | Covers Topics |
|-------|--------|------|--------|-------|---------------|
| **uno-figma-integration** | 3 | 2 | 3 | 18 | 10 (Figma plugin, design-to-code workflow) |
| **uno-maui-embedding** | 3 | 2 | 4 | 24 | 22 (MAUI embedding, third-party controls) |

---

## 4. Roadmap

**Phase 1 (P0)** - 5 skills to write now
1. uno-extensions-services
2. uno-toolkit-controls
3. uno-wasm-pwa
4. uno-csharp-markup
5. uno-migration-troubleshoot

**Phase 2 (P1)** - 3 skills
6. uno-platform-apis
7. uno-publishing
8. uno-hot-design

**Phase 3 (P2)** - 2 skills
9. uno-figma-integration
10. uno-maui-embedding

---

## 5. Unresolved Questions

1. Should C# Markup skill cover MVUX-specific C# Markup bindings, or keep those in a separate MVUX-deep skill?
2. Should Figma integration be a standalone skill or folded into styling/theming?
3. How deep should the WinRT APIs skill go - per-API patterns, or just the cross-platform compatibility layer?
4. Should Uno Chefs reference architecture be its own skill or distributed across existing skills as examples?
5. Is there demand for an "Uno for WPF/Silverlight developers" migration skill separate from general migration?
