# Uno Toolkit Helpers and Extensions Reference

## Prerequisites

```xml
xmlns:utu="using:Uno.Toolkit.UI"
```

```xml
<UnoFeatures>Toolkit;Material;</UnoFeatures>
```

---

### ResponsiveExtension for Adaptive Property Values
**Rule**: Use `{utu:Responsive}` markup extension to set property values based on window width breakpoints. Define only the breakpoints you need; the extension resolves to the nearest matching breakpoint at or below the current width.
**Why**: ResponsiveExtension eliminates VisualStateManager triggers for width-based adaptations. It updates property values live as the window resizes. Default breakpoints are Narrowest(150), Narrow(300), Normal(600), Wide(800), Widest(1080). Only the breakpoints you specify are evaluated.
**Example (XAML)**:
```xml
<!-- Simple two-breakpoint responsive property -->
<Border Background="{utu:Responsive Narrow=Red, Wide=Blue}" />

<!-- Responsive text and layout -->
<TextBlock Text="{utu:Responsive Narrow=Mobile, Wide=Desktop}"
           Style="{utu:Responsive Narrow={StaticResource BodyMedium},
                                  Wide={StaticResource HeadlineMedium}}" />

<!-- Responsive Grid column count -->
<Grid>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="{utu:Responsive Narrow=*, Wide=300}" />
        <ColumnDefinition Width="{utu:Responsive Narrow=0, Wide=*}" />
    </Grid.ColumnDefinitions>

    <ListView ItemsSource="{Binding Items}" />
    <Frame Grid.Column="1"
           Visibility="{utu:Responsive Narrow=Collapsed, Wide=Visible}" />
</Grid>
```
**Common Mistakes**:
- Using ResponsiveExtension on non-FrameworkElement targets (e.g., ColumnDefinition, Run) inside a ResourceDictionary file. It works within Page/Control XAML but not in standalone resource dictionary files because it needs the XAML root object to initialize.
- Defining all five breakpoints when only two are needed. The extension resolves to the nearest defined breakpoint, so specifying Narrow and Wide is sufficient for most two-column layouts.
- Using this extension on UWP targeting Windows (non-Uno). The required `MarkupExtension.ProvideValue(IXamlServiceProvider)` overload is WinUI-only. Workaround: declare values as StaticResource references.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/responsive-extension.html

---

### Custom ResponsiveLayout Breakpoints
**Rule**: Define a `ResponsiveLayout` resource to override default breakpoints. Reference it via the `Layout` property on the markup extension or name it `DefaultResponsiveLayout` to apply globally.
**Why**: Different apps need different breakpoints. A tablet-first app might need wider thresholds. Custom ResponsiveLayout lets you define breakpoints per-page, per-section, or app-wide without modifying the global defaults.
**Example (XAML)**:
```xml
<!-- Per-page custom breakpoints -->
<Page.Resources>
    <utu:ResponsiveLayout x:Key="CustomLayout"
                          Narrow="400"
                          Wide="900" />
</Page.Resources>

<TextBlock Text="{utu:Responsive Layout={StaticResource CustomLayout},
                                Narrow=Small Screen,
                                Wide=Large Screen}" />

<!-- App-wide default override in App.xaml -->
<Application.Resources>
    <utu:ResponsiveLayout x:Key="DefaultResponsiveLayout"
                          Narrowest="0"
                          Narrow="360"
                          Normal="640"
                          Wide="1024"
                          Widest="1440" />
</Application.Resources>

<!-- This will use the app-wide DefaultResponsiveLayout automatically -->
<Border Background="{utu:Responsive Narrow=LightGray, Wide=White}" />
```
**Common Mistakes**:
- Naming the resource something other than `DefaultResponsiveLayout` and expecting it to be picked up automatically. The auto-lookup searches for exactly `x:Key="DefaultResponsiveLayout"` in the resource chain.
- Setting breakpoints that overlap or are not in ascending order.
**Uno Platform Notes**: Resolution precedence: (1) explicit `Layout` property on the extension, (2) `DefaultResponsiveLayout` resource walking up the tree, (3) Application.Resources, (4) hardcoded defaults [150/300/600/800/1080].
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/responsive-extension.html

---

