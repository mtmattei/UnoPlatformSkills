# Uno Toolkit Layout Controls Reference

## Prerequisites

```xml
xmlns:utu="using:Uno.Toolkit.UI"
```

```xml
<UnoFeatures>Toolkit;Material;</UnoFeatures>
```

---

### AutoLayout Over StackPanel
**Rule**: Use `AutoLayout` instead of `StackPanel` for all single-axis layouts when Uno Toolkit is present.
**Why**: AutoLayout mirrors Figma Auto Layout behavior, providing built-in Spacing, Padding, axis alignment, and per-child overrides in a single control. This eliminates the need to set Margin on individual children (which breaks spacing consistency) and enables direct Figma-to-XAML translation.
**Example (XAML)**:
```xml
<utu:AutoLayout Orientation="Horizontal"
                Spacing="16"
                Padding="24"
                PrimaryAxisAlignment="Center"
                CounterAxisAlignment="Center">
    <TextBlock Text="Label" Style="{StaticResource TitleMedium}" />
    <Button Content="Action" />
</utu:AutoLayout>
```
**Common Mistakes**:
- Setting `Margin` on children inside an AutoLayout. Use `Spacing` and `Padding` on the AutoLayout container instead.
- Using `StackPanel` with manual Margin on each child to simulate spacing. AutoLayout handles this with a single `Spacing` property.
- Nesting StackPanels when a single AutoLayout with proper axis alignment would suffice.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/AutoLayoutControl.html

---

### AutoLayout Per-Child Overrides
**Rule**: Use AutoLayout attached properties to override alignment and sizing for individual children without breaking the parent layout flow.
**Why**: Per-child overrides (`AutoLayout.PrimaryAlignment`, `AutoLayout.CounterAlignment`, `AutoLayout.PrimaryLength`, `AutoLayout.CounterLength`) let you deviate from the parent's alignment for specific children. This matches Figma's per-child "Resizing" and "Alignment" options, enabling faithful design reproduction without wrapper elements.
**Example (XAML)**:
```xml
<utu:AutoLayout Orientation="Vertical"
                Spacing="12"
                Padding="16">
    <!-- This child stretches to fill the cross-axis -->
    <TextBox PlaceholderText="Full-width input"
             utu:AutoLayout.CounterAlignment="Stretch" />

    <!-- This child has a fixed primary length (height in vertical orientation) -->
    <Border Background="{ThemeResource SurfaceVariantBrush}"
            utu:AutoLayout.PrimaryLength="200"
            utu:AutoLayout.CounterAlignment="Stretch" />

    <!-- This child overrides alignment to end -->
    <Button Content="Submit"
            utu:AutoLayout.PrimaryAlignment="End" />
</utu:AutoLayout>
```
**Common Mistakes**:
- Setting explicit `Width`/`Height` on children when `PrimaryLength`/`CounterLength` attached properties exist for this purpose.
- Forgetting that `PrimaryLength` refers to the dimension along the `Orientation` axis (Height when Vertical, Width when Horizontal).
- Wrapping a child in an extra Grid just to change alignment. Use the attached properties directly.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/AutoLayoutControl.html

---

### AutoLayout Justify (Space Between)
**Rule**: Set `Justify="SpaceBetween"` to distribute children evenly along the primary axis, matching Figma's "Space Between" packing mode.
**Why**: This eliminates manual spacing calculations for evenly distributed layouts such as navigation bars, toolbars, or stat rows. The control automatically distributes remaining space between children.
**Example (XAML)**:
```xml
<utu:AutoLayout Orientation="Horizontal"
                Justify="SpaceBetween"
                Padding="16,0"
                CounterAxisAlignment="Center">
    <TextBlock Text="Start" Style="{StaticResource BodyLarge}" />
    <TextBlock Text="Center" Style="{StaticResource BodyLarge}" />
    <TextBlock Text="End" Style="{StaticResource BodyLarge}" />
</utu:AutoLayout>
```
**Common Mistakes**:
- Setting both `Justify="SpaceBetween"` and a `Spacing` value. When Justify is SpaceBetween, Spacing is ignored because the layout distributes all remaining space.
- Using Grid with three columns (Auto, *, Auto) when AutoLayout with Justify achieves the same result with cleaner markup.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/AutoLayoutControl.html

---

