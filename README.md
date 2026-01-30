# UnoPlatformSkills

Claude Code skills for Uno Platform development and WinUI 3 XAML best practices.

## Example Prompt: 

Use the winui-xaml and uno-platform-skills to review our Uno Platform codebase for best practices. Identify issues and opportunities across XAML, MVVM, navigation, performance, resource usage, theming, accessibility, and cross-platform considerations. Propose concrete improvements with rationale, prioritized by impact/effort, and include example code snippets or patch-style diffs for the highest-impact changes.

## Skills

### uno-platform-agent

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

### winui-xaml

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

## Installation

Symlink each skill directory into `~/.claude/skills/`:

```bash
# Linux/macOS
ln -s /path/to/uno-platform-agent-skills ~/.claude/skills/uno-platform-agent
ln -s /path/to/winui-xaml-skills ~/.claude/skills/winui-xaml
```

```powershell
# Windows (run as admin or with developer mode enabled)
mklink /D %USERPROFILE%\.claude\skills\uno-platform-agent C:\path\to\uno-platform-agent-skills
mklink /D %USERPROFILE%\.claude\skills\winui-xaml C:\path\to\winui-xaml-skills
```

## Related Resources

- [Uno Platform Documentation](https://platform.uno/docs/articles/intro.html)
- [Uno Toolkit](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/getting-started.html)
- [WinUI 3 Documentation](https://learn.microsoft.com/en-us/windows/apps/winui/winui3/)

## License

MIT
