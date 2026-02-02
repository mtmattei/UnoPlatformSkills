---
name: uno-toolkit-controls
description: "Uno Toolkit UI controls and helpers for cross-platform apps: AutoLayout, Card, Chip, NavigationBar, SafeArea, TabBar, DrawerControl, and helper extensions (CommandExtensions, ResponsiveExtension, ItemsRepeaterExtensions). Use when: (1) Building responsive layouts with AutoLayout, (2) Adding NavigationBar or TabBar navigation chrome, (3) Handling safe areas on mobile, (4) Using Chip/ChipGroup for filters or tags, (5) Binding commands on ItemsRepeater, (6) Adapting UI to screen size with ResponsiveExtension"
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

```xml
<utu:AutoLayout Spacing="16" Padding="24"
                PrimaryAxisAlignment="Center"
                Orientation="Vertical">
    <TextBlock Text="Title" Style="{StaticResource TitleLarge}" />
    <TextBlock Text="Subtitle" Style="{StaticResource BodyMedium}" />
</utu:AutoLayout>
```

**Key Properties**: `Spacing`, `Padding`, `PrimaryAxisAlignment`, `CounterAxisAlignment`, `Orientation`, `Justify` (stretch children to fill), `IsReverseZIndex`

**Per-child overrides**: `AutoLayout.PrimaryAlignment`, `AutoLayout.CounterAlignment`, `AutoLayout.PrimaryLength`, `AutoLayout.CounterLength`

### NavigationBar

Cross-platform navigation bar. Use with Uno Navigation Extensions for automatic back-button behavior.

```xml
<utu:NavigationBar Content="Page Title">
    <utu:NavigationBar.MainCommand>
        <AppBarButton>
            <AppBarButton.Icon>
                <BitmapIcon UriSource="ms-appx:///Assets/Icons/back.png" />
            </AppBarButton.Icon>
        </AppBarButton>
    </utu:NavigationBar.MainCommand>
    <utu:NavigationBar.PrimaryCommands>
        <AppBarButton Icon="Setting" Label="Settings" />
    </utu:NavigationBar.PrimaryCommands>
</utu:NavigationBar>
```

### SafeArea

Handles notches, status bars, and rounded corners on mobile devices. Wrap page content to avoid overlap.

```xml
<utu:SafeArea Insets="Top,Bottom">
    <Grid>
        <!-- Page content here -->
    </Grid>
</utu:SafeArea>
```

**Modes**: `Padding` (default, adds padding), `InsetMask` (clips to safe area), `SoftInput` (adjusts for keyboard)

### TabBar

Lateral navigation control. Integrates with Uno Navigation Extensions via `UseToolkitNavigation()`.

```xml
<utu:TabBar SelectedIndex="0">
    <utu:TabBarItem Content="Home">
        <utu:TabBarItem.Icon>
            <FontIcon Glyph="&#xE80F;" />
        </utu:TabBarItem.Icon>
    </utu:TabBarItem>
    <utu:TabBarItem Content="Search">
        <utu:TabBarItem.Icon>
            <FontIcon Glyph="&#xE721;" />
        </utu:TabBarItem.Icon>
    </utu:TabBarItem>
</utu:TabBar>
```

### Card and CardContentControl

Material Design card surfaces. Use `CardContentControl` for actionable cards with elevation.

```xml
<utu:CardContentControl Style="{StaticResource ElevatedCardContentControlStyle}">
    <StackPanel Padding="16" Spacing="8">
        <TextBlock Text="Card Title" Style="{StaticResource TitleMedium}" />
        <TextBlock Text="Card content goes here" Style="{StaticResource BodyMedium}" />
    </StackPanel>
</utu:CardContentControl>
```

### Chip and ChipGroup

Compact interactive elements for filters, tags, or selections.

```xml
<utu:ChipGroup SelectionMode="Multiple"
               ItemsSource="{Binding FilterOptions}"
               SelectedItems="{Binding SelectedFilters, Mode=TwoWay}">
    <utu:ChipGroup.ChipTemplate>
        <DataTemplate>
            <utu:Chip Content="{Binding Name}" />
        </DataTemplate>
    </utu:ChipGroup.ChipTemplate>
</utu:ChipGroup>
```

### DrawerControl

Hidden pane revealed by swipe gesture.

```xml
<utu:DrawerControl OpenDirection="Left" DrawerLength="300">
    <utu:DrawerControl.DrawerContent>
        <Grid Background="{ThemeResource SurfaceBrush}">
            <!-- Drawer menu -->
        </Grid>
    </utu:DrawerControl.DrawerContent>
    <!-- Main page content -->
</utu:DrawerControl>
```

### LoadingView

Indicates async loading state. Use with MVUX Feeds for automatic loading/error/data states.

```xml
<utu:LoadingView Source="{Binding DataFeed}">
    <utu:LoadingView.LoadingContent>
        <ProgressRing IsActive="True" />
    </utu:LoadingView.LoadingContent>
    <DataTemplate>
        <TextBlock Text="{Binding}" />
    </DataTemplate>
</utu:LoadingView>
```

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

## Detailed References

- [references/01-layout-controls.md](references/01-layout-controls.md) - Read when using AutoLayout, SafeArea, or DrawerControl
- [references/02-navigation-controls.md](references/02-navigation-controls.md) - Read when using NavigationBar, TabBar, or TabBarItem
- [references/03-data-controls.md](references/03-data-controls.md) - Read when using Card, Chip, ChipGroup, or LoadingView
- [references/04-helpers-extensions.md](references/04-helpers-extensions.md) - Read when using ResponsiveExtension, CommandExtensions, ItemsRepeaterExtensions, or StatusBar