### SafeArea Insets and Modes
**Rule**: Apply `SafeArea.Insets` to edge-touching controls to prevent content from being obscured by notches, status bars, or rounded corners. Choose `Padding` mode (default) to bleed backgrounds into unsafe areas, or `Margin` mode to push the entire control inward.
**Why**: Mobile devices have irregular screen shapes. Without SafeArea, interactive content or text can appear behind notches or below the navigation bar. Padding mode preserves background color bleeding (preferred for bars), while Margin mode shifts the entire element.
**Example (XAML)**:
```xml
<!-- Attached property form (preferred for most cases) -->
<Grid utu:SafeArea.Insets="Top,Bottom">
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto" />
        <RowDefinition Height="*" />
        <RowDefinition Height="Auto" />
    </Grid.RowDefinitions>

    <!-- Top bar with background bleeding into status bar area -->
    <utu:NavigationBar Content="My Page"
                       Background="{ThemeResource PrimaryBrush}"
                       utu:SafeArea.Insets="Top" />

    <!-- Page content -->
    <ScrollViewer Grid.Row="1">
        <!-- ... -->
    </ScrollViewer>

    <!-- Bottom bar with background bleeding into navigation bar area -->
    <utu:TabBar Grid.Row="2"
                Background="{ThemeResource SurfaceBrush}"
                utu:SafeArea.Insets="Bottom" />
</Grid>
```
**Common Mistakes**:
- Applying SafeArea.Insets to both a parent and a child for the same edge, causing double padding.
- Using `Mode="Margin"` on a colored bar, which leaves a gap between the bar and the screen edge instead of bleeding the background color.
- Forgetting `Insets="SoftInput"` on pages with text entry, causing the keyboard to cover input fields.
**Uno Platform Notes**: SafeArea with `SoftInput` introduces a ScrollViewer into the visual tree. On Android, set `WindowSoftInputMode` to `adjustNothing` to avoid conflicts between SafeArea's keyboard handling and the OS-level adjustment.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/SafeArea.html

---

### SafeArea SoftInput for Keyboards
**Rule**: Wrap pages containing text input with `SafeArea` using `Insets="SoftInput"` to keep input fields visible when the on-screen keyboard appears.
**Why**: On mobile, the on-screen keyboard can cover input fields, making forms unusable. SafeArea with SoftInput monitors keyboard state and adjusts content layout to keep focused inputs visible, including scrolling support.
**Example (XAML)**:
```xml
<Page xmlns:utu="using:Uno.Toolkit.UI">
    <utu:SafeArea Insets="SoftInput">
        <Grid Padding="24,0">
            <Grid.RowDefinitions>
                <RowDefinition Height="Auto" />
                <RowDefinition Height="*" />
                <RowDefinition Height="Auto" />
            </Grid.RowDefinitions>

            <Image Source="ms-appx:///Assets/Logo.png"
                   Height="120"
                   Stretch="Uniform" />

            <!-- Spacer -->
            <Border Grid.Row="1" />

            <utu:AutoLayout Grid.Row="2"
                            Orientation="Vertical"
                            Spacing="12"
                            Padding="0,0,0,24">
                <TextBox x:Uid="LoginPage.TextBox.Username"
                         PlaceholderText="Username" />
                <PasswordBox x:Uid="LoginPage.PasswordBox.Password"
                             PlaceholderText="Password" />
                <Button Content="Sign In"
                        Style="{StaticResource FilledButtonStyle}"
                        utu:AutoLayout.CounterAlignment="Stretch" />
            </utu:AutoLayout>
        </Grid>
    </utu:SafeArea>
</Page>
```
**Common Mistakes**:
- Using SafeArea SoftInput as an attached property instead of a control. The control form is strongly recommended because it inserts a ScrollViewer automatically.
- Combining SafeArea SoftInput with Android's `adjustResize` or `adjustPan` WindowSoftInputMode, which causes double adjustments.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/SafeArea.html

---