### CommandExtensions for Non-Button Elements
**Rule**: Attach `utu:CommandExtensions.Command` to any UIElement to bind a command to its natural interaction gesture. The extension automatically selects the correct event: Tapped for generic UIElements, ItemClick for ListView, SelectionChanged for Selectors, Enter key for TextBox/PasswordBox, toggled for ToggleSwitch.
**Why**: Many controls (Grid, Border, Image, ToggleSwitch, ItemsRepeater) lack a built-in Command property. CommandExtensions provides a uniform command attachment that routes to the most appropriate event per control type, with automatic parameter passing (SelectedItem, Text, IsOn, etc.).
**Example (XAML)**:
```xml
<!-- Tappable card (UIElement.Tapped) -->
<Border Background="{ThemeResource SurfaceBrush}"
        CornerRadius="12"
        Padding="16"
        utu:CommandExtensions.Command="{Binding OpenDetailCommand}">
    <TextBlock Text="Tap me" Style="{StaticResource BodyLarge}" />
</Border>

<!-- ListView item click (requires IsItemClickEnabled) -->
<ListView ItemsSource="{Binding People}"
          IsItemClickEnabled="True"
          utu:CommandExtensions.Command="{Binding SelectPersonCommand}" />

<!-- PasswordBox on Enter (also dismisses keyboard) -->
<PasswordBox PlaceholderText="Password"
             utu:CommandExtensions.Command="{Binding LoginCommand}" />

<!-- ToggleSwitch (parameter is IsOn boolean) -->
<ToggleSwitch Header="Dark Mode"
              utu:CommandExtensions.Command="{Binding ToggleDarkModeCommand}" />

<!-- ItemsRepeater item tap (parameter is item DataContext) -->
<muxc:ItemsRepeater ItemsSource="{Binding Items}"
                    utu:CommandExtensions.Command="{Binding ItemTappedCommand}">
    <muxc:ItemsRepeater.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Name}"
                       Padding="12"
                       Style="{StaticResource BodyLarge}" />
        </DataTemplate>
    </muxc:ItemsRepeater.ItemTemplate>
</muxc:ItemsRepeater>

<!-- NavigationView item invoke -->
<NavigationView utu:CommandExtensions.Command="{Binding NavigateCommand}">
    <NavigationView.MenuItems>
        <NavigationViewItem Content="Home" />
        <NavigationViewItem Content="Settings" />
    </NavigationView.MenuItems>
</NavigationView>
```
**Common Mistakes**:
- Using CommandExtensions.Command on a `ListView` without setting `IsItemClickEnabled="True"`. The command relies on the ItemClick event, which is disabled by default.
- Setting `CommandParameter` on the control itself when you want the automatic parameter (SelectedItem, Text, etc.). CommandParameter overrides the automatic parameter, so only set it when you need a custom value.
- Using code-behind event handlers for interactions that CommandExtensions already supports. Prefer declarative command binding.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/command-extensions.html

---

### ItemsRepeaterExtensions for Selection
**Rule**: Attach `utu:ItemsRepeaterExtensions.SelectionMode` to an `ItemsRepeater` to enable selection. Use `SelectedItem`/`SelectedIndex` for single selection and `SelectedItems`/`SelectedIndexes` for multiple selection. The item template root must be a `SelectorItem` or `ToggleButton`-derived control.
**Why**: ItemsRepeater has no built-in selection support. This extension adds Single, SingleOrNone, Multiple, and None selection modes with automatic visual state management. The extension handles IsSelected/IsChecked toggling on supported template roots (ListViewItem, CheckBox, ToggleButton, Chip, RadioButton).
**Example (XAML)**:
```xml
<!-- Single selection with RadioButton template -->
<muxc:ItemsRepeater ItemsSource="{Binding Options}"
                    utu:ItemsRepeaterExtensions.SelectionMode="Single"
                    utu:ItemsRepeaterExtensions.SelectedItem="{Binding SelectedOption, Mode=TwoWay}"
                    utu:ItemsRepeaterExtensions.SelectedIndex="{Binding SelectedIndex, Mode=TwoWay}">
    <muxc:ItemsRepeater.ItemTemplate>
        <DataTemplate>
            <RadioButton Content="{Binding Name}" />
        </DataTemplate>
    </muxc:ItemsRepeater.ItemTemplate>
</muxc:ItemsRepeater>

<!-- Multiple selection with CheckBox template -->
<muxc:ItemsRepeater ItemsSource="{Binding Filters}"
                    utu:ItemsRepeaterExtensions.SelectionMode="Multiple"
                    utu:ItemsRepeaterExtensions.SelectedItems="{Binding SelectedFilters, Mode=TwoWay}">
    <muxc:ItemsRepeater.ItemTemplate>
        <DataTemplate>
            <CheckBox Content="{Binding Label}" />
        </DataTemplate>
    </muxc:ItemsRepeater.ItemTemplate>
</muxc:ItemsRepeater>
```

