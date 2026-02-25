# Uno Toolkit Navigation Controls Reference

## Prerequisites

```xml
xmlns:utu="using:Uno.Toolkit.UI"
```

```xml
<UnoFeatures>Toolkit;Material;</UnoFeatures>
```

---

### NavigationBar Structure
**Rule**: Use `NavigationBar` with string `Content` for the title, `MainCommand` for the leading button, and `PrimaryCommands` for trailing action buttons. Only use `AppBarButton` for commands.
**Why**: NavigationBar maps to native controls on iOS (UINavigationBar) and Android (Toolbar), providing platform-native transitions, pressed states, and overflow menus. The string Content path uses native font rendering. Custom FrameworkElement Content is rendered within the native bar's available area.
**Example (XAML)**:
```xml
<utu:NavigationBar Content="Page Title"
                   Background="{ThemeResource SurfaceBrush}"
                   Foreground="{ThemeResource OnSurfaceBrush}">
    <utu:NavigationBar.MainCommand>
        <AppBarButton x:Uid="NavigationBar.AppBarButton.Back"
                      Label="Back">
            <AppBarButton.Icon>
                <BitmapIcon UriSource="ms-appx:///Assets/Icons/arrow_back.png" />
            </AppBarButton.Icon>
        </AppBarButton>
    </utu:NavigationBar.MainCommand>
    <utu:NavigationBar.PrimaryCommands>
        <AppBarButton x:Uid="NavigationBar.AppBarButton.Search"
                      Label="Search">
            <AppBarButton.Icon>
                <FontIcon Glyph="&#xE721;" />
            </AppBarButton.Icon>
        </AppBarButton>
        <AppBarButton x:Uid="NavigationBar.AppBarButton.More"
                      Label="More">
            <AppBarButton.Icon>
                <FontIcon Glyph="&#xE712;" />
            </AppBarButton.Icon>
        </AppBarButton>
    </utu:NavigationBar.PrimaryCommands>
</utu:NavigationBar>
```
**Common Mistakes**:
- Using `AppBarToggleButton` or `AppBarSeparator` in PrimaryCommands. Only `AppBarButton` is supported.
- Setting `Height` on NavigationBar for iOS/Android. The height is fixed by the platform (44pt on iOS, 48-64dp on Android).
- Putting `AppBarButton` outside of a `NavigationBar` or `CommandBar`. Use regular `Button` with an icon for standalone actions.
**Uno Platform Notes**: SecondaryCommands are not supported on iOS. On iOS, Width should always be `NaN` and HorizontalAlignment should be `Stretch`. Native mode is the default on mobile; use `XamlDefaultNavigationBar` style for full template customization on desktop.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/NavigationBar.html

---

### NavigationBar MainCommand and Back Navigation
**Rule**: Set `MainCommandMode` to `Back` (default) for automatic back navigation, or `Action` when the MainCommand serves a different purpose such as a hamburger menu or confirmation dialog.
**Why**: When MainCommandMode is `Back`, the NavigationBar automatically displays a back arrow when the Frame's back stack is non-empty. When set to `Action`, it prevents the automatic back navigation and lets you bind a custom command. The default back icon can be customized on non-mobile platforms via the `NavigationBarBackIconData` resource.
**Example (XAML)**:
```xml
<!-- Default back navigation (automatic when Frame has back stack) -->
<utu:NavigationBar Content="Detail Page" />

<!-- Custom back icon on non-mobile platforms -->
<Application.Resources>
    <x:String x:Key="NavigationBarBackIconData">M20,11V13H8L13.5,18.5L12.08,19.92L4.16,12L12.08,4.08L13.5,5.5L8,11H20Z</x:String>
</Application.Resources>

<!-- Hamburger menu (Action mode) -->
<utu:NavigationBar Content="Home"
                   MainCommandMode="Action">
    <utu:NavigationBar.MainCommand>
        <AppBarButton Command="{Binding ToggleMenu}">
            <AppBarButton.Icon>
                <FontIcon Glyph="&#xE700;" />
            </AppBarButton.Icon>
        </AppBarButton>
    </utu:NavigationBar.MainCommand>
</utu:NavigationBar>
```
**Common Mistakes**:
- Leaving MainCommandMode as `Back` when using the MainCommand for a hamburger menu. This causes unintended back navigation.
- Providing a custom MainCommand but not setting any Icon or Label. On iOS, the previous page's title or "Back" text is shown by default; custom icons override this.
- On iOS, setting `MainCommand.Label` on the current page instead of the previous page. The label on page B's back button is determined by page A's NavigationBar.Content or MainCommand.Label.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/NavigationBar.html

