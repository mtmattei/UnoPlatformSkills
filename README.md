# UnoSkills

Claude Code skills for Uno Platform development, WinUI 3 XAML best practices, and content marketing.

## WinUI 3 XAML Best Practices

Comprehensive, validated guidelines for building performant WinUI 3 applications with Uno Platform compatibility.

### Structure

```
WinUI-XAML-Skills/
├── 00-Index.md              # Quick reference & navigation
├── 01-Layout.md             # Panel selection, visual tree depth
├── 02-Binding.md            # x:Bind, modes, function binding
├── 03-Async-Threading.md    # UI thread, DispatcherQueue
├── 04-Collections.md        # Virtualization, ItemsRepeater
├── 05-Rendering.md          # Composition APIs, animations
├── 06-Memory.md             # Event cleanup, leak prevention
├── 07-Accessibility.md      # AutomationProperties, a11y
├── 08-Localization.md       # x:Uid, .resw resources
├── 09-Styles-Theming.md     # ResourceDictionary, themes
├── 10-XAML-Loading.md       # x:Load, deferred loading
├── 11-Navigation.md         # Frame, page lifecycle
├── 12-Input.md              # Touch targets, keyboard
├── 99-Fix-Log.md            # Change documentation
├── 99-Validation-Notes.md   # Uno Platform compatibility
└── 99-Open-Questions.md     # Unresolved items
```

### Skill Format

Each skill follows a consistent structure:

- **Rule**: Concise imperative statement
- **Why**: Technical rationale with measurable impact
- **Example**: Correct XAML/C# implementation
- **Common Mistakes**: Anti-patterns to avoid
- **Uno Platform Notes**: Platform-specific considerations
- **Reference**: Microsoft Learn documentation link

### Top 10 Rules

1. Never block the UI thread - use async/await
2. Use `x:Bind` over `{Binding}` - 3x faster, compile-time safe
3. Use `x:Load` for conditional UI - prevents element creation
4. Enable virtualization for lists > 20 items
5. Unsubscribe event handlers in `Unloaded`
6. Keep visual tree shallow (4-6 levels max)
7. Use `OneTime` binding mode for static data
8. Set `AutomationProperties` for accessibility
9. Use `x:Uid` for all user-visible text
10. Prefer `Grid` over nested `StackPanels`

### Validation

All skills validated against:
- Microsoft Learn WinUI 3 documentation
- Uno Platform 5.6+ documentation
- Windows App SDK 1.5+ APIs

## Content Marketing Skills

Skills for content creation and marketing workflows.

| Skill | Description |
|-------|-------------|
| [00-Skills-Index](00-Skills-Index.md) | Master index and orchestration guide |
| [01-Brand-Voice](01-Brand-Voice.md) | Brand voice and tone guidelines |
| [02-Positioning-Angles](02-Positioning-Angles.md) | Market positioning strategies |
| [03-Keyword-Research](03-Keyword-Research.md) | SEO keyword research |
| [04-Lead-Magnet](04-Lead-Magnet.md) | Lead magnet creation |
| [05-Direct-Response-Copy](05-Direct-Response-Copy.md) | Direct response copywriting |
| [06-SEO-Content](06-SEO-Content.md) | SEO content optimization |
| [07-Newsletter](07-Newsletter.md) | Newsletter writing |
| [08-Email-Sequences](08-Email-Sequences.md) | Email sequence design |
| [09-Content-Atomizer](09-Content-Atomizer.md) | Content repurposing |
| [10-Orchestrator](10-Orchestrator.md) | Workflow orchestration |
| [11-Trending-Topic-Finder](11-Trending-Topic-Finder.md) | Trend identification |

## Usage

These skills are designed to be loaded as context for Claude Code:

```bash
# Reference in your project's .claude/settings.json
# or load directly when working with Claude Code
```

Skills provide:
- Correct vs incorrect code examples
- Performance measurements and benchmarks
- Official documentation references
- Practical implementation patterns

## Related Resources

- [Uno Platform Documentation](https://platform.uno/docs/articles/intro.html)
- [WinUI 3 Documentation](https://learn.microsoft.com/en-us/windows/apps/winui/winui3/)
- [Windows App SDK](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/)
- [Claude Code](https://claude.ai/claude-code)

## License

MIT
