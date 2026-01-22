# Layout Skills - WinUI 3 XAML

## LAYOUT-001: Prefer Grid Over Nested StackPanels

**Rule**: Use a single Grid with row/column definitions instead of nested StackPanels.

**Why**: Grid performs single-pass layout while nested StackPanels require multiple layout passes. Each nested panel adds measure/arrange overhead. Performance impact: 2-4x faster layout for complex forms.

**Example (Correct)**:
```xml
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="Auto"/>
        <ColumnDefinition Width="*"/>
    </Grid.ColumnDefinitions>

    <TextBlock Text="Name:" Grid.Row="0" Grid.Column="0" VerticalAlignment="Center"/>
    <TextBox Grid.Row="0" Grid.Column="1"/>

    <TextBlock Text="Email:" Grid.Row="1" Grid.Column="0" VerticalAlignment="Center"/>
    <TextBox Grid.Row="1" Grid.Column="1"/>

    <ListView Grid.Row="2" Grid.ColumnSpan="2"/>
</Grid>
```

**Common Mistakes**:
```xml
<!-- WRONG: Nested StackPanels cause multiple layout passes -->
<StackPanel>
    <StackPanel Orientation="Horizontal">
        <TextBlock Text="Name:"/>
        <TextBox/>
    </StackPanel>
    <StackPanel Orientation="Horizontal">
        <TextBlock Text="Email:"/>
        <TextBox/>
    </StackPanel>
</StackPanel>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/layout/layout-panels

---

## LAYOUT-002: Keep Visual Tree Shallow

**Rule**: Minimize visual tree depth to 4-6 levels. Remove unnecessary container elements.

**Why**: Each visual tree level adds layout pass time and hit-testing overhead. Deep trees cause exponential performance degradation. Measured impact: 2.5x faster layout/hit-testing with shallow trees.

**Example (Correct)**:
```xml
<!-- 2 levels deep -->
<Grid Padding="16">
    <TextBlock Text="{x:Bind Title}" Style="{StaticResource TitleTextBlockStyle}"/>
</Grid>
```

**Common Mistakes**:
```xml
<!-- WRONG: 6 unnecessary levels -->
<Border>
    <Grid>
        <StackPanel>
            <Border>
                <Grid>
                    <TextBlock Text="{x:Bind Title}"/>
                </Grid>
            </Border>
        </StackPanel>
    </Grid>
</Border>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/performance/optimize-xaml-layout

---

## LAYOUT-003: Use Fixed Sizes for Virtualized Items

**Rule**: Set explicit Height on items in virtualized lists (ListView, GridView, ItemsRepeater).

**Why**: Fixed sizes allow the virtualizing panel to skip the measure pass for off-screen items. Auto-sized items require measuring every item. Impact: 4x faster layout for 1000+ item lists.

**Example (Correct)**:
```xml
<ListView ItemsSource="{x:Bind Items}">
    <ListView.ItemTemplate>
        <DataTemplate x:DataType="local:Item">
            <Grid Height="72" Padding="12">
                <TextBlock Text="{x:Bind Name}"/>
            </Grid>
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

**Common Mistakes**:
```xml
<!-- WRONG: Auto height defeats virtualization optimization -->
<ListView.ItemTemplate>
    <DataTemplate x:DataType="local:Item">
        <StackPanel>  <!-- No fixed height -->
            <TextBlock Text="{x:Bind Name}" TextWrapping="Wrap"/>
            <TextBlock Text="{x:Bind Description}" TextWrapping="Wrap"/>
        </StackPanel>
    </DataTemplate>
</ListView.ItemTemplate>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/performance/optimize-gridview-and-listview

---

## LAYOUT-004: Use Canvas for Absolute Positioning

**Rule**: When elements have fixed absolute positions (games, diagrams, overlays), use Canvas instead of Grid with margins.

**Why**: Canvas skips measure/arrange entirely for children with explicit Canvas.Left/Top. Grid with margins still calculates relative positioning. Impact: 5x faster for 100+ positioned elements.

**Example (Correct)**:
```xml
<Canvas Width="800" Height="600">
    <Ellipse Canvas.Left="100" Canvas.Top="50" Width="20" Height="20" Fill="Red"/>
    <Ellipse Canvas.Left="200" Canvas.Top="150" Width="20" Height="20" Fill="Blue"/>
    <Line Canvas.Left="100" Canvas.Top="50" X2="100" Y2="100" Stroke="Black"/>
</Canvas>
```

**Common Mistakes**:
```xml
<!-- WRONG: Using Grid margins for absolute positioning -->
<Grid Width="800" Height="600">
    <Ellipse Margin="100,50,0,0" HorizontalAlignment="Left" VerticalAlignment="Top"
             Width="20" Height="20" Fill="Red"/>
</Grid>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/layout/layout-panels#canvas

---

## LAYOUT-005: Limit RelativePanel Constraint Complexity

**Rule**: Use RelativePanel for simple 2-5 element layouts. For complex layouts with many cross-dependencies, prefer Grid.

**Why**: RelativePanel resolves constraints iteratively. Many inter-element dependencies create O(n^2) resolution complexity. Grid's row/column model resolves in O(n).

**Example (Correct - Simple RelativePanel)**:
```xml
<RelativePanel>
    <Image x:Name="Avatar" Width="48" Height="48"/>
    <TextBlock x:Name="Title"
               RelativePanel.RightOf="Avatar"
               RelativePanel.AlignTopWith="Avatar"
               Margin="12,0,0,0"/>
    <TextBlock RelativePanel.RightOf="Avatar"
               RelativePanel.Below="Title"
               Margin="12,4,0,0"/>
