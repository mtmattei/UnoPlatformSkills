# Uno Toolkit Data Controls Reference

## Prerequisites

```xml
xmlns:utu="using:Uno.Toolkit.UI"
```

```xml
<UnoFeatures>Toolkit;Material;</UnoFeatures>
```

---

### Card and CardContentControl Style Selection
**Rule**: Choose the correct card style based on visual emphasis level. Use `ElevatedCardStyle`/`ElevatedCardContentControlStyle` for high emphasis (z-axis shadow), `FilledCardStyle`/`FilledCardContentControlStyle` for medium emphasis (background color only), and `OutlinedCardStyle`/`OutlinedCardContentControlStyle` for low emphasis (border stroke only).
**Why**: Material Design defines three card variants with increasing levels of visual attention. Consistent use of elevation levels establishes visual hierarchy across the app. Using Card (with built-in properties) is faster for simple layouts; CardContentControl (with DataTemplate) gives full layout control.
**Example (XAML)**:
```xml
<!-- Card with built-in properties (simple usage) -->
<utu:Card HeaderContent="Meeting Notes"
          SubHeaderContent="Updated 2 hours ago"
          Style="{StaticResource ElevatedCardStyle}" />

<utu:Card HeaderContent="Quick Settings"
          SubHeaderContent="Tap to configure"
          Style="{StaticResource FilledCardStyle}" />

<utu:Card HeaderContent="Recent Activity"
          SubHeaderContent="3 new items"
          Style="{StaticResource OutlinedCardStyle}" />

<!-- CardContentControl with full custom layout -->
<utu:CardContentControl Style="{StaticResource ElevatedCardContentControlStyle}">
    <utu:CardContentControl.ContentTemplate>
        <DataTemplate>
            <utu:AutoLayout Orientation="Vertical"
                            Spacing="8"
                            Padding="16">
                <Image Source="ms-appx:///Assets/cover.png"
                       Height="180"
                       Stretch="UniformToFill" />
                <TextBlock Text="Custom Card Layout"
                           Style="{StaticResource HeadlineSmall}" />
                <TextBlock Text="Full control over content arrangement"
                           Style="{StaticResource BodyMedium}" />
                <Button Content="Action"
                        Style="{StaticResource FilledButtonStyle}"
                        HorizontalAlignment="Right" />
            </utu:AutoLayout>
        </DataTemplate>
    </utu:CardContentControl.ContentTemplate>
</utu:CardContentControl>
```
**Common Mistakes**:
- Using `CardContentControl` without setting a `Style`. You must explicitly apply one of the three styles (Elevated, Filled, Outlined).
- Using `Card` when you need complex custom layouts. Card provides HeaderContent/SubHeaderContent/MediaContent properties for standard layouts; switch to `CardContentControl` for full DataTemplate control.
- Applying a hardcoded `Background` color on a Card, overriding the style's theme-aware surface color.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/CardAndCardContentControl.html

---

### CardContentControl in Data-Bound Lists
**Rule**: Use `CardContentControl` as the root element of `ItemTemplate` in `ListView` or `ItemsRepeater` to create card-based list layouts. Bind the `Content` to the data item and define the layout in `ContentTemplate`.
**Why**: CardContentControl inherits from ContentControl, making it ideal as a data-template root. It handles elevation, border, and background per the chosen style, while DataTemplate defines the card's inner content.
**Example (XAML)**:
```xml
<ListView ItemsSource="{Binding Projects}">
    <ListView.ItemTemplate>
        <DataTemplate>
            <utu:CardContentControl Style="{StaticResource OutlinedCardContentControlStyle}"
                                    Margin="0,0,0,8">
                <utu:CardContentControl.ContentTemplate>
                    <DataTemplate>
                        <utu:AutoLayout Orientation="Horizontal"
                                        Spacing="16"
                                        Padding="16"
                                        CounterAxisAlignment="Center">
                            <FontIcon Glyph="&#xE8B7;"
                                      Foreground="{ThemeResource PrimaryBrush}" />
                            <utu:AutoLayout Orientation="Vertical" Spacing="4">
                                <TextBlock Text="{Binding Name}"
                                           Style="{StaticResource TitleMedium}" />
                                <TextBlock Text="{Binding Description}"
                                           Style="{StaticResource BodySmall}"
                                           Foreground="{ThemeResource OnSurfaceVariantBrush}" />
                            </utu:AutoLayout>
                        </utu:AutoLayout>
                    </DataTemplate>
                </utu:CardContentControl.ContentTemplate>
            </utu:CardContentControl>
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```
**Common Mistakes**:
- Nesting the data-bound content directly inside CardContentControl without using ContentTemplate. The inner DataTemplate is required for proper content projection.
- Using a `Button` as the item template root for clickable cards. Use `CardContentControl` with `CommandExtensions.Command` instead.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/CardAndCardContentControl.html