---

### NavigationBar with Custom Content
**Rule**: When setting `Content` to a `FrameworkElement` (instead of a string), be aware of the fixed available height and platform-specific alignment behavior.
**Why**: Using a FrameworkElement as Content allows custom title layouts such as a search box or segmented control in the title area. However, the available height is constrained (30px on iOS, 48px on Android), and HorizontalContentAlignment behaves differently across platforms.
**Example (XAML)**:
```xml
<!-- Search box in navigation bar -->
<utu:NavigationBar>
    <utu:NavigationBar.Content>
        <TextBox PlaceholderText="Search..."
                 VerticalAlignment="Center" />
    </utu:NavigationBar.Content>
    <utu:NavigationBar.PrimaryCommands>
        <AppBarButton Label="Cancel">
            <AppBarButton.Icon>
                <FontIcon Glyph="&#xE711;" />
            </AppBarButton.Icon>
        </AppBarButton>
    </utu:NavigationBar.PrimaryCommands>
</utu:NavigationBar>
```
**Common Mistakes**:
- Setting HorizontalContentAlignment or VerticalContentAlignment. These are ignored on mobile when Content is a FrameworkElement. On iOS, Content is automatically centered unless you set `HorizontalAlignment="Stretch"`.
- Making Content taller than the native bar's available area. Content is clipped to the available height.
**Uno Platform Notes**: On Android, `HorizontalContentAlignment` supports only `Stretch` and `Left`. On Windows with XAML mode, set `IsDynamicOverflowEnabled="False"` for proper HorizontalContentAlignment behavior.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/NavigationBar.html

---

### TabBar for Lateral Navigation
**Rule**: Use `TabBar` with `TabBarItem` children for lateral (same-level) navigation. Apply `BottomTabBarStyle` for bottom navigation bars and `TopTabBarStyle` for top tab strips. Always pair with SafeArea on mobile.
**Why**: TabBar implements Material Design lateral navigation with built-in selection management. The BottomTabBarStyle already includes SafeArea bottom insets by default. TabBar supports single selection and raises `SelectionChanged` for navigation coordination.
**Example (XAML)**:
```xml
<Page xmlns:utu="using:Uno.Toolkit.UI">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="*" />
            <RowDefinition Height="Auto" />
        </Grid.RowDefinitions>

        <!-- Page content area -->
        <Frame x:Name="ContentFrame" />

        <!-- Bottom navigation bar -->
        <utu:TabBar Grid.Row="1"
                    Style="{StaticResource BottomTabBarStyle}">
            <utu:TabBar.Items>
                <utu:TabBarItem Content="Home"
                                x:Uid="Shell.TabBarItem.Home">
                    <utu:TabBarItem.Icon>
                        <FontIcon Glyph="&#xE80F;" />
                    </utu:TabBarItem.Icon>
                </utu:TabBarItem>
                <utu:TabBarItem Content="Search"
                                x:Uid="Shell.TabBarItem.Search">
                    <utu:TabBarItem.Icon>
                        <FontIcon Glyph="&#xE721;" />
                    </utu:TabBarItem.Icon>
                </utu:TabBarItem>
                <utu:TabBarItem Content="Profile"
                                x:Uid="Shell.TabBarItem.Profile">
                    <utu:TabBarItem.Icon>
                        <FontIcon Glyph="&#xE77B;" />
                    </utu:TabBarItem.Icon>
                </utu:TabBarItem>
            </utu:TabBar.Items>
        </utu:TabBar>

        <!-- Top tab bar variant -->
        <!--
        <utu:TabBar Grid.Row="0"
                    Style="{StaticResource TopTabBarStyle}">
            ...
        </utu:TabBar>
        -->
    </Grid>
</Page>
```
**Common Mistakes**:
- Forgetting to set the `Icon` property on TabBarItems when using `BottomTabBarStyle`. Bottom navigation requires icons per Material Design guidelines.
- Removing SafeArea insets from BottomTabBarStyle. The style includes bottom SafeArea by default; overriding it without re-adding SafeArea causes content to appear behind the device's navigation bar.
- Using TabBar for more than 5 items in bottom navigation, which violates Material Design guidelines.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/TabBarAndTabBarItem.html

