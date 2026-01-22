# Validation Notes - Uno Platform Compatibility

This document records Uno Platform-specific considerations, gaps, and workarounds for the WinUI 3 XAML skills.

## Overview

All skills in this document are validated against:
- **WinUI 3** (Windows App SDK 1.5+)
- **Uno Platform** (version 5.6+, Skia renderer)

Uno Platform aims for API compatibility with WinUI 3. Most skills work identically. This document notes exceptions.

---

## Platform-Specific Notes by Category

### Layout

| Skill | Uno Platform Status | Notes |
|-------|---------------------|-------|
| LAYOUT-001: Grid vs StackPanel | Full support | - |
| LAYOUT-002: Visual tree depth | Full support | - |
| LAYOUT-003: Fixed sizes | Full support | - |
| LAYOUT-004: Canvas | Full support | - |
| LAYOUT-005: RelativePanel | Full support | - |
| LAYOUT-006: Translation animations | Partial | Composition animations: Skia only. Use XAML animations on native iOS/Android. |
| LAYOUT-007: AdaptiveTrigger | Full support | - |
| LAYOUT-008: InvalidateArrange | Full support | - |

---

### Binding

| Skill | Uno Platform Status | Notes |
|-------|---------------------|-------|
| BINDING-001: x:Bind | Full support | Compiled bindings work identically. |
| BINDING-002: Binding modes | Full support | - |
| BINDING-003: Function binding | Full support | Functions must be public static or instance methods. |
| BINDING-004: Hot path binding | Full support | - |
| BINDING-005: Batch notifications | Full support | - |
| BINDING-006: UpdateSourceTrigger | Full support | - |
| BINDING-007: No StringFormat | Full support | StringFormat intentionally not supported (matches WinUI 3). |

---

### Async & Threading

| Skill | Uno Platform Status | Notes |
|-------|---------------------|-------|
| ASYNC-001: UI thread | Full support | - |
| ASYNC-002: Task.WhenAll | Full support | .NET Standard behavior. |
| ASYNC-003: Sequencing | Full support | - |
| ASYNC-004: DispatcherQueue | Full support | Use `DispatcherQueue.GetForCurrentThread()`. |
| ASYNC-005: Incremental loading | Full support | Use Uno Toolkit `ItemsRepeaterExtensions` for ItemsRepeater. |
| ASYNC-006: ConfigureAwait | Full support | - |
| ASYNC-007: Async void | Full support | - |

---

### Collections

| Skill | Uno Platform Status | Notes |
|-------|---------------------|-------|
| COLLECTIONS-001: Virtualization | Full support | ListView, GridView, ItemsRepeater all virtualize. |
| COLLECTIONS-002: ItemsRepeater | Full support | Requires ScrollViewer wrapper for virtualization. |
| COLLECTIONS-003: x:Name in templates | Full support | Same recycling behavior as WinUI 3. |
| COLLECTIONS-004: Phased loading | Partial | `x:Phase` supported; `ContainerContentChanging` varies by platform. |
| COLLECTIONS-005: DataTemplateSelector | Full support | - |
| COLLECTIONS-006: ItemsPanel | Full support | - |
| COLLECTIONS-007: ObservableCollection | Full support | Consider `ObservableRangeCollection` from CommunityToolkit. |

**Uno Toolkit Enhancement**: Use `ItemsRepeaterExtensions` for selection and incremental loading on ItemsRepeater:
```xml
<ItemsRepeater ItemsSource="{x:Bind Items}"
               utu:ItemsRepeaterExtensions.SelectedItem="{x:Bind SelectedItem, Mode=TwoWay}"/>
```

---

### Rendering & Composition

| Skill | Uno Platform Status | Notes |
|-------|---------------------|-------|
| RENDERING-001: Composition APIs | Partial | **Skia backends only**. Not available on native iOS/Android renderers. |
| RENDERING-002: Visibility vs Opacity | Full support | - |
| RENDERING-003: Acrylic | Partial | Supported on Skia backends. Fallback to solid color on unsupported platforms. |
| RENDERING-004: ThemeShadow | Full support | Use `Translation` Z-value > 0. Works on Skia. |
| RENDERING-005: BitmapCache | Partial | Skia backends only. |
| RENDERING-006: DecodePixelWidth | Full support | - |
| RENDERING-007: ConnectedAnimation | Partial | Support varies by platform. Test on each target. |

**Composition API Availability**:
```csharp
// Check if composition effects are supported
var capabilities = CompositionCapabilities.GetForCurrentView();
if (capabilities.AreEffectsSupported())
{
    // Use composition effects
}
else
{
    // Fallback to XAML
}
```

**Known Issues**:
- Some Composition effects don't render on software rendering (CPU)
- Native ripple effect on Android doesn't work with compositor thread

---

### Memory

| Skill | Uno Platform Status | Notes |
|-------|---------------------|-------|
| MEMORY-001: Event unsubscription | Full support | - |
| MEMORY-002: WeakReference | Full support | - |
| MEMORY-003: IDisposable | Full support | - |
| MEMORY-004: Clear references | Full support | - |
| MEMORY-005: Closure capturing | Full support | - |
| MEMORY-006: Object pooling | Full support | - |
| MEMORY-007: Memory diagnostics | Partial | `MemoryManager` not available on all platforms. Use platform-specific profiling. |

---

### Accessibility

