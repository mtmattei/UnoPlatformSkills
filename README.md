# UnoSkills

Claude Code skills for Uno Platform development and WinUI 3 XAML best practices.

## WinUI 3 XAML Best Practices

Comprehensive, validated guidelines for building performant WinUI 3 applications with Uno Platform compatibility.

### Structure

```
WinUI-XAML-Skills/
├── 00-Index.md              # Quick reference & navigation
├── 01-Layout.md             # Panel selection, AutoLayout, visual tree
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
- **Reference**: Microsoft Learn or Uno Platform documentation link

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
10. Use `AutoLayout` with `Spacing` instead of margins on children

### Validation

All skills validated against:
- Microsoft Learn WinUI 3 documentation
- Uno Platform 5.6+ documentation
- Windows App SDK 1.5+ APIs

## Usage

These skills are designed to be loaded as context for Claude Code when working on Uno Platform projects.

Skills provide:
- Correct vs incorrect code examples
- Performance measurements and benchmarks
- Official documentation references
- Practical implementation patterns

## Related Resources

- [Uno Platform Documentation](https://platform.uno/docs/articles/intro.html)
- [Uno Toolkit](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/getting-started.html)
- [WinUI 3 Documentation](https://learn.microsoft.com/en-us/windows/apps/winui/winui3/)
- [Windows App SDK](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/)

## License

MIT