---

### TabBar with Navigation Extensions
**Rule**: When using Uno Navigation Extensions, integrate TabBar with region-based navigation by setting `Region.Attached="True"` and `Region.Name` on TabBarItems rather than handling SelectionChanged in code-behind.
**Why**: XAML-based navigation via attached properties keeps navigation declarative and testable. Each TabBarItem maps to a named region/route, and the navigation framework handles page resolution. This avoids code-behind navigation logic entirely.
**Example (XAML)**:
```xml
<!-- Shell page with navigation-integrated TabBar -->
<Page xmlns:utu="using:Uno.Toolkit.UI"
      xmlns:uen="using:Uno.Extensions.Navigation.UI">
    <Grid uen:Region.Attached="True">
        <Grid.RowDefinitions>
            <RowDefinition Height="*" />
            <RowDefinition Height="Auto" />
        </Grid.RowDefinitions>

        <Frame uen:Region.Attached="True"
               uen:Region.Name="" />

        <utu:TabBar Grid.Row="1"
                    Style="{StaticResource BottomTabBarStyle}"
                    uen:Region.Attached="True">
            <utu:TabBar.Items>
                <utu:TabBarItem Content="Home"
                                uen:Region.Name="HomePage">
                    <utu:TabBarItem.Icon>
                        <FontIcon Glyph="&#xE80F;" />
                    </utu:TabBarItem.Icon>
                </utu:TabBarItem>
                <utu:TabBarItem Content="Settings"
                                uen:Region.Name="SettingsPage">
                    <utu:TabBarItem.Icon>
                        <FontIcon Glyph="&#xE713;" />
                    </utu:TabBarItem.Icon>
                </utu:TabBarItem>
            </utu:TabBar.Items>
        </utu:TabBar>
    </Grid>
</Page>
```
**Common Mistakes**:
- Handling `SelectionChanged` in code-behind to trigger navigation. With Uno Navigation Extensions, use `Region.Name` on TabBarItems and let the framework handle navigation declaratively.
- Forgetting `Region.Attached="True"` on the TabBar itself. The region must be attached to both the TabBar and the Frame for navigation to work.
- Not registering routes in the navigation configuration that match the `Region.Name` values.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/TabBarAndTabBarItem.html

---

### TabBarItem Styles and Floating Action Button
**Rule**: Use pre-built `TabBarItem` styles for Material and Cupertino themes. Use `BottomFabTabBarItemStyle` to embed a Floating Action Button (FAB) within the bottom TabBar.
**Why**: The toolkit provides several pre-built styles that implement Material Design and Cupertino guidelines. The BottomFabTabBarItemStyle creates a prominently elevated action button within the tab bar, following Material Design 3 FAB-in-bottom-nav patterns.
**Example (XAML)**:
```xml
<utu:TabBar Style="{StaticResource BottomTabBarStyle}">
    <utu:TabBar.Items>
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

        <!-- Floating Action Button in the tab bar -->
        <utu:TabBarItem Style="{StaticResource BottomFabTabBarItemStyle}"
                        Content="Add">
            <utu:TabBarItem.Icon>
                <FontIcon Glyph="&#xE710;" />
            </utu:TabBarItem.Icon>
        </utu:TabBarItem>

        <utu:TabBarItem Content="Notifications">
            <utu:TabBarItem.Icon>
                <FontIcon Glyph="&#xEA8F;" />
            </utu:TabBarItem.Icon>
        </utu:TabBarItem>
        <utu:TabBarItem Content="Profile">
            <utu:TabBarItem.Icon>
                <FontIcon Glyph="&#xE77B;" />
            </utu:TabBarItem.Icon>
        </utu:TabBarItem>
    </utu:TabBar.Items>
</utu:TabBar>
```
**Common Mistakes**:
- Using the FAB TabBarItem style without the `BottomTabBarStyle` on the parent TabBar.
- Expecting FAB-styled TabBarItems to behave as toggle selections. The FAB typically triggers an action rather than navigating to a tab.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/TabBarAndTabBarItem.html