### DrawerControl for Swipe-Revealed Panels
**Rule**: Use `DrawerControl` for panels that reveal via swipe gesture. Set `OpenDirection` to control which edge the drawer slides from, and use `DrawerLength` to define the revealed size.
**Why**: DrawerControl provides native swipe gesture support for hamburger menus, filter panels, or action sheets. It integrates directly with NavigationView via `DrawerNavigationViewStyle`, adding gesture-based open/close to the standard pane.
**Example (XAML)**:
```xml
<utu:DrawerControl OpenDirection="Left"
                   DrawerLength="320">
    <utu:DrawerControl.Content>
        <Grid>
            <TextBlock Text="Main Content"
                       HorizontalAlignment="Center"
                       VerticalAlignment="Center"
                       Style="{StaticResource HeadlineMedium}" />
        </Grid>
    </utu:DrawerControl.Content>
    <utu:DrawerControl.DrawerContent>
        <Grid Background="{ThemeResource SurfaceBrush}"
              Padding="16">
            <utu:AutoLayout Orientation="Vertical" Spacing="8">
                <TextBlock Text="Menu" Style="{StaticResource TitleLarge}" />
                <Button Content="Home" />
                <Button Content="Settings" />
                <Button Content="About" />
            </utu:AutoLayout>
        </Grid>
    </utu:DrawerControl.DrawerContent>
</utu:DrawerControl>

<!-- Or enhance NavigationView with gesture support -->
<muxc:NavigationView PaneTitle="App Menu"
                     OpenPaneLength="320"
                     utu:DrawerControlBehavior.FitToDrawerContent="False"
                     Style="{StaticResource DrawerNavigationViewStyle}">
    <muxc:NavigationView.MenuItems>
        <muxc:NavigationViewItem Content="Home" />
        <muxc:NavigationViewItem Content="Settings" />
    </muxc:NavigationView.MenuItems>
    <muxc:NavigationView.Content>
        <Frame x:Name="ContentFrame" />
    </muxc:NavigationView.Content>
</muxc:NavigationView>
```
**Common Mistakes**:
- Placing DrawerControl inside a smaller container rather than as a full-window control. Due to lack of clipping, the drawer's opening edge should align with the screen edge.
- Forgetting to set `DrawerLength`, which results in the drawer having no defined size.
- Using `OpenDirection="Up"` or `"Down"` without ensuring the control spans the full screen height.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/DrawerControl.html

---

### ShadowContainer for Multi-Layer Elevation
**Rule**: Use `ShadowContainer` to apply one or more shadows to content. Set `Background` on the ShadowContainer itself (not the child) when using inner shadows.
**Why**: ShadowContainer supports multiple shadows (inner and outer), each with independent OffsetX/Y, BlurRadius, Spread, Color, and Opacity. This enables Material-style elevation and custom neumorphic designs. Shadow order matters because later shadows overlap earlier ones.
**Example (XAML)**:
```xml
<!-- Define reusable shadow collections in resources -->
<Page.Resources>
    <utu:ShadowCollection x:Key="CardElevation">
        <utu:Shadow BlurRadius="8"
                    OffsetY="4"
                    Opacity="0.12"
                    Color="{ThemeResource ShadowColor}" />
        <utu:Shadow BlurRadius="4"
                    OffsetY="2"
                    Opacity="0.08"
                    Color="{ThemeResource ShadowColor}" />
    </utu:ShadowCollection>
</Page.Resources>

<!-- Apply shadows to content -->
<utu:ShadowContainer Background="{ThemeResource SurfaceBrush}"
                     Shadows="{StaticResource CardElevation}">
    <Grid Padding="16"
          CornerRadius="12">
        <TextBlock Text="Elevated Card"
                   Style="{StaticResource TitleMedium}" />
    </Grid>
</utu:ShadowContainer>

<!-- Inner shadow (neumorphic style) -->
<utu:ShadowContainer Background="{ThemeResource SurfaceVariantBrush}">
    <utu:ShadowContainer.Shadows>
        <utu:ShadowCollection>
            <utu:Shadow BlurRadius="10"
                        OffsetX="5" OffsetY="5"
                        Opacity="0.3"
                        IsInner="True"
                        Color="Black" />
            <utu:Shadow BlurRadius="10"
                        OffsetX="-5" OffsetY="-5"
                        Opacity="0.7"
                        IsInner="True"
                        Color="White" />
        </utu:ShadowCollection>
    </utu:ShadowContainer.Shadows>
    <Grid Height="100" Width="100" CornerRadius="16" />
</utu:ShadowContainer>
```
**Common Mistakes**:
- Setting `Background` on the child element instead of the ShadowContainer when using inner shadows (`IsInner="True"`). Inner shadows paint on top of the background, so the background must be on the ShadowContainer.
- Confusing the `Shadow` property (inherited from UIElement) with the `Shadows` property (ShadowCollection). Always use the plural `Shadows` property.
- Not considering shadow order: later shadows in the collection overlap earlier ones.
**Uno Platform Notes**: Requires the `Uno.Toolkit.Skia.WinUI` package. ShadowContainer mimics the shape of its content by size and corner radius. Complex shapes such as text or images with alpha are not yet supported.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/ShadowContainer.html