**C# ViewModel**:
```csharp
// SelectedItems must be object[] (not List<T>)
public object[] SelectedFilters { get; set; }

// SelectedItem can be the item type directly
public OptionModel SelectedOption { get; set; }
```
**Common Mistakes**:
- Binding `SelectedItems` to `List<T>` or `ObservableCollection<T>`. It must be `object[]`.
- Using `RadioButton` template root with `SelectionMode="Multiple"`. RadioButton does not support multiple selection due to control limitations.
- Adding manual `IsChecked` or `IsSelected` bindings to the template root. The extension manages these automatically.
- Forgetting that the template root must be one of: `ListViewItem`, `CheckBox`, `ToggleButton`, `Chip`, or `RadioButton`. Other controls will not show selection visuals.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/itemsrepeater-extensions.html

---

### StatusBar Extensions for Mobile
**Rule**: Use `utu:StatusBar.Foreground` and `utu:StatusBar.Background` attached properties on `Page` to control the mobile status bar appearance. Use `Auto` or `AutoInverse` for theme-aware foreground.
**Why**: The status bar foreground on mobile is either light (white) or dark (black). StatusBar extensions let you match the status bar to each page's design without platform-specific code. `Auto` follows the current theme (light foreground in dark mode), while `AutoInverse` reverses it.
**Example (XAML)**:
```xml
<!-- Dark status bar text on a light page -->
<Page xmlns:utu="using:Uno.Toolkit.UI"
      utu:StatusBar.Foreground="Dark"
      utu:StatusBar.Background="{ThemeResource SurfaceBrush}">
    <!-- Page content -->
</Page>

<!-- Light status bar text on a dark header page -->
<Page xmlns:utu="using:Uno.Toolkit.UI"
      utu:StatusBar.Foreground="Light"
      utu:StatusBar.Background="{ThemeResource PrimaryBrush}">
    <!-- Page content -->
</Page>

<!-- Auto-following theme (recommended default) -->
<Page xmlns:utu="using:Uno.Toolkit.UI"
      utu:StatusBar.Foreground="Auto"
      utu:StatusBar.Background="{ThemeResource SurfaceBrush}">
    <!-- Page content -->
</Page>
```
**Common Mistakes**:
- Applying StatusBar extensions on a control other than `Page`. The attached properties only work on Page elements.
- Setting a gradient or ImageBrush as Background. Only SolidColorBrush is supported.
- Using StatusBar extensions on desktop/web platforms. These properties have no effect on non-mobile platforms.
**Uno Platform Notes**: On iOS, you must set `UIViewControllerBasedStatusBarAppearance` to `false` in `info.plist` for these extensions to work. The `Auto` and `AutoInverse` values automatically update when the system or app theme changes.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/StatusBar-extensions.html

---

### InputExtensions for TextBox and PasswordBox
**Rule**: Use `utu:InputExtensions.AutoDismiss` to dismiss the keyboard on Enter, `utu:InputExtensions.AutoFocusNext` to auto-advance focus to the next field, and `utu:InputExtensions.ReturnType` to customize the soft keyboard's return button.
**Why**: These attached properties eliminate code-behind for common input form behaviors. AutoFocusNext improves form completion speed by moving focus on Enter. AutoDismiss prevents the keyboard from staying open after the last field. ReturnType signals intent to the user (Search, Send, Next, Done).
**Example (XAML)**:
```xml
<utu:AutoLayout Orientation="Vertical" Spacing="12" Padding="16">
    <!-- First field: auto-advance to next on Enter -->
    <TextBox x:Name="FirstNameInput"
             x:Uid="Form.TextBox.FirstName"
             PlaceholderText="First Name"
             utu:InputExtensions.AutoFocusNext="True"
             utu:InputExtensions.ReturnType="Next" />

    <!-- Second field: advance to a specific element -->
    <TextBox x:Name="LastNameInput"
             x:Uid="Form.TextBox.LastName"
             PlaceholderText="Last Name"
             utu:InputExtensions.AutoFocusNextElement="{Binding ElementName=EmailInput}"
             utu:InputExtensions.ReturnType="Next" />

    <!-- Third field: skip to password (non-sequential) -->
    <TextBox x:Name="EmailInput"
             x:Uid="Form.TextBox.Email"
             PlaceholderText="Email"
             utu:InputExtensions.AutoFocusNextElement="{Binding ElementName=PasswordInput}"
             utu:InputExtensions.ReturnType="Next" />

    <!-- Last field: dismiss keyboard on Enter -->
    <PasswordBox x:Name="PasswordInput"
                 x:Uid="Form.PasswordBox.Password"
                 PlaceholderText="Password"
                 utu:InputExtensions.AutoDismiss="True"
                 utu:InputExtensions.ReturnType="Done"
                 utu:CommandExtensions.Command="{Binding SubmitCommand}" />
</utu:AutoLayout>
```
**Common Mistakes**:
- Setting both `AutoFocusNext` and `AutoFocusNextElement` on the same control. `AutoFocusNextElement` takes precedence, so `AutoFocusNext` is ignored when both are set.
- Forgetting that `AutoFocusNext` uses `FocusManager.FindNextFocusableElement`, which follows the visual tree order. If the next focusable element is not the next input field in your form, use `AutoFocusNextElement` to specify explicitly.
- Not combining `AutoDismiss` with `CommandExtensions.Command` on the final field. Both can work together: the keyboard dismisses and the command executes on Enter.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/Input-extensions.html

