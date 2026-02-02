# UnoPlatformSkills

Claude Code skills for Uno Platform development and WinUI 3 XAML best practices.

## Example Prompt: 

Use the winui-xaml and uno-platform-skills to review our Uno Platform codebase for best practices. Identify issues and opportunities across XAML, MVVM, navigation, performance, resource usage, theming, accessibility, and cross-platform considerations. Propose concrete improvements with rationale, prioritized by impact/effort, and include example code snippets or patch-style diffs for the highest-impact changes.

## Skills

### Core Skills (Existing)

#### uno-platform-agent

Uno Platform development patterns for Single Project architecture, MVVM/MVUX, navigation, styling, platform-specific code, and custom controls. Extracted from uno.extensions (50+ modules) and 17+ production applications.

```
uno-platform-agent-skills/
├── SKILL.md
└── references/
    ├── 01-project-setup-critical.md
    ├── 02-build-configuration-critical.md
    ├── 03-mvvm-mvux-patterns-high.md
    ├── 04-navigation-high.md
    ├── 05-styling-theming-medium.md
    ├── 06-platform-specific-high.md
    ├── 07-data-http-medium.md
    └── 08-custom-controls-medium.md
```

#### winui-xaml

WinUI 3 and XAML best practices for layout, binding, async operations, collections, rendering, memory management, accessibility, and localization.

```
winui-xaml-skills/
├── SKILL.md
└── references/
    ├── 01-Layout.md
    ├── 02-Binding.md
    ├── 03-Async-Threading.md
    ├── 04-Collections.md
    ├── 05-Rendering.md
    ├── 06-Memory.md
    ├── 07-Accessibility.md
    ├── 08-Localization.md
    ├── 09-Styles-Theming.md
    ├── 10-XAML-Loading.md
    ├── 11-Navigation.md
    └── 12-Input.md
```

### Expansion Skills (P0)

#### uno-extensions-services

Uno Platform Extensions for hosting, DI, authentication (MSAL/OIDC), HTTP clients (Kiota/Refit), configuration, and logging.

```
uno-extensions-services-skills/
├── SKILL.md
└── references/
    ├── 01-hosting-di.md
    ├── 02-authentication.md
    ├── 03-http-clients.md
    └── 04-configuration-logging.md
```

#### uno-toolkit-controls

Uno Toolkit UI controls and helpers: AutoLayout, NavigationBar, SafeArea, TabBar, Card, Chip, DrawerControl, and extensions (CommandExtensions, ResponsiveExtension, ItemsRepeaterExtensions).

```
uno-toolkit-controls-skills/
├── SKILL.md
└── references/
    ├── 01-layout-controls.md
    ├── 02-navigation-controls.md
    ├── 03-data-controls.md
    └── 04-helpers-extensions.md
```

#### uno-wasm-pwa

WebAssembly and Progressive Web App development: bootstrapper, PWA manifests, debugging, hosting, performance, and deployment.

```
uno-wasm-pwa-skills/
├── SKILL.md
└── references/
    ├── 01-setup-bootstrapper.md
    ├── 02-debugging.md
    ├── 03-hosting-deployment.md
    └── 04-performance-pwa.md
```

#### uno-csharp-markup

C# Markup for code-first UI: fluent API, strongly-typed data binding, resources, styles, templates, and Uno Toolkit integration.

```
uno-csharp-markup-skills/
├── SKILL.md
└── references/
    ├── 01-getting-started.md
    ├── 02-binding-resources.md
    ├── 03-styles-templates.md
    └── 04-toolkit-integration.md
```

#### uno-migration-troubleshoot

Migration paths (Single Project, .NET upgrades, WPF/Silverlight) and troubleshooting common build/runtime errors across all platforms.

```
uno-migration-troubleshoot-skills/
├── SKILL.md
└── references/
    ├── 01-single-project-migration.md
    ├── 02-dotnet-version-upgrade.md
    ├── 03-wpf-silverlight-migration.md
    └── 04-common-errors.md
```

## Installation

Symlink each skill directory into `~/.claude/skills/`:

```bash
# Linux/macOS
ln -s /path/to/uno-platform-agent-skills ~/.claude/skills/uno-platform-agent
ln -s /path/to/winui-xaml-skills ~/.claude/skills/winui-xaml
ln -s /path/to/uno-extensions-services-skills ~/.claude/skills/uno-extensions-services
ln -s /path/to/uno-toolkit-controls-skills ~/.claude/skills/uno-toolkit-controls
ln -s /path/to/uno-wasm-pwa-skills ~/.claude/skills/uno-wasm-pwa
ln -s /path/to/uno-csharp-markup-skills ~/.claude/skills/uno-csharp-markup
ln -s /path/to/uno-migration-troubleshoot-skills ~/.claude/skills/uno-migration-troubleshoot
```

```powershell
# Windows (run as admin or with developer mode enabled)
mklink /D %USERPROFILE%\.claude\skills\uno-platform-agent C:\path\to\uno-platform-agent-skills
mklink /D %USERPROFILE%\.claude\skills\winui-xaml C:\path\to\winui-xaml-skills
mklink /D %USERPROFILE%\.claude\skills\uno-extensions-services C:\path\to\uno-extensions-services-skills
mklink /D %USERPROFILE%\.claude\skills\uno-toolkit-controls C:\path\to\uno-toolkit-controls-skills
mklink /D %USERPROFILE%\.claude\skills\uno-wasm-pwa C:\path\to\uno-wasm-pwa-skills
mklink /D %USERPROFILE%\.claude\skills\uno-csharp-markup C:\path\to\uno-csharp-markup-skills
mklink /D %USERPROFILE%\.claude\skills\uno-migration-troubleshoot C:\path\to\uno-migration-troubleshoot-skills
```

## Expansion Roadmap

See [SKILLS-EXPANSION-PLAN.md](SKILLS-EXPANSION-PLAN.md) for the full taxonomy, coverage matrix, and prioritized backlog.

### P1 (Should Have)
- **uno-platform-apis** - WinRT APIs, sensors, storage, geolocation
- **uno-publishing** - Store publishing, desktop and web deployment
- **uno-hot-design** - Hot Reload, Hot Design, Uno Studio tooling

### P2 (Nice to Have)
- **uno-figma-integration** - Figma plugin, design-to-code workflow
- **uno-maui-embedding** - MAUI embedding, third-party controls

## Related Resources

- [Uno Platform Documentation](https://platform.uno/docs/articles/intro.html)
- [Uno Toolkit](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/getting-started.html)
- [WinUI 3 Documentation](https://learn.microsoft.com/en-us/windows/apps/winui/winui3/)

## License

MIT
