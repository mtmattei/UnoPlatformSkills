---
name: uno-toolkit-controls
description: "Uno Toolkit UI controls and helpers for cross-platform apps: AutoLayout, Card, Chip, NavigationBar, SafeArea, TabBar, DrawerControl, and helper extensions (CommandExtensions, ResponsiveExtension, ItemsRepeaterExtensions). Use when: (1) Building responsive layouts with AutoLayout, (2) Adding NavigationBar or TabBar navigation chrome, (3) Handling safe areas on mobile, (4) Using Chip/ChipGroup for filters or tags, (5) Binding commands on ItemsRepeater, (6) Adapting UI to screen size with ResponsiveExtension. Do NOT use for: standard WinUI controls (see winui-xaml) or C# Markup equivalents (see uno-csharp-markup)."
license: "Apache 2.0 (patterns derived from Uno Platform documentation)"
metadata:
  version: "1.0.0"
---

# Uno Toolkit Controls and Helpers

Patterns for the Uno.Toolkit.UI library: controls, helpers, and responsive extensions that work cross-platform on iOS, Android, macOS, Linux, Windows, and WebAssembly.

## Installation

Add `Toolkit` to UnoFeatures in your `.csproj`:

```xml
<UnoFeatures>
    Material;
    Toolkit;
</UnoFeatures>
```

XAML namespace: `xmlns:utu="using:Uno.Toolkit.UI"`

## Controls

### AutoLayout

Bridges Figma Auto Layout to WinUI. Prefer over StackPanel for consistent spacing without per-child margins.

**Key Properties**: `Spacing`, `Padding`, `PrimaryAxisAlignment`, `CounterAxisAlignment`, `Orientation`, `Justify`, `IsReverseZIndex`

**Per-child overrides**: `AutoLayout.PrimaryAlignment`, `AutoLayout.CounterAlignment`, `AutoLayout.PrimaryLength`, `AutoLayout.CounterLength`

For XAML examples, see [references/01-layout-controls.md](references/01-layout-controls.md).

### NavigationBar

Cross-platform navigation bar with `MainCommand` (back button) and `PrimaryCommands`. Use with Uno Navigation Extensions for automatic back-button behavior.

For XAML examples, see [references/02-navigation-controls.md](references/02-navigation-controls.md).

### SafeArea

Handles notches, status bars, and rounded corners. Wrap page content to avoid overlap. **Modes**: `Padding` (default), `InsetMask`, `SoftInput` (keyboard adjustment).

For XAML examples, see [references/01-layout-controls.md](references/01-layout-controls.md).

### TabBar

Lateral navigation control. Integrates with Uno Navigation Extensions via `UseToolkitNavigation()`.

For XAML examples, see [references/02-navigation-controls.md](references/02-navigation-controls.md).

### Card, Chip, DrawerControl, LoadingView

- **Card / CardContentControl**: Material Design card surfaces with elevation styles
- **Chip / ChipGroup**: Compact elements for filters, tags, or selections (`SelectionMode`: Single/Multiple)
- **DrawerControl**: Hidden pane revealed by swipe (`OpenDirection`, `DrawerLength`)
- **LoadingView**: Async loading state indicator; pairs with MVUX Feeds

For XAML examples, see [references/03-data-controls.md](references/03-data-controls.md).

## Helpers

### ResponsiveExtension

Adapt property values based on screen width. Breakpoints: Narrowest (150), Narrow (300), Normal (600), Wide (800), Widest (1080).

```xml
<TextBlock Text="{utu:Responsive Narrow='Mobile View', Wide='Desktop View'}" />
<Grid ColumnSpacing="{utu:Responsive Narrow=8, Wide=24}" />
```

Custom breakpoints:
```xml
<Page.Resources>
    <utu:ResponsiveLayout x:Key="CustomLayout" Narrow="400" Wide="800" />
</Page.Resources>
<TextBlock Text="{utu:Responsive Layout={StaticResource CustomLayout},
                                Narrow=Small, Wide=Large}" />
```

### CommandExtensions

Attach commands to any control without code-behind.

```xml
<Border utu:CommandExtensions.Command="{Binding TapCommand}">
    <TextBlock Text="Tappable area" />
</Border>
```

### ItemsRepeaterExtensions

Add selection support to ItemsRepeater (which lacks it natively).

```xml
<muxc:ItemsRepeater ItemsSource="{Binding Items}"
                    utu:ItemsRepeaterExtensions.SelectedItem="{Binding SelectedItem, Mode=TwoWay}">
```

### StatusBar

Control mobile status bar appearance per page.

```xml
<Page utu:StatusBar.Foreground="Light"
      utu:StatusBar.Background="{ThemeResource PrimaryBrush}">
```

## Common Mistakes

- Using `StackPanel` with per-child `Margin` instead of `AutoLayout` with `Spacing`
- Forgetting `SafeArea` on pages with NavigationBar, causing content to render behind the notch
- Not calling `UseToolkitNavigation()` when using TabBar with Uno Navigation Extensions
- Hardcoding breakpoint values instead of using `ResponsiveExtension`
- Using `AppBarButton` outside of `CommandBar` - use regular `Button` with icon instead

## Related Skills

| Skill | Use instead when... |
|-------|-------------------|
| `winui-xaml` | Working with standard WinUI controls (Grid, ListView, StackPanel) or XAML best practices |
| `uno-csharp-markup` | Using Toolkit controls in C# Markup instead of XAML |
| `uno-platform-agent` | Setting up projects, MVVM/MVUX, or navigation routing |
| `uno-wasm-pwa` | Wasm-specific layout or performance concerns |

## Detailed References

- [references/01-layout-controls.md](references/01-layout-controls.md) - Read when using AutoLayout, SafeArea, or DrawerControl
- [references/02-navigation-controls.md](references/02-navigation-controls.md) - Read when using NavigationBar, TabBar, or TabBarItem
- [references/03-data-controls.md](references/03-data-controls.md) - Read when using Card, Chip, ChipGroup, or LoadingView
- [references/04-helpers-extensions.md](references/04-helpers-extensions.md) - Read when using ResponsiveExtension, CommandExtensions, ItemsRepeaterExtensions, or StatusBar