---

### Chip Styles and Standalone Usage
**Rule**: Apply the correct Chip style based on intent: `AssistChipStyle` for actions, `FilterChipStyle` for filtering, `InputChipStyle` for user input, and `SuggestionChipStyle` for dynamic suggestions. Use the `Icon` property for leading visuals.
**Why**: Material Design defines four chip types with distinct visual treatments and interaction patterns. Chip derives from ToggleButton, so it supports checked/unchecked states. The style determines the visual appearance and behavior (e.g., FilterChip shows a checkmark when selected).
**Example (XAML)**:
```xml
<!-- Assist chip (action, not toggleable) -->
<utu:Chip Content="Get directions"
          Style="{StaticResource AssistChipStyle}">
    <utu:Chip.Icon>
        <FontIcon Glyph="&#xE707;" />
    </utu:Chip.Icon>
</utu:Chip>

<!-- Filter chip (toggleable with checkmark) -->
<utu:Chip Content="Vegetarian"
          IsChecked="True"
          Style="{StaticResource FilterChipStyle}" />

<!-- Input chip (removable) -->
<utu:Chip Content="john@example.com"
          CanRemove="True"
          Style="{StaticResource InputChipStyle}">
    <utu:Chip.Icon>
        <Image Source="ms-appx:///Assets/Avatar.png" />
    </utu:Chip.Icon>
</utu:Chip>

<!-- Suggestion chip -->
<utu:Chip Content="Try Italian food"
          Style="{StaticResource SuggestionChipStyle}" />
```
**Common Mistakes**:
- Using `AssistChipStyle` when the chip should toggle (filter/select). Assist chips are for one-time actions, not selection.
- Setting `CanRemove="True"` without handling the `Removing` event. When used outside a ChipGroup, the Removed event does not automatically remove the Chip from the visual tree.
- Forgetting that `Chip` derives from `ToggleButton`. Binding to `IsChecked` (not a custom "IsSelected" property) controls the checked state.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/ChipAndChipGroup.html

---

### ChipGroup Selection Modes
**Rule**: Use `ChipGroup` to manage a collection of Chips with built-in selection. Set `SelectionMode` to `None`, `SingleOrNone`, `Single`, or `Multiple`. Bind `SelectedItem` or `SelectedItems` for data-driven selection.
**Why**: ChipGroup (derived from ItemsControl) manages chip creation from an ItemsSource and enforces selection constraints. Selection modes guarantee item counts: None=0, SingleOrNone=0-1, Single=always 1, Multiple=0-many. This eliminates manual toggle coordination.
**Example (XAML)**:
```xml
<!-- Single selection (one always selected) -->
<utu:ChipGroup ItemsSource="{Binding Categories}"
               SelectedItem="{Binding SelectedCategory, Mode=TwoWay}"
               SelectionMode="Single"
               Style="{StaticResource FilterChipGroupStyle}">
    <utu:ChipGroup.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Name}" />
        </DataTemplate>
    </utu:ChipGroup.ItemTemplate>
</utu:ChipGroup>

<!-- Multiple selection -->
<utu:ChipGroup ItemsSource="{Binding Tags}"
               SelectedItems="{Binding SelectedTags, Mode=TwoWay}"
               SelectionMode="Multiple"
               Style="{StaticResource FilterChipGroupStyle}">
    <utu:ChipGroup.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding}" />
        </DataTemplate>
    </utu:ChipGroup.ItemTemplate>
</utu:ChipGroup>

<!-- Inline chips (no binding) -->
<utu:ChipGroup SelectionMode="SingleOrNone"
               Style="{StaticResource SuggestionChipGroupStyle}">
    <utu:Chip Content="Morning" />
    <utu:Chip Content="Afternoon" />
    <utu:Chip Content="Evening" />
</utu:ChipGroup>
```
**Common Mistakes**:
- Binding `SelectedItems` to a type other than `object[]`. The property requires `object[]`, not `List<T>` or `ObservableCollection<T>`.
- Using `SelectionMode="Single"` when the user should be able to deselect all chips. Use `SingleOrNone` instead.
- Confusing `ItemClick` (fires on every tap) with selection state changes. Use `SelectedItem`/`SelectedItems` bindings for state tracking.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/ChipAndChipGroup.html

---