---

### AncestorBinding and ItemsControlBinding for DataTemplate Access
**Rule**: Use `{utu:AncestorBinding AncestorType=TypeName, Path=...}` to bind to an ancestor's property from inside a DataTemplate. Use `{utu:ItemsControlBinding Path=...}` as a shortcut for binding to the nearest parent ItemsControl.
**Why**: Inside a DataTemplate, the DataContext is the item, not the page or parent ViewModel. In WinUI, there is no built-in `RelativeSource FindAncestor` support. These markup extensions solve this by walking up the visual tree to find an ancestor of the specified type and binding to its property.
**Example (XAML)**:
```xml
<!-- AncestorBinding: access Page-level properties from inside a template -->
<Page x:Name="MyPage"
      Tag="Page-level data">
    <ListView ItemsSource="{Binding Items}">
        <ListView.ItemTemplate>
            <DataTemplate>
                <utu:AutoLayout Orientation="Vertical" Spacing="4" Padding="12">
                    <!-- Bind to item property (normal) -->
                    <TextBlock Text="{Binding Name}"
                               Style="{StaticResource TitleSmall}" />

                    <!-- Bind to parent ViewModel property via AncestorBinding -->
                    <TextBlock Style="{StaticResource BodySmall}">
                        <Run Text="Category: " />
                        <Run Text="{utu:AncestorBinding AncestorType=Page,
                                                         Path=DataContext.CategoryName}" />
                    </TextBlock>

                    <!-- Bind to parent Page.Tag -->
                    <TextBlock Text="{utu:AncestorBinding AncestorType=Page, Path=Tag}"
                               Style="{StaticResource LabelSmall}" />
                </utu:AutoLayout>
            </DataTemplate>
        </ListView.ItemTemplate>
    </ListView>
</Page>

<!-- ItemsControlBinding shortcut: access the nearest ItemsControl -->
<ListView ItemsSource="{Binding Items}" Tag="ListView Tag">
    <ListView.ItemTemplate>
        <DataTemplate>
            <StackPanel>
                <TextBlock Text="{Binding Name}" />
                <!-- Access the ListView's DataContext (parent ViewModel) -->
                <TextBlock Text="{utu:ItemsControlBinding Path=DataContext.StatusMessage}" />
                <!-- Access the ListView's Tag -->
                <TextBlock Text="{utu:ItemsControlBinding Path=Tag}" />
            </StackPanel>
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```
**Common Mistakes**:
- Using `ElementName` binding instead of AncestorBinding inside a DataTemplate. ElementName bindings may not resolve correctly inside data templates across all platforms.
- Specifying `AncestorType` as the DataTemplate's host control type when you need the page's DataContext. Walk up to `Page` or the specific ancestor type that holds the property you need.
- Forgetting the `DataContext.` prefix in the Path when accessing ViewModel properties. `Path=DataContext.PropertyName` accesses the ancestor's DataContext; `Path=PropertyName` accesses a property of the ancestor element itself (like Tag, Width, etc.).
**Uno Platform Notes**: Both markup extensions are available on all non-Windows UWP platforms and all WinUI 3 platforms. They are not available on Windows UWP.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/ancestor-itemscontrol-binding.html
