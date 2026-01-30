---
name: winui-xaml
description: "WinUI 3 and XAML best practices for layout, binding, async operations, collections, rendering, memory management, accessibility, and localization. Use when: (1) Working with WinUI 3 or Uno Platform XAML, (2) Optimizing UI performance, (3) Implementing data binding with x:Bind, (4) Managing collections and virtualization, (5) Ensuring accessibility compliance, (6) Handling async operations on UI thread, (7) Preventing memory leaks, (8) Implementing localization"
---

# WinUI 3 XAML Best Practices

Twelve categories of XAML best practices validated against WinUI 3 and Uno Platform, covering performance-critical patterns through polish optimizations.

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

## Categories by Impact

| # | Category | Impact | Key Topics |
|---|----------|--------|------------|
| 1 | [Layout](references/01-Layout.md) | HIGH | Panel selection, visual tree depth, AutoLayout, responsive design |
| 2 | [Binding](references/02-Binding.md) | HIGH | x:Bind vs Binding, modes, function binding |
| 3 | [Async & Threading](references/03-Async-Threading.md) | CRITICAL | UI thread, DispatcherQueue, parallelization |
| 4 | [Collections & Virtualization](references/04-Collections.md) | CRITICAL | ListView, ItemsRepeater, incremental loading |
| 5 | [Rendering & Composition](references/05-Rendering.md) | MEDIUM-HIGH | Composition APIs, animations, caching |
| 6 | [Memory Management](references/06-Memory.md) | HIGH | Event handlers, leaks, resource cleanup |
| 7 | [Accessibility](references/07-Accessibility.md) | HIGH | AutomationProperties, keyboard navigation |
| 8 | [Localization](references/08-Localization.md) | MEDIUM | x:Uid, .resw files, runtime switching |
| 9 | [Styles & Theming](references/09-Styles-Theming.md) | MEDIUM | ResourceDictionary, ThemeDictionaries, lightweight styling |
| 10 | [XAML Loading](references/10-XAML-Loading.md) | CRITICAL | x:Load, deferred loading, startup optimization |
| 11 | [Navigation](references/11-Navigation.md) | MEDIUM-HIGH | Frame caching, page lifecycle |
| 12 | [Input & Interaction](references/12-Input.md) | MEDIUM | Touch targets, gesture handling |

## Impact Levels

- **CRITICAL**: Can cause app crashes, multi-second freezes, or memory leaks
- **HIGH**: Noticeable performance degradation (100ms+)
- **MEDIUM-HIGH**: Cumulative impact across many elements
- **MEDIUM**: Optimization for polish and responsiveness

## Pattern Structure

Each reference file follows this format:

```markdown
### Skill Name
**Rule**: Concise imperative statement
**Why**: Technical rationale with measurable impact
**Example (XAML/C#)**: Correct implementation
**Common Mistakes**: Anti-patterns to avoid
**Uno Platform Notes**: Platform-specific gaps/workarounds (if applicable)
**Reference**: Microsoft Learn link
```

## WinUI 3 vs UWP Key Differences

| Feature | UWP | WinUI 3 |
|---------|-----|---------|
| Namespace | `Windows.UI.Xaml` | `Microsoft.UI.Xaml` |
| Dispatcher | `CoreDispatcher` | `DispatcherQueue` |
| Deferred Loading | `x:DeferLoadStrategy` (deprecated) | `x:Load` |
| Window | `CoreWindow` | `Microsoft.UI.Xaml.Window` |
| App Model | UWP sandbox | Desktop (full trust) |

## Uno Platform Compatibility

All patterns validated against Uno Platform with notes for:
- Platform-specific behavior differences
- Unsupported APIs and alternatives
- Skia renderer considerations

## Using References

Consult the individual reference files listed in the table above, matched to the task at hand. Each file is self-contained with rules, code examples, common mistakes, and Uno Platform compatibility notes.