### LoadingView with ILoadable Source
**Rule**: Bind `LoadingView.Source` to an `ILoadable` (typically an `AsyncCommand`) to automatically show loading UI while asynchronous operations execute. Use `LoadingContent` to customize the loading overlay.
**Why**: LoadingView observes the `IsExecuting` property of the bound ILoadable source and automatically toggles between content and loading state. This eliminates manual IsLoading boolean management. For multiple concurrent operations, use `CompositeLoadableSource`.
**Example (XAML)**:
```xml
<!-- Single source -->
<utu:LoadingView Source="{Binding FetchDataCommand}">
    <!-- Normal content shown when not loading -->
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition Height="*" />
        </Grid.RowDefinitions>
        <Button Content="Refresh"
                Command="{Binding FetchDataCommand}" />
        <ListView Grid.Row="1"
                  ItemsSource="{Binding Items}" />
    </Grid>

    <!-- Custom loading overlay -->
    <utu:LoadingView.LoadingContent>
        <utu:AutoLayout Orientation="Vertical"
                        Spacing="16"
                        PrimaryAxisAlignment="Center"
                        CounterAxisAlignment="Center">
            <ProgressRing IsActive="True" />
            <TextBlock Text="Loading data..."
                       Style="{StaticResource BodyMedium}" />
        </utu:AutoLayout>
    </utu:LoadingView.LoadingContent>
</utu:LoadingView>

<!-- Multiple sources (loading while ANY source is executing) -->
<utu:LoadingView>
    <utu:LoadingView.Source>
        <utu:CompositeLoadableSource>
            <utu:LoadableSource Source="{Binding LoadProfileCommand}" />
            <utu:LoadableSource Source="{Binding LoadSettingsCommand}" />
        </utu:CompositeLoadableSource>
    </utu:LoadingView.Source>

    <utu:AutoLayout Orientation="Vertical" Spacing="16" Padding="16">
        <TextBlock Text="{Binding ProfileName}" Style="{StaticResource TitleLarge}" />
        <TextBlock Text="{Binding SettingsSummary}" Style="{StaticResource BodyMedium}" />
    </utu:AutoLayout>

    <utu:LoadingView.LoadingContent>
        <ProgressRing IsActive="True" />
    </utu:LoadingView.LoadingContent>
</utu:LoadingView>
```

**C# ViewModel**:
```csharp
public class MyViewModel
{
    public AsyncCommand FetchDataCommand { get; }

    public MyViewModel()
    {
        FetchDataCommand = new AsyncCommand(async () =>
        {
            // IsExecuting is automatically set to true/false
            Items = await _dataService.GetItemsAsync();
        });
    }
}
```
**Common Mistakes**:
- Binding Source to a regular `ICommand` that does not implement `ILoadable`. The command must implement `ILoadable` with `IsExecuting` and `IsExecutingChanged` for LoadingView to react.
- Not providing `LoadingContent`. Without it, LoadingView shows a default ProgressRing, which may not match your design.
- Binding Source to data instead of a command. Source expects an `ILoadable`, not the data itself.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/LoadingView.html

---

### Divider for Visual Separation
**Rule**: Use `Divider` to create thin line separators between content sections. Place inside AutoLayout or StackPanel for automatic orientation.
**Why**: Divider provides a Material Design-compliant separator line with proper thickness and color that respects theming. It is more semantic than a manual Border with 1px height and avoids hardcoded colors.
**Example (XAML)**:
```xml
<utu:AutoLayout Orientation="Vertical" Spacing="0" Padding="16">
    <TextBlock Text="Section 1"
               Style="{StaticResource TitleMedium}" />
    <TextBlock Text="Content for the first section."
               Style="{StaticResource BodyMedium}" />

    <utu:Divider />

    <TextBlock Text="Section 2"
               Style="{StaticResource TitleMedium}" />
    <TextBlock Text="Content for the second section."
               Style="{StaticResource BodyMedium}" />
</utu:AutoLayout>

<!-- Divider inside a list item template -->
<DataTemplate>
    <utu:AutoLayout Orientation="Vertical" Spacing="0">
        <utu:AutoLayout Orientation="Horizontal"
                        Spacing="12"
                        Padding="16"
                        CounterAxisAlignment="Center">
            <FontIcon Glyph="&#xE77B;" />
            <TextBlock Text="{Binding Name}"
                       Style="{StaticResource BodyLarge}" />
        </utu:AutoLayout>
        <utu:Divider />
    </utu:AutoLayout>
</DataTemplate>
```
**Common Mistakes**:
- Using a `Border` with `Height="1"` and a hardcoded color instead of `Divider`. Divider respects Material theming automatically.
- Setting `Spacing` on the parent AutoLayout when you want the Divider flush against adjacent content. Set `Spacing="0"` and use Padding on content sections instead.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls-styles.html
