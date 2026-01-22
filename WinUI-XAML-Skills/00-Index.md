# WinUI 3 XAML Best Practices - Skills Index

> **Scope**: WinUI 3 (Windows App SDK) with Uno Platform compatibility notes
> **Last Validated**: 2026-01-22

## Categories

| # | Category | Impact | File |
|---|----------|--------|------|
| 1 | [Layout](./01-Layout.md) | HIGH | Panel selection, visual tree depth, AutoLayout, responsive design |
| 2 | [Binding](./02-Binding.md) | HIGH | x:Bind vs Binding, modes, function binding |
| 3 | [Async & Threading](./03-Async-Threading.md) | CRITICAL | UI thread, DispatcherQueue, parallelization |
| 4 | [Collections & Virtualization](./04-Collections.md) | CRITICAL | ListView, ItemsRepeater, incremental loading |
| 5 | [Rendering & Composition](./05-Rendering.md) | MEDIUM-HIGH | Composition APIs, animations, caching |
| 6 | [Memory Management](./06-Memory.md) | HIGH | Event handlers, leaks, resource cleanup |
| 7 | [Accessibility](./07-Accessibility.md) | HIGH | AutomationProperties, keyboard navigation |
| 8 | [Localization](./08-Localization.md) | MEDIUM | x:Uid, .resw files, runtime switching |
| 9 | [Styles & Theming](./09-Styles-Theming.md) | MEDIUM | ResourceDictionary, ThemeDictionaries, lightweight styling |
| 10 | [XAML Loading](./10-XAML-Loading.md) | CRITICAL | x:Load, deferred loading, startup optimization |
| 11 | [Navigation](./11-Navigation.md) | MEDIUM-HIGH | Frame caching, page lifecycle |
| 12 | [Input & Interaction](./12-Input.md) | MEDIUM | Touch targets, gesture handling |

## Document Structure

Each skill follows this format:

```markdown
### Skill Name
**Rule**: Concise imperative statement
**Why**: Technical rationale with measurable impact
**Example (XAML/C#)**: Correct implementation
**Common Mistakes**: Anti-patterns to avoid
**Uno Platform Notes**: Platform-specific gaps/workarounds (if applicable)
**Reference**: Microsoft Learn link
```

## Impact Levels

- **CRITICAL**: Can cause app crashes, multi-second freezes, or memory leaks
- **HIGH**: Noticeable performance degradation (100ms+)
- **MEDIUM-HIGH**: Cumulative impact across many elements
- **MEDIUM**: Optimization for polish and responsiveness
- **LOW-MEDIUM**: Minor improvements, best practices

## Quick Reference - Top 10 Rules

1. **Never block the UI thread** - Use async/await for all I/O and long operations
2. **Use x:Bind over Binding** - Compile-time validation, 3x faster
3. **Use x:Load for conditional UI** - Prevents element creation until needed
4. **Enable virtualization** - ListView/ItemsRepeater for any list > 20 items
5. **Unsubscribe event handlers** - Clean up in Unloaded to prevent leaks
6. **Keep visual tree shallow** - Minimize nesting depth
7. **Use OneTime mode for static data** - Avoid unnecessary change tracking
8. **Set AutomationProperties** - Required for accessibility compliance
9. **Use x:Uid for all user-visible text** - Enables localization
10. **Prefer Grid over nested StackPanels** - Single layout pass vs multiple

## WinUI 3 vs UWP Key Differences

| Feature | UWP | WinUI 3 |
|---------|-----|---------|
| Namespace | `Windows.UI.Xaml` | `Microsoft.UI.Xaml` |
| Dispatcher | `CoreDispatcher` | `DispatcherQueue` |
| Deferred Loading | `x:DeferLoadStrategy` (deprecated) | `x:Load` |
| Window | `CoreWindow` | `Microsoft.UI.Xaml.Window` |
| App Model | UWP sandbox | Desktop (full trust) |

## Uno Platform Compatibility

All skills are validated against Uno Platform with notes for:
- Platform-specific behavior differences
- Unsupported APIs and alternatives
- Skia renderer considerations