</RelativePanel>
```

**Common Mistakes**:
```xml
<!-- WRONG: Complex cross-dependencies in RelativePanel -->
<RelativePanel>
    <Element1 x:Name="E1"/>
    <Element2 x:Name="E2" RelativePanel.RightOf="E1"/>
    <Element3 x:Name="E3" RelativePanel.Below="E2" RelativePanel.AlignLeftWith="E1"/>
    <Element4 RelativePanel.RightOf="E3" RelativePanel.AlignTopWith="E2"
              RelativePanel.LeftOf="E5"/>
    <Element5 x:Name="E5" RelativePanel.AlignRightWithPanel="True"/>
    <!-- Many more interdependent elements... -->
</RelativePanel>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/layout/layout-panels#relativepanel

---

## LAYOUT-006: Use Translation for Position Animations

**Rule**: Animate position using RenderTransform (TranslateTransform) or Composition Offset, never Margin.

**Why**: Margin changes trigger layout passes. Translation/Offset changes run on the composition thread without layout. Impact: 60 FPS vs 15-25 FPS for multi-element animations.

**Example (Correct)**:
```xml
<Button x:Name="SlideButton" Content="Slide Me">
    <Button.RenderTransform>
        <TranslateTransform x:Name="SlideTransform"/>
    </Button.RenderTransform>
</Button>
```

```csharp
// Composition API animation (preferred)
var visual = ElementCompositionPreview.GetElementVisual(SlideButton);
var animation = visual.Compositor.CreateVector3KeyFrameAnimation();
animation.InsertKeyFrame(1.0f, new Vector3(100, 0, 0));
animation.Duration = TimeSpan.FromMilliseconds(300);
visual.StartAnimation("Offset", animation);
```

**Common Mistakes**:
```xml
<!-- WRONG: Animating Margin triggers layout -->
<Storyboard>
    <ThicknessAnimation Storyboard.TargetProperty="Margin"
                        To="100,0,0,0" Duration="0:0:0.3"/>
</Storyboard>
```

**Uno Platform Notes**: Composition animations are supported on Skia backends. On native iOS/Android, use standard XAML animations which are hardware-accelerated.

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/motion/motion-in-practice

---

## LAYOUT-007: Use AdaptiveTrigger for Responsive Layouts

**Rule**: Use VisualStateManager with AdaptiveTrigger for responsive breakpoints instead of code-behind window size handlers.

**Why**: AdaptiveTrigger is declarative, works with XAML Hot Reload, and is optimized for window resize events. Code-behind handlers can miss resize events or cause flickering.

**Example (Correct)**:
```xml
<Grid>
    <VisualStateManager.VisualStateGroups>
        <VisualStateGroup>
            <VisualState x:Name="Narrow">
                <VisualState.StateTriggers>
                    <AdaptiveTrigger MinWindowWidth="0"/>
                </VisualState.StateTriggers>
                <VisualState.Setters>
                    <Setter Target="ContentPanel.Orientation" Value="Vertical"/>
                    <Setter Target="SidePanel.Visibility" Value="Collapsed"/>
                </VisualState.Setters>
            </VisualState>
            <VisualState x:Name="Wide">
                <VisualState.StateTriggers>
                    <AdaptiveTrigger MinWindowWidth="720"/>
                </VisualState.StateTriggers>
                <VisualState.Setters>
                    <Setter Target="ContentPanel.Orientation" Value="Horizontal"/>
                    <Setter Target="SidePanel.Visibility" Value="Visible"/>
                </VisualState.Setters>
            </VisualState>
        </VisualStateGroup>
    </VisualStateManager.VisualStateGroups>

    <StackPanel x:Name="ContentPanel">
        <Grid x:Name="SidePanel" Width="300"/>
        <Grid x:Name="MainContent"/>
    </StackPanel>
</Grid>
```

**Common Mistakes**:
```csharp
// WRONG: Manual size handling in code-behind
private void Page_SizeChanged(object sender, SizeChangedEventArgs e)
{
    if (e.NewSize.Width > 720)
    {
        ContentPanel.Orientation = Orientation.Horizontal;
        SidePanel.Visibility = Visibility.Visible;
    }
    else
    {
        ContentPanel.Orientation = Orientation.Vertical;
        SidePanel.Visibility = Visibility.Collapsed;
    }
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/layout/layouts-with-xaml#adaptive-layouts-with-visual-states-and-state-triggers

---

## LAYOUT-008: Avoid InvalidateArrange Cascades

**Rule**: Batch multiple property changes that affect layout. Avoid setting Width, Height, Margin in loops.

**Why**: Each layout-affecting property change can trigger InvalidateArrange. Multiple changes cause multiple layout passes. Batch changes together for a single pass.

**Example (Correct)**:
```csharp
// Batch changes - properties are set without intermediate layout
foreach (var item in items)
{
    var element = GetElement(item);
    // These queue layout invalidation but don't execute until next frame
    element.Width = newWidth;
    element.Height = newHeight;
    element.Margin = new Thickness(4);
}
// Single layout pass happens on next render frame
```

**Common Mistakes**:
```csharp
// WRONG: Forcing layout inside loop
foreach (var item in items)
{
    var element = GetElement(item);
    element.Width = newWidth;
    element.UpdateLayout(); // Forces immediate layout pass
    // Subsequent property changes trigger more passes
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/performance/optimize-xaml-layout
