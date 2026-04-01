# Uno Platform Skills

Claude Code agent skills for Uno Platform, WinUI 3, and .NET development. Each skill encodes knowledge directly — no external doc fetching required at runtime.

## Example Prompt

> Use the winui-xaml, uno-platform-agent, and mvux skills to review our Uno Platform codebase for best practices. Identify issues across XAML, MVUX, navigation, performance, theming, accessibility, and cross-platform considerations. Propose concrete improvements prioritized by impact.

## Skills

### Consolidated Knowledge Hubs

These 4 hubs replaced ~70 individual doc-fetching skills. Each encodes complete API knowledge with correct/incorrect code patterns, common mistakes, and reference files.

| Skill | Refs | Description |
|---|---|---|
| [`mvux`](mvux/) | 6 | MVUX reactive architecture: IFeed/IState, FeedView, commands, selection, messaging, pagination |
| [`uno-navigation`](uno-navigation/) | 7 | Navigation Extensions: regions, routes, ViewMap/DataViewMap, qualifiers, NavigationView, TabBar, dialogs |
| [`uno-material`](uno-material/) | 5 | Material Design 3 theming: colors, typography, button/TextBox/FAB styles, lightweight styling, migration |
| [`uno-toolkit`](uno-toolkit/) | 7 | Toolkit controls: AutoLayout, SafeArea, Card, Chip, TabBar, NavigationBar, DrawerControl, extensions |

### Platform & Architecture Skills

| Skill | Refs | Description |
|---|---|---|
| [`uno-platform-agent`](uno-platform-agent/) | 8 | Comprehensive Uno Platform patterns: Single Project, MVVM/MVUX, navigation, styling, platform-specific code |
| [`winui-xaml`](winui-xaml/) | 12 | WinUI 3 XAML best practices: layout, binding, async, collections, rendering, memory, accessibility |
| [`dotnet-csharp`](dotnet-csharp/) | 5 | .NET C# best practices: performance, security, async/await, modern patterns |
| [`uno-extensions-services`](uno-extensions-services/) | 4 | Extensions: hosting, DI, authentication (MSAL/OIDC), HTTP clients, configuration, logging |
| [`uno-csharp-markup`](uno-csharp-markup/) | 4 | C# Markup: fluent API, strongly-typed binding, resources, styles, Toolkit integration |
| [`uno-wasm-pwa`](uno-wasm-pwa/) | 4 | WebAssembly and PWA: bootstrapper, manifests, debugging, hosting, performance |

### Migration & Troubleshooting

| Skill | Refs | Description |
|---|---|---|
| [`wpf-to-uno-migration`](wpf-to-uno-migration/) | 5 | WPF → Uno migration: namespace mapping, XAML replacements, NavigationView, settings, services |
| [`uno-migration-troubleshoot`](uno-migration-troubleshoot/) | 4 | Migration paths (Single Project, .NET upgrades, Silverlight) and common build/runtime errors |

### Testing

| Skill | Refs | Description |
|---|---|---|
| [`uno-app-ui-testing`](uno-app-ui-testing/) | 2 | Automated UI testing for Uno Platform apps via MCP tools |
| [`uno-app-test-assertions`](uno-app-test-assertions/) | 0 | Assertion and validation patterns for UI testing |

## Installation

Clone this repo, then symlink each skill into `~/.claude/skills/`:

```bash
# Linux/macOS
REPO_DIR="/path/to/UnoPlatformSkills"
for skill in "$REPO_DIR"/*/; do
  name=$(basename "$skill")
  ln -sf "$skill" ~/.claude/skills/"$name"
done
```

```powershell
# Windows (developer mode or admin)
$repoDir = "C:\path\to\UnoPlatformSkills"
Get-ChildItem -Directory $repoDir | ForEach-Object {
    $target = Join-Path $env:USERPROFILE ".claude\skills\$($_.Name)"
    if (Test-Path $target) { Remove-Item $target -Recurse -Force }
    New-Item -ItemType Junction -Path $target -Target $_.FullName
}
```

## Architecture

Each skill follows the **Archetype B (Technical/Knowledge)** template:

```
skill-name/
├── SKILL.md              # Hub: Quick Reference, Key Concepts, Critical Patterns, Common Mistakes
└── references/
    ├── 01-topic.md       # Deep-dive: Impact levels, CORRECT/WRONG code, edge cases
    ├── 02-topic.md
    └── ...
```

- **Hub** provides actionable guidance for common cases
- **References** are loaded on-demand for specific sub-topics
- **Docs Fallback** section in each hub provides MCP search queries when encoded knowledge may be outdated

## Related Resources

- [Uno Platform Documentation](https://platform.uno/docs/articles/intro.html)
- [Uno Toolkit](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/getting-started.html)
- [WinUI 3 Documentation](https://learn.microsoft.com/en-us/windows/apps/winui/winui3/)

## License

MIT