| Skill | Uno Platform Status | Notes |
|-------|---------------------|-------|
| A11Y-001: AutomationProperties.Name | Full support | Maps to native accessibility labels. |
| A11Y-002: LabeledBy | Full support | - |
| A11Y-003: Color contrast | Full support | Same requirements on all platforms. |
| A11Y-004: Touch targets | Full support | - |
| A11Y-005: Keyboard navigation | Full support | Works on desktop platforms; less relevant on mobile. |
| A11Y-006: Image descriptions | Full support | - |
| A11Y-007: LiveSetting | Partial | Behavior varies by screen reader (Narrator, TalkBack, VoiceOver). |
| A11Y-008: High contrast | Partial | Windows-specific. Other platforms use system accessibility settings. |

**Uno-Specific**: `SimpleAccessibility` mode available for iOS/Android where nested accessible elements cause issues:
```csharp
FeatureConfiguration.AutomationPeer.UseSimpleAccessibility = true;
```

---

### Localization

| Skill | Uno Platform Status | Notes |
|-------|---------------------|-------|
| LOC-001: x:Uid | Full support | - |
| LOC-002: Multi-property x:Uid | Full support | - |
| LOC-003: ResourceLoader | Full support | - |
| LOC-004: Resource organization | Full support | - |
| LOC-005: RTL support | Full support | `FlowDirection` works on all platforms. |
| LOC-006: Runtime switching | Partial | Use `Uno.Extensions.Localization` for best cross-platform support. |
| LOC-007: Date/number formatting | Full support | Uses .NET globalization. |

---

### Styles & Theming

| Skill | Uno Platform Status | Notes |
|-------|---------------------|-------|
| STYLES-001: ThemeResource | Full support | - |
| STYLES-002: ResourceDictionary | Full support | - |
| STYLES-003: Lightweight styling | Full support | Works with Uno Themes (Material, Cupertino). |
| STYLES-004: Theme dictionaries | Full support | - |
| STYLES-005: BasedOn | Full support | - |
| STYLES-006: Implicit styles | Full support | - |
| STYLES-007: StaticResource vs ThemeResource | Full support | - |

---

### XAML Loading

| Skill | Uno Platform Status | Notes |
|-------|---------------------|-------|
| XAML-001: x:Load | Full support | **Behavior aligned to WinUI in Uno 5.6**. See migration note below. |
| XAML-002: x:DeferLoadStrategy | Deprecated | Use x:Load instead. |
| XAML-003: Tab deferral | Full support | - |
| XAML-004: Startup optimization | Full support | - |
| XAML-005: x:Load restrictions | Full support | Same restrictions as WinUI 3. |
| XAML-006: x:Phase | Full support | - |
| XAML-007: Merged dictionaries | Full support | - |

**Uno Platform 5.6 Breaking Change**:
> Lazy loading using `x:Load="False"` is no longer affected by changes to the visibility of the lazily-loaded element. Previously, binding the Visibility property of the lazily-loaded element and then updating the value would realize the element. This behavior now aligns with WinUI.

---

### Navigation

| Skill | Uno Platform Status | Notes |
|-------|---------------------|-------|
| NAV-001: Frame.Navigate | Full support | - |
| NAV-002: NavigationFailed | Full support | - |
| NAV-003: Page caching | Full support | - |
| NAV-004: Back navigation | Full support | Back button behavior varies by platform. |
| NAV-005: NavigationView | Full support | - |
| NAV-006: Resource cleanup | Full support | - |
| NAV-007: Deep linking | Platform-specific | Requires platform-specific URI scheme registration. |

**Uno.Extensions.Navigation**: Recommended for declarative navigation:
```xml
<Button Content="Go to Details"
        uen:Navigation.Request="Details"/>
```

---

### Input

| Skill | Uno Platform Status | Notes |
|-------|---------------------|-------|
| INPUT-001: Touch targets | Full support | - |
| INPUT-002: InputScope | Full support | Maps to native keyboard types. |
| INPUT-003: Pointer events | Full support | - |
| INPUT-004: Keyboard accelerators | Partial | Desktop only. Mobile platforms don't have keyboard shortcuts. |
| INPUT-005: Focus management | Full support | - |
| INPUT-006: Drag and drop | Partial | Desktop support; limited on mobile. |
| INPUT-007: Debouncing | Full support | - |

---

## Recommendations

### For Cross-Platform Development

1. **Test on all target platforms** - Behavior can vary, especially for:
   - Composition APIs
   - Connected animations
   - Keyboard shortcuts
   - High contrast mode

2. **Use Uno Toolkit** - Provides enhanced cross-platform support:
   - `ItemsRepeaterExtensions` for selection/incremental loading
   - `ResponsiveExtension` for responsive layouts
   - `VisualStateManagerExtensions` for state management

3. **Use Uno.Extensions** - For navigation and localization:
   - Region-based navigation
   - `IStringLocalizer` for runtime string resolution

4. **Prefer Skia renderer** - Best WinUI 3 compatibility:
   - Full Composition API support
   - Consistent rendering across platforms
   - ThemeShadow works correctly

### Feature Detection

```csharp
// Check platform capabilities before using advanced features
#if HAS_UNO
    // Uno Platform specific code
    #if __ANDROID__ || __IOS__
        // Use XAML animations instead of Composition
    #else
        // Skia - can use Composition APIs
    #endif
#else
    // WinUI 3 on Windows
#endif
```

---

## Resources

- [Uno Platform Documentation](https://platform.uno/docs/articles/)
- [Uno Platform Feature Support](https://platform.uno/docs/articles/supported-features.html)
- [Composition API in Uno](https://platform.uno/docs/articles/composition.html)
- [x:Bind in Uno](https://platform.uno/docs/articles/features/windows-ui-xaml-xbind.html)
- [Migrating from Previous Releases](https://platform.uno/docs/articles/migrating-from-previous-releases.html)
