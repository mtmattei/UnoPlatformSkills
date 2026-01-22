# Complete WinUI XAML Performance Best Practices

> Generated from provided skill snippets.  
> Note: Some XML/code blocks were incomplete in the source text and are preserved as-is.

---

## Sections

This file defines all sections, their ordering, impact levels, and descriptions. The section ID (in parentheses) is the filename prefix used to group rules.

1. **Eliminating UI Thread Blocking (async)**
   - Impact: **CRITICAL**
   - Description: UI thread blocking is the #1 performance killer in XAML apps. Each synchronous operation blocks user interaction and degrades responsiveness.

2. **XAML Loading & Compilation (xaml)**
   - Impact: **CRITICAL**
   - Description: XAML parsing and object creation directly impacts startup time, navigation performance, and memory usage.

3. **Data Binding Performance (binding)**
   - Impact: **HIGH**
   - Description: Binding updates trigger property notifications and UI updates. Efficient binding reduces CPU usage and improves responsiveness.

4. **Collection Virtualization (collections)**
   - Impact: **CRITICAL**
   - Description: Large collections without virtualization cause UI freezes, excessive memory usage, and poor scrolling performance.

5. **Layout Performance (layout)**
   - Impact: **HIGH**
   - Description: Layout passes are expensive. Minimizing layout recalculations and choosing efficient panels improves render performance.

6. **Rendering & Composition (rendering)**
   - Impact: **MEDIUM-HIGH**
   - Description: Efficient rendering through Composition APIs reduces GPU/CPU usage and enables smooth 60 FPS animations.

7. **Memory Management (memory)**
   - Impact: **HIGH**
   - Description: Memory leaks and excessive allocations degrade performance over time and can cause crashes.

8. **Navigation Performance (navigation)**
   - Impact: **MEDIUM-HIGH**
   - Description: Fast navigation between pages improves perceived performance and user satisfaction.

9. **Threading & Concurrency (threading)**
   - Impact: **HIGH**
   - Description: Proper threading prevents UI freezes by moving compute-intensive work off the UI thread.

10. **Controls & Templating (controls)**
    - Impact: **MEDIUM**
    - Description: Control complexity and template design affect rendering performance and memory usage.

11. **Graphics & Media (media)**
    - Impact: **MEDIUM-HIGH**
    - Description: Images, videos, and animations are resource-intensive. Optimization reduces memory and improves frame rates.

12. **Text & Typography (text)**
    - Impact: **LOW-MEDIUM**
    - Description: Text rendering optimizations accumulate across many text elements in an application.

13. **Input & Interaction (input)**
    - Impact: **MEDIUM**
    - Description: Responsive input handling and gesture recognition improve user experience and perceived performance.

14. **Resource Management (resources)**
    - Impact: **MEDIUM**
    - Description: Efficient resource dictionary usage reduces memory footprint and improves startup time.

15. **Application Lifecycle (lifecycle)**
    - Impact: **MEDIUM**
    - Description: Proper lifecycle management ensures smooth suspend/resume and efficient resource usage.

16. **Platform Integration (platform)**
    - Impact: **LOW-MEDIUM**
    - Description: Efficient use of Windows platform APIs minimizes overhead from interop boundaries.

---

# ASYNC — Eliminating UI Thread Blocking

## async-task-all.md

```markdown
---
title: Task.WhenAll() for Independent Operations
impact: CRITICAL
impactDescription: 2-10× improvement
tags: async, parallelization, tasks, waterfalls
---
```

### Task.WhenAll() for Independent Operations

**Impact: CRITICAL** (2-10× improvement)

When async operations have no interdependencies, execute them concurrently using Task.WhenAll() to eliminate sequential waterfalls.

**Incorrect** (sequential execution - 300ms + 200ms + 150ms = 650ms total):
```csharp
public async Task LoadPageDataAsync()
{
    var user = await _userService.GetUserAsync();
    var posts = await _postService.GetPostsAsync();
    var comments = await _commentService.GetCommentsAsync();

    return new PageData(user, posts, comments);
}
```

**Correct** (parallel execution - max(300ms, 200ms, 150ms) = 300ms total):
```csharp
public async Task LoadPageDataAsync()
{
    var userTask = _userService.GetUserAsync();
    var postsTask = _postService.GetPostsAsync();
    var commentsTask = _commentService.GetCommentsAsync();

    await Task.WhenAll(userTask, postsTask, commentsTask);

    return new PageData(userTask.Result, postsTask.Result, commentsTask.Result);
}
```

**Reference:** https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.whenall

---

## async-dispatcher-queue.md

```markdown
---
title: Use DispatcherQueue for UI Thread Updates
impact: CRITICAL
impactDescription: Prevents UI thread blocking
tags: async, dispatcher, threading, ui-updates
---
```

### Use DispatcherQueue for UI Thread Updates

**Impact: CRITICAL** (Prevents UI thread blocking)

Always use DispatcherQueue to marshal background work results back to the UI thread. Never block the UI thread waiting for background work.

**Incorrect** (blocks UI thread):
```csharp
public void LoadData()
{
    // This blocks the UI thread while waiting
    var data = Task.Run(async () => await _service.GetDataAsync()).Result;
    DataList.ItemsSource = data;
}
```

**Correct** (async with dispatcher):
```csharp
public async Task LoadDataAsync()
{
    var data = await Task.Run(async () => await _service.GetDataAsync());

    // Already on UI thread with async/await
    DataList.ItemsSource = data;
}

// Or if you're on a background thread:
private async void OnBackgroundWorkCompleted(object sender, DataEventArgs e)
{
    await DispatcherQueue.TryEnqueue(() =>
    {
        DataList.ItemsSource = e.Data;
    });
}
```

**Reference:** https://docs.microsoft.com/en-us/windows/windows-app-sdk/api/winrt/microsoft.ui.dispatching.dispatcherqueue

---

## async-incremental-loading.md

```markdown
---
title: Implement Incremental Loading for Large Collections
impact: CRITICAL
impactDescription: Eliminates initial load delays
tags: async, collections, incremental-loading, virtualization
---
```

### Implement Incremental Loading for Large Collections

**Impact: CRITICAL** (Eliminates initial load delays)

Use ISupportIncrementalLoading to load collection items on-demand as the user scrolls, rather than loading everything upfront.

**Incorrect** (loads all 10,000 items at once):
```csharp
public async Task LoadItemsAsync()
{
    var allItems = await _service.GetAllItemsAsync(); // Could be 10,000+ items
    Items = new ObservableCollection(allItems);
}
```

**Correct** (loads items incrementally):
```csharp
public class IncrementalItemsCollection : ObservableCollection, 
    ISupportIncrementalLoading
{
    private readonly IItemService _service;
    private int _currentPage = 0;
    private const int PageSize = 50;

    public bool HasMoreItems { get; private set; } = true;

    public IAsyncOperation LoadMoreItemsAsync(uint count)
    {
        return Task.Run(async () =>
        {
            var items = await _service.GetItemsAsync(_currentPage, PageSize);

            await DispatcherQueue.TryEnqueue(() =>
            {
                foreach (var item in items)
                {
                    Add(item);
                }
            });

            _currentPage++;
            HasMoreItems = items.Count == PageSize;

            return new LoadMoreItemsResult { Count = (uint)items.Count };
        }).AsAsyncOperation();
    }
}
```

**Reference:** https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.data.isupportincrementalloading

---

## async-image-loading.md

```markdown
---
title: Use Async Image Decoding
impact: HIGH
impactDescription: 50-100ms per image improvement
tags: async, images, decoding, performance
---
```

### Use Async Image Decoding

**Impact: HIGH** (50-100ms per image improvement)

Enable async image decoding to prevent blocking the UI thread during image load operations.

**Incorrect** (synchronous decoding blocks UI):
```xml

```

**Correct** (async decoding with proper sizing):
```xml





```

**Better** (with code-behind for more control):
```csharp
var bitmap = new BitmapImage();
bitmap.DecodePixelWidth = 400;
bitmap.DecodePixelType = DecodePixelType.Logical;

// Start async decode
await bitmap.SetSourceAsync(await file.OpenReadAsync());

MyImage.Source = bitmap;
```

**Reference:** https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.media.imaging.bitmapimage

---

## async-file-operations.md

```markdown
---
title: Never Block UI Thread with File I/O
impact: CRITICAL
impactDescription: Prevents multi-second freezes
tags: async, file-io, storage, performance
---
```

### Never Block UI Thread with File I/O

**Impact: CRITICAL** (Prevents multi-second freezes)

File operations can take hundreds of milliseconds to seconds. Always use async file APIs.

**Incorrect** (blocks UI thread):
```csharp
public void SaveFile(string content)
{
    // NEVER do this - blocks UI thread
    var file = StorageFile.GetFileFromPathAsync(path).GetAwaiter().GetResult();
    FileIO.WriteTextAsync(file, content).GetAwaiter().GetResult();
}
```

**Correct** (async file operations):
```csharp
public async Task SaveFileAsync(string content)
{
    var folder = ApplicationData.Current.LocalFolder;
    var file = await folder.CreateFileAsync("data.txt", 
        CreationCollisionOption.ReplaceExisting);
    await FileIO.WriteTextAsync(file, content);
}

public async Task LoadFileAsync()
{
    try
    {
        var folder = ApplicationData.Current.LocalFolder;
        var file = await folder.GetFileAsync("data.txt");
        return await FileIO.ReadTextAsync(file);
    }
    catch (FileNotFoundException)
    {
        return string.Empty;
    }
}
```

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/files/best-practices-for-writing-to-files

---

## async-dependent-sequencing.md

```markdown
---
title: Sequence Only Dependent Operations
impact: CRITICAL
impactDescription: Eliminates unnecessary waits
tags: async, dependencies, sequencing, waterfalls
---
```

### Sequence Only Dependent Operations

**Impact: CRITICAL** (Eliminates unnecessary waits)

Only use sequential await when operations are actually dependent. Parallelize everything else.

**Incorrect** (userId needed by both, but posts/comments are independent):
```csharp
public async Task LoadDashboardAsync()
{
    var user = await GetUserAsync();
    var posts = await GetUserPostsAsync(user.Id);
    var comments = await GetUserCommentsAsync(user.Id);
    var stats = await GetUserStatsAsync(user.Id);

    return new DashboardData(user, posts, comments, stats);
}
```

**Correct** (get userId first, then parallelize dependent operations):
```csharp
public async Task LoadDashboardAsync()
{
    // First get the user (required for subsequent calls)
    var user = await GetUserAsync();

    // Now parallelize all operations that depend only on user.Id
    var (posts, comments, stats) = await (
        GetUserPostsAsync(user.Id),
        GetUserCommentsAsync(user.Id),
        GetUserStatsAsync(user.Id)
    );

    return new DashboardData(user, posts, comments, stats);
}
```

**Reference:** https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/

---

# XAML — Loading & Compilation

## xaml-x-load.md

```markdown
---
title: Use x:Load for Conditional UI
impact: CRITICAL
impactDescription: 50-200ms startup improvement per deferred element
tags: xaml, x:Load, deferred-loading, startup, memory
---
```

### Use x:Load for Conditional UI

**Impact: CRITICAL** (50-200ms startup improvement per deferred element)

Use x:Load instead of Visibility.Collapsed for UI elements that may not be shown initially. x:Load prevents element creation entirely, reducing memory and startup time.

**Incorrect** (element fully created but hidden - wastes memory and startup time):
```xml





```

**Correct** (element not created until ShowAdvancedOptions is true):
```xml





```

**Code-behind to toggle x:Load:**
```csharp
// To show the element
AdvancedOptionsGrid.Visibility = Visibility.Visible; // This loads it

// To hide and unload
AdvancedOptionsGrid.Visibility = Visibility.Collapsed; // Unloads after next layout pass
```

**Important Notes:**
- x:Load requires x:Name to control from code
- Setting to Visible triggers creation on next layout pass
- Setting to Collapsed unloads element and releases resources
- Cannot use with ElementName binding targets

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/xaml-platform/x-load-attribute

---

## xaml-x-bind.md

```markdown
---
title: Prefer x:Bind Over Binding
impact: HIGH
impactDescription: 20-50% binding performance improvement
tags: xaml, x:Bind, binding, performance, compile-time
---
```

### Prefer x:Bind Over Binding

**Impact: HIGH** (20-50% binding performance improvement)

Use x:Bind instead of Binding for compile-time validation, better performance, and less memory overhead.

**Incorrect** (runtime Binding - slow, no compile-time checking):
```xml



```

**Correct** (compile-time x:Bind - fast, type-safe):
```xml



```

**Performance Comparison:**
```
Scenario: 1000 TextBlocks binding to properties

Binding (runtime):
- Initial binding: ~45ms
- Memory: ~1.2MB
- Update time: ~8ms

x:Bind (compile-time):
- Initial binding: ~12ms (3.75× faster)
- Memory: ~0.4MB (3× less memory)
- Update time: ~3ms (2.6× faster)
```

**Key Differences:**
- x:Bind defaults to OneTime (Binding defaults to OneWay)
- x:Bind requires explicit Mode for updates
- x:Bind supports function binding: `{x:Bind local:Helpers.Format(Value)}`
- x:Bind catches errors at compile time

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/xaml-platform/x-bind-markup-extension

---

## xaml-x-deferloadstrategy.md

```markdown
---
title: Defer Loading of Complex Controls
impact: CRITICAL
impactDescription: Reduces initial page load time
tags: xaml, x:DeferLoadStrategy, lazy-loading, startup
---
```

### Defer Loading of Complex Controls

**Impact: CRITICAL** (Reduces initial page load time)

Use x:DeferLoadStrategy to delay creation of expensive controls until explicitly needed via FindName().

**Incorrect** (all tabs created immediately):
```xml











```

**Correct** (tabs created on-demand):
```xml

















```

**Code-behind:**
```csharp
private void Pivot_SelectionChanged(object sender, SelectionChangedEventArgs e)
{
    var pivot = sender as Pivot;
    var selectedItem = pivot.SelectedItem as PivotItem;

    if (selectedItem?.Header?.ToString() == "Charts")
    {
        // This realizes the deferred element
        FindName("ChartsContainer");
    }
    else if (selectedItem?.Header?.ToString() == "Settings")
    {
        FindName("SettingsContainer");
    }
}
```

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/xaml-platform/x-deferloadstrategy-attribute

---

## xaml-minimal-visual-tree.md

```markdown
---
title: Keep Visual Tree Shallow
impact: HIGH
impactDescription: Faster layout and rendering
tags: xaml, visual-tree, layout, performance, hierarchy
---
```

### Keep Visual Tree Shallow

**Impact: HIGH** (Faster layout and rendering)

Minimize visual tree depth. Each level adds layout passes and hit-testing overhead.

**Incorrect** (unnecessary nesting - 7 levels deep):
```xml













```

**Correct** (minimal nesting - 2 levels deep):
```xml



```

**Better Example** - Avoid unnecessary containers:

**Incorrect:**
```xml






```

**Correct:**
```xml








```

**Measurement:**
```
Deep tree (10 levels, 100 elements):
- Layout pass: ~18ms
- Hit testing: ~2ms

Shallow tree (4 levels, 100 elements):
- Layout pass: ~7ms (2.5× faster)
- Hit testing: ~0.8ms (2.5× faster)
```

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/debug-test-perf/optimize-your-xaml-layout

---

## xaml-avoid-nested-panels.md

```markdown
---
title: Avoid Deeply Nested Panels
impact: HIGH
impactDescription: Reduces layout passes
tags: xaml, layout, panels, nested, performance
---
```

### Avoid Deeply Nested Panels

**Impact: HIGH** (Reduces layout passes)

Each nested panel adds a layout pass. Use single panels with advanced features instead of nesting.

**Incorrect** (nested panels - multiple layout passes):
```xml










```

**Correct** (single Grid with proper layout):
```xml

















```

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/debug-test-perf/optimize-your-xaml-layout

---

## xaml-usercontrol-reuse.md

```markdown
---
title: Cache and Reuse UserControl Instances
impact: MEDIUM-HIGH
impactDescription: Reduces object allocation
tags: xaml, usercontrol, caching, memory, reuse
---
```

### Cache and Reuse UserControl Instances

**Impact: MEDIUM-HIGH** (Reduces object allocation)

For frequently shown/hidden controls, cache instances instead of recreating them.

**Incorrect** (creates new control every time):
```csharp
private void ShowDetailsPanel()
{
    var detailsPanel = new DetailsPanelControl
    {
        DataContext = _selectedItem
    };
    ContentArea.Content = detailsPanel;
}
```

**Correct** (reuses cached instance):
```csharp
private DetailsPanelControl _cachedDetailsPanel;

private void ShowDetailsPanel()
{
    if (_cachedDetailsPanel == null)
    {
        _cachedDetailsPanel = new DetailsPanelControl();
    }

    _cachedDetailsPanel.DataContext = _selectedItem;
    ContentArea.Content = _cachedDetailsPanel;
}

private void HideDetailsPanel()
{
    // Keep the instance cached, just remove from visual tree
    ContentArea.Content = null;
}
```

**For navigation scenarios:**
```csharp
private readonly Dictionary _pageCache = new();

protected override void OnNavigatedTo(NavigationEventArgs e)
{
    var pageType = e.SourcePageType;

    if (!_pageCache.TryGetValue(pageType, out var page))
    {
        page = (Page)Activator.CreateInstance(pageType);
        _pageCache[pageType] = page;
    }

    Frame.Content = page;
}
```

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/debug-test-perf/optimize-xaml-loading

---

# BINDING — Data Binding Performance

## binding-x-bind-mode.md

```markdown
---
title: Use Appropriate Binding Modes
impact: HIGH
impactDescription: 20-40% binding overhead reduction
tags: binding, x:Bind, mode, onetime, oneway, twoway
---
```

### Use Appropriate Binding Modes

**Impact: HIGH** (20-40% binding overhead reduction)

Choose the minimal binding mode for your scenario. Unnecessary change tracking wastes CPU and memory.

**Incorrect** (uses default OneWay for static data):
```xml




```

**Correct** (uses OneTime for data that never changes):
```xml



```

**Binding Mode Decision Tree:**
```
Does the data NEVER change?
└─ YES → Mode=OneTime

Does the user edit the value?
└─ YES → Mode=TwoWay
└─ NO → Does the source property change?
        └─ YES → Mode=OneWay
        └─ NO → Mode=OneTime
```

**Complete Example:**
```xml











```

**Performance Impact:**
```
1000 bindings over 10 seconds with 100 property changes:

OneTime:  0 CPU cycles for updates
OneWay:   ~150ms CPU time
TwoWay:   ~320ms CPU time (includes reverse binding checks)
```

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/data-binding/data-binding-in-depth

---

## binding-x-bind-function.md

```markdown
---
title: Use x:Bind Function Binding for Computed Values
impact: MEDIUM-HIGH
impactDescription: Eliminates converter overhead
tags: binding, x:Bind, functions, converters, computed
---
```

### Use x:Bind Function Binding for Computed Values

**Impact: MEDIUM-HIGH** (Eliminates converter overhead)

Use x:Bind's function binding feature instead of IValueConverter for better performance and type safety.

**Incorrect** (requires IValueConverter instance and boxing):
```xml

```
```csharp
public class PriceConverter : IValueConverter
{
    public object Convert(object value, Type targetType, 
                         object parameter, string language)
    {
        var price = (double)value; // Boxing/unboxing
        var currency = parameter as string;
        return $"{currency} {price:F2}";
    }

    public object ConvertBack(object value, Type targetType, 
                             object parameter, string language)
    {
        throw new NotImplementedException();
    }
}
```

**Correct** (direct function call - no converter needed):
```xml

```
```csharp
public static class Helpers
{
    public static string FormatPrice(double price, string currency)
    {
        return $"{currency} {price:F2}";
    }
}
```

**Multiple Parameters Example:**
```xml





```
```csharp
public static class Helpers
{
    public static string GetFullName(string first, string last)
    {
        return $"{first} {last}";
    }

    public static Visibility ToVisibility(bool isVisible)
    {
        return isVisible ? Visibility.Visible : Visibility.Collapsed;
    }
}
```

**Performance Comparison:**
```
1000 conversions:

IValueConverter: ~8ms (object allocation, boxing, dictionary lookup)
x:Bind function: ~1.5ms (direct call, no allocations)

5.3× faster with x:Bind functions
```

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/xaml-platform/function-bindings

---

## binding-update-source-trigger.md

```markdown
---
title: Control Binding Update Frequency
impact: MEDIUM
impactDescription: Reduces unnecessary updates
tags: binding, update-source-trigger, performance, throttling
---
```

### Control Binding Update Frequency

**Impact: MEDIUM** (Reduces unnecessary updates)

For TwoWay bindings, control when the source is updated to avoid excessive property notifications.

**Incorrect** (updates on every keystroke - can cause performance issues):
```xml

```
```csharp
// This fires on EVERY keystroke, potentially triggering expensive operations
private string _searchQuery;
public string SearchQuery
{
    get => _searchQuery;
    set
    {
        _searchQuery = value;
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(SearchQuery)));
        PerformExpensiveSearch(); // Called on every keystroke!
    }
}
```

**Correct** (update only when user finishes - use UpdateSourceTrigger):
```xml

```
```csharp
private DispatcherTimer _searchDebounceTimer;

private void SearchBox_TextChanged(object sender, TextChangedEventArgs e)
{
    _searchDebounceTimer?.Stop();
    _searchDebounceTimer = new DispatcherTimer { Interval = TimeSpan.FromMilliseconds(300) };
    _searchDebounceTimer.Tick += (s, args) =>
    {
        _searchDebounceTimer.Stop();
        ViewModel.SearchQuery = SearchBox.Text;
    };
    _searchDebounceTimer.Start();
}
```

**Or use a dedicated debouncing approach:**
```csharp
public class DebouncedProperty
{
    private DispatcherTimer _timer;
    private T _value;
    public event Action ValueChanged;

    public T Value
    {
        get => _value;
        set
        {
            _value = value;
            _timer?.Stop();
            _timer = new DispatcherTimer { Interval = TimeSpan.FromMilliseconds(300) };
            _timer.Tick += (s, e) =>
            {
                _timer.Stop();
                ValueChanged?.Invoke(_value);
            };
            _timer.Start();
        }
    }
}
```

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/data-binding/data-binding-in-depth

---

## binding-batch-updates.md

```markdown
---
title: Batch Multiple Property Updates
impact: MEDIUM
impactDescription: Reduces notification overhead
tags: binding, batch-updates, inotifypropertychanged, performance
---
```

### Batch Multiple Property Updates

**Impact: MEDIUM** (Reduces notification overhead)

When updating multiple related properties, batch the updates to reduce binding evaluations.

**Incorrect** (triggers 3 separate binding updates):
```csharp
public void UpdateUser(User user)
{
    FirstName = user.FirstName; // Triggers bindings
    LastName = user.LastName;   // Triggers bindings
    Email = user.Email;         // Triggers bindings
}

private string _firstName;
public string FirstName
{
    get => _firstName;
    set
    {
        _firstName = value;
        OnPropertyChanged();
    }
}
```

**Correct** (batch updates, single notification):
```csharp
public void UpdateUser(User user)
{
    _firstName = user.FirstName;  // No notification yet
    _lastName = user.LastName;    // No notification yet
    _email = user.Email;          // No notification yet

    // Single batched notification
    OnPropertyChanged(string.Empty); // Empty string means "all properties changed"
}

// Or notify specific properties
public void UpdateUser(User user)
{
    _firstName = user.FirstName;
    _lastName = user.LastName;
    _email = user.Email;

    OnPropertiesChanged(nameof(FirstName), nameof(LastName), nameof(Email));
}

protected void OnPropertiesChanged(params string[] propertyNames)
{
    foreach (var name in propertyNames)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
    }
}
```

**Better with modern C# (.NET 5+):**
```csharp
using CommunityToolkit.Mvvm.ComponentModel;

public partial class UserViewModel : ObservableObject
{
    [ObservableProperty]
    [NotifyPropertyChangedFor(nameof(FullName))]
    private string _firstName;

    [ObservableProperty]
    [NotifyPropertyChangedFor(nameof(FullName))]
    private string _lastName;

    public string FullName => $"{FirstName} {LastName}";

    public void UpdateUser(User user)
    {
        // Source generator creates efficient batched updates
        FirstName = user.FirstName;
        LastName = user.LastName;
    }
}
```

**Reference:** https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.inotifypropertychanged

---

## binding-converter-performance.md

```markdown
---
title: Optimize IValueConverter Implementations
impact: MEDIUM
impactDescription: Reduces converter overhead
tags: binding, converter, performance, optimization
---
```

### Optimize IValueConverter Implementations

**Impact: MEDIUM** (Reduces converter overhead)

IValueConverters can be called thousands of times. Optimize their implementations.

**Incorrect** (allocates on every call):
```csharp
public class BoolToVisibilityConverter : IValueConverter
{
    public object Convert(object value, Type targetType, 
                         object parameter, string language)
    {
        var isVisible = (bool)value;

        // Creates new Visibility object on every call (boxing)
        return isVisible ? Visibility.Visible : Visibility.Collapsed;
    }

    public object ConvertBack(object value, Type targetType, 
                             object parameter, string language)
    {
        // Often not implemented but still allocates exception
        throw new NotImplementedException();
    }
}
```

**Correct** (cached values, no allocations):
```csharp
public class BoolToVisibilityConverter : IValueConverter
{
    // Cache boxed values to avoid repeated boxing
    private static readonly object VisibleBox = Visibility.Visible;
    private static readonly object CollapsedBox = Visibility.Collapsed;

    public object Convert(object value, Type targetType, 
                         object parameter, string language)
    {
        if (value is bool boolValue)
        {
            return boolValue ? VisibleBox : CollapsedBox;
        }
        return CollapsedBox;
    }

    public object ConvertBack(object value, Type targetType, 
                             object parameter, string language)
    {
        // Return sensible default instead of throwing
        return value is Visibility vis && vis == Visibility.Visible;
    }
}
```

**Best** (avoid converters entirely with x:Bind functions):
```xml


```
```csharp
public static class Helpers
{
    public static Visibility ToVisibility(bool isVisible)
    {
        return isVisible ? Visibility.Visible : Visibility.Collapsed;
    }
}
```

**Reference:** https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.data.ivalueconverter

---

# COLLECTIONS — Virtualization

## collections-itemsrepeater.md

```markdown
---
title: Use ItemsRepeater for Complex Lists
impact: HIGH
impactDescription: Better virtualization and layout control
tags: collections, itemsrepeater, virtualization, layout
---
```

### Use ItemsRepeater for Complex Lists

**Impact: HIGH** (Better virtualization and layout control)

ItemsRepeater provides more efficient virtualization and flexible layouts compared to ListView/GridView.

**Incorrect** (ListView with limited layout options):
```xml









```

**Correct** (ItemsRepeater with efficient virtualization):
```xml














```

**Advanced with custom layout:**
```xml
<ScrollViewer>
    <ItemsRepeater ItemsSource="{x:Bind Items}"
                   ItemTemplate="{...}" />
</ScrollViewer>
```

---

## collections-virtualization-mode.md

```markdown
---
title: Choose Correct Virtualization Mode
impact: HIGH
impactDescription: Balances memory vs scrolling performance
tags: collections, virtualization, itemswrapgrid, performance
---
```

### Choose Correct Virtualization Mode

**Impact: HIGH** (Balances memory vs scrolling performance)

Choose the right VirtualizingStackPanel mode based on your item characteristics and scrolling patterns.

**Standard Mode** (default - recycles containers aggressively):
```xml








```

**Recycling Mode** (keeps more containers in memory for smoother scrolling):
```xml







```

**When to use each mode:**

**Standard Mode:**
- Items have varying heights/complex templates
- Memory is constrained
- List size is moderate (< 1000 items)
- Slower scrolling is acceptable

**Recycling Mode:**
- Items have uniform size
- Fast scrolling is critical
- More memory available
- Large lists (1000+ items)

**Performance Comparison:**
```
Scenario: 10,000 items, uniform height, fast scrolling

Standard Mode:
- Memory: ~15MB
- Scroll FPS: ~40 FPS
- Container creation: Frequent

Recycling Mode:
- Memory: ~22MB
- Scroll FPS: ~58 FPS
- Container creation: Minimal
```

**Reference:** https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.controls.virtualizationmode

---

## collections-incremental-loading.md

```markdown
---
title: Implement ISupportIncrementalLoading
impact: CRITICAL
impactDescription: Eliminates large dataset load times
tags: collections, incremental-loading, pagination, performance
---
```

### Implement ISupportIncrementalLoading

**Impact: CRITICAL** (Eliminates large dataset load times)

Load collection data incrementally as user scrolls to avoid blocking on large datasets.

**Incorrect** (loads all items upfront - could take seconds):
```csharp
public async Task LoadItemsAsync()
{
    var allItems = await _api.GetAllItemsAsync(); // 10,000+ items
    Items = new ObservableCollection(allItems);
}
```

**Correct** (loads in batches of 50):
```csharp
public class IncrementalLoadingCollection : ObservableCollection, 
    ISupportIncrementalLoading
{
    private readonly Func>> _loadMoreItems;
    private int _currentPage = 0;
    private const int PageSize = 50;
    private bool _isLoading;

    public IncrementalLoadingCollection(
        Func>> loadMoreItems)
    {
        _loadMoreItems = loadMoreItems;
    }

    public bool HasMoreItems { get; private set; } = true;

    public IAsyncOperation LoadMoreItemsAsync(uint count)
    {
        if (_isLoading)
        {
            return Task.FromResult(new LoadMoreItemsResult { Count = 0 })
                      .AsAsyncOperation();
        }

        return LoadMoreItemsInternalAsync(count).AsAsyncOperation();
    }

    private async Task LoadMoreItemsInternalAsync(uint count)
    {
        _isLoading = true;

        try
        {
            var items = await _loadMoreItems(_currentPage, PageSize);
            var itemsList = items.ToList();

            foreach (var item in itemsList)
            {
                Add(item);
            }

            _currentPage++;
            HasMoreItems = itemsList.Count == PageSize;

            return new LoadMoreItemsResult { Count = (uint)itemsList.Count };
        }
        finally
        {
            _isLoading = false;
        }
    }
}
```

**Usage:**
```csharp
public class MainViewModel
{
    public IncrementalLoadingCollection Products { get; }

    public MainViewModel(IProductService productService)
    {
        Products = new IncrementalLoadingCollection(
            async (page, pageSize) => 
                await productService.GetProductsAsync(page, pageSize)
        );
    }
}
```

**XAML:**
```xml







```

**Reference:** https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.data.isupportincrementalloading

---

## collections-container-recycling.md

```markdown
---
title: Enable Container Recycling
impact: HIGH
impactDescription: Reduces memory allocations
tags: collections, recycling, containers, memory
---
```

### Enable Container Recycling

**Impact: HIGH** (Reduces memory allocations)

ListView and GridView automatically recycle containers, but you must ensure your DataTemplate doesn't prevent recycling.

**Incorrect** (prevents recycling by using x:Name):
```xml










```

**Correct** (allows proper recycling):
```xml










```

**If you need to access elements, use ChoosingItemContainer:**
```csharp
private void ListView_ChoosingItemContainer(
    ListViewBase sender, 
    ChoosingItemContainerEventArgs args)
{
    if (args.ItemContainer == null)
    {
        args.ItemContainer = new ListViewItem();
    }

    // Configure the container
    args.ItemContainer.Margin = new Thickness(4);
}
```

**Container Recycling Best Practices:**
- Avoid x:Name in DataTemplates
- Don't store container references
- Use x:Bind for automatic cleanup
- Reset state in ContainerContentChanging event

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/debug-test-perf/optimize-gridview-and-listview

---

## collections-data-template-selector.md

```markdown
---
title: Optimize DataTemplateSelector Performance
impact: MEDIUM-HIGH
impactDescription: Reduce template selection overhead
tags: collections, datatemplateselector, templates, performance
---
```

### Optimize DataTemplateSelector Performance

**Impact: MEDIUM-HIGH** (Reduce template selection overhead)

DataTemplateSelector is called for every item. Make SelectTemplateCore() as fast as possible.

**Incorrect** (expensive type checks and string comparisons):
```csharp
public class MessageTemplateSelector : DataTemplateSelector
{
    public DataTemplate TextTemplate { get; set; }
    public DataTemplate ImageTemplate { get; set; }
    public DataTemplate VideoTemplate { get; set; }

    protected override DataTemplate SelectTemplateCore(
        object item, DependencyObject container)
    {
        var message = item as Message;

        // Slow string comparison
        if (message?.Type?.ToLower() == "text")
            return TextTemplate;
        else if (message?.Type?.ToLower() == "image")
            return ImageTemplate;
        else if (message?.Type?.ToLower() == "video")
            return VideoTemplate;

        return base.SelectTemplateCore(item, container);
    }
}
```

**Correct** (enum-based, fast lookup):
```csharp
public enum MessageType { Text, Image, Video }

public class Message
{
    public MessageType Type { get; set; }
    // ... other properties
}

public class MessageTemplateSelector : DataTemplateSelector
{
    private readonly Dictionary _templateCache = new();

    public DataTemplate TextTemplate { get; set; }
    public DataTemplate ImageTemplate { get; set; }
    public DataTemplate VideoTemplate { get; set; }

    public MessageTemplateSelector()
    {
        // Build cache once
        InitializeTemplateCache();
    }

    private void InitializeTemplateCache()
    {
        _templateCache[MessageType.Text] = TextTemplate;
        _templateCache[MessageType.Image] = ImageTemplate;
        _templateCache[MessageType.Video] = VideoTemplate;
    }

    protected override DataTemplate SelectTemplateCore(
        object item, DependencyObject container)
    {
        if (item is Message message)
        {
            // Fast dictionary lookup
            return _templateCache[message.Type];
        }

        return base.SelectTemplateCore(item, container);
    }
}
```

**Performance Comparison:**
```
10,000 item list scrolling:

String comparison approach: ~120ms template selection time
Enum + Dictionary lookup: ~8ms template selection time

15× faster
```

**Reference:** https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.controls.datatemplateselector

---

## collections-grouped-virtualization.md

```markdown
---
title: Enable Virtualization for Grouped Collections
impact: HIGH
impactDescription: Improves grouped list performance
tags: collections, grouping, virtualization, performance
---
```

### Enable Virtualization for Grouped Collections

**Impact: HIGH** (Improves grouped list performance)

Grouped collections require special consideration to maintain virtualization benefits.

**Incorrect** (loses virtualization with grouping):
```xml











```

**Correct** (maintains virtualization with ItemsStackPanel):
```xml



















```

**ViewModel with proper grouping:**
```csharp
public class MainViewModel
{
    public CollectionViewSource GroupedItems { get; }

    public MainViewModel()
    {
        GroupedItems = new CollectionViewSource
        {
            IsSourceGrouped = true,
            ItemsPath = new PropertyPath("Items")
        };

        LoadGroupedData();
    }

    private async Task LoadGroupedData()
    {
        var items = await LoadItemsAsync();

        var groups = items
            .GroupBy(i => i.Category)
            .Select(g => new GroupInfoList
            {
                Key = g.Key,
                Items = g.ToList()
            });

        GroupedItems.Source = groups;
    }
}

public class GroupInfoList
{
    public string Key { get; set; }
    public List Items { get; set; }
}
```

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/design/controls-and-patterns/listview-and-gridview

---

# LAYOUT — Performance

## layout-fixed-sizes.md

```markdown
---
title: Use Fixed Sizes When Possible
impact: HIGH
impactDescription: Eliminates measure passes
tags: layout, sizing, measure, performance
---
```

### Use Fixed Sizes When Possible

**Impact: HIGH** (Eliminates measure passes)

Fixed sizes skip the measure phase of layout, significantly improving performance.

**Incorrect** (requires measurement):
```xml













```

**Correct** (fixed height eliminates measure):
```xml












```

**Performance Impact:**
```
1000 items in a list:

Auto-sized items:
- Measure pass: ~45ms
- Arrange pass: ~12ms
- Total: 57ms

Fixed-height items:
- Measure pass: ~2ms (skipped for most)
- Arrange pass: ~12ms
- Total: 14ms

4× faster layout
```

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/debug-test-perf/optimize-your-xaml-layout

---

## layout-grid-vs-stackpanel.md

```markdown
---
title: Prefer Grid Over Nested StackPanels
impact: HIGH
impactDescription: Reduces layout passes
tags: layout, grid, stackpanel, nesting
---
```

### Prefer Grid Over Nested StackPanels

**Impact: HIGH** (Reduces layout passes)

Grid performs single-pass layout while nested StackPanels require multiple passes.

**Incorrect** (nested StackPanels - multiple layout passes):
```xml














```

**Correct** (single Grid - one layout pass):
```xml






















```

**Performance Measurement:**
```
Complex form with 20 fields:

Nested StackPanels:
- Layout passes: 6
- Total layout time: ~28ms

Single Grid:
- Layout passes: 1
- Total layout time: ~8ms

3.5× faster
```

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/debug-test-perf/optimize-your-xaml-layout

---

## layout-relativepanel-complexity.md

```markdown
---
title: Limit RelativePanel Constraint Complexity
impact: MEDIUM-HIGH
impactDescription: Complex constraints are expensive
tags: layout, relativepanel, constraints, performance
---
```

### Limit RelativePanel Constraint Complexity

**Impact: MEDIUM-HIGH** (Complex constraints are expensive to resolve)

RelativePanel with many cross-dependencies requires expensive constraint solving. Use Grid for complex layouts.

**Incorrect** (complex RelativePanel constraints):
```xml











```

**Correct** (use Grid for predictable layout):
```xml



















```

**When to use RelativePanel:**
- Simple layouts with 2-3 elements
- Responsive designs with few breakpoints
- Constraints are mostly to panel edges

**When to use Grid:**
- Complex layouts with many elements
- Predictable row/column structure
- Performance is critical

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/design/layout/relativepanel

---

## layout-canvas-optimization.md

```markdown
---
title: Use Canvas for Absolute Positioning
impact: MEDIUM
impactDescription: Fastest panel for fixed positions
tags: layout, canvas, absolute-positioning, performance
---
```

### Use Canvas for Absolute Positioning

**Impact: MEDIUM** (Fastest panel for fixed positions)

When elements have absolute positions, Canvas is the most efficient panel since it skips measurement entirely.

**Incorrect** (Grid with margins for positioning):
```xml





```

**Correct** (Canvas for absolute positions):
```xml





```

**Performance with 1000 elements:**
```
Grid with Margins:
- Measure: ~35ms
- Arrange: ~15ms
- Total: 50ms

Canvas:
- Measure: ~2ms
- Arrange: ~8ms
- Total: 10ms

5× faster layout
```

**Best Use Cases:**
- Game overlays
- Data visualizations
- Drawing/diagramming apps
- Custom animations with absolute positioning

**Reference:** https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.controls.canvas

---

## layout-invalidate-arrange.md

```markdown
---
title: Minimize InvalidateArrange() Calls
impact: MEDIUM-HIGH
impactDescription: Reduces forced layout passes
tags: layout, invalidate, arrange, performance
---
```

### Minimize InvalidateArrange() Calls

**Impact: MEDIUM-HIGH** (Reduces forced layout passes)

Calling InvalidateArrange() forces a layout pass. Batch changes to minimize these expensive operations.

**Incorrect** (triggers layout for each property):
```csharp
foreach (var item in Items)
{
    var element = ContainerFromItem(item) as FrameworkElement;
    element.Width = newWidth;           // Triggers layout
    element.Height = newHeight;         // Triggers layout
    element.Margin = new Thickness(4);  // Triggers layout
}
```

**Correct** (batch updates):
```csharp
// Suspend layout updates
SuspendLayout();

foreach (var item in Items)
{
    var element = ContainerFromItem(item) as FrameworkElement;
    element.Width = newWidth;
    element.Height = newHeight;
    element.Margin = new Thickness(4);
}

// Resume and perform single layout pass
ResumeLayout();
InvalidateArrange();
```

**Better with manual batching:**
```csharp
public class LayoutBatchScope : IDisposable
{
    private readonly FrameworkElement _element;
    private bool _isDisposed;

    public LayoutBatchScope(FrameworkElement element)
    {
        _element = element;
        // Store current layout state
    }

    public void Dispose()
    {
        if (!_isDisposed)
        {
            _element.InvalidateArrange();
            _isDisposed = true;
        }
    }
}

// Usage:
using (new LayoutBatchScope(MyPanel))
{
    // Make multiple changes
    Child1.Width = 100;
    Child2.Height = 200;
    Child3.Visibility = Visibility.Collapsed;
} // Single layout pass happens here
```

**Reference:** https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.uielement.invalidatearrange

---

## layout-translation-vs-margin.md

```markdown
---
title: Use Translation Instead of Margin for Animations
impact: HIGH
impactDescription: Enables composition layer animations
tags: layout, animation, translation, composition, margin
---
```

### Use Translation Instead of Margin for Animations

**Impact: HIGH** (Enables composition layer animations)

Animating Margin triggers layout passes. Use Translation (RenderTransform) to animate on the composition layer for 60 FPS.

**Incorrect** (animates Margin - triggers layout):
```xml







```

**Correct** (animates Translation - no layout):
```xml










```

**Best (use Composition APIs):**
```csharp
private void AnimateButton()
{
    var visual = ElementCompositionPreview.GetElementVisual(SlideButton);
    var compositor = visual.Compositor;

    var animation = compositor.CreateVector3KeyFrameAnimation();
    animation.InsertKeyFrame(1.0f, new Vector3(100, 0, 0));
    animation.Duration = TimeSpan.FromMilliseconds(300);

    visual.StartAnimation("Offset", animation);
}
```

**Performance Comparison:**
```
Animating 50 elements simultaneously:

Margin Animation:
- FPS: ~25 FPS
- CPU: 85%
- Layout passes: 50 per frame

Translation/Composition:
- FPS: 60 FPS
- CPU: 12%
- Layout passes: 0

Composition is 5× more efficient
```

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/composition/composition-animation

---

# RENDERING — Composition & Visual Performance

## rendering-composition-animations.md

```markdown
---
title: Use Composition APIs for Smooth Animations
impact: HIGH
impactDescription: 60 FPS GPU-accelerated animations
tags: rendering, composition, animation, performance, 60fps
---
```

### Use Composition APIs for Smooth Animations

**Impact: HIGH** (60 FPS GPU-accelerated animations)

Composition animations run on a separate thread from UI, ensuring smooth 60 FPS even when UI thread is busy.

**Incorrect** (XAML Storyboard on UI thread):
```csharp
private void AnimateFadeIn()
{
    var storyboard = new Storyboard();
    var animation = new DoubleAnimation
    {
        From = 0,
        To = 1,
        Duration = TimeSpan.FromMilliseconds(300)
    };

    Storyboard.SetTarget(animation, MyElement);
    Storyboard.SetTargetProperty(animation, "Opacity");

    storyboard.Children.Add(animation);
    storyboard.Begin();
}
```

**Correct** (Composition API - runs independently):
```csharp
private void AnimateFadeIn()
{
    var visual = ElementCompositionPreview.GetElementVisual(MyElement);
    var compositor = visual.Compositor;

    var animation = compositor.CreateScalarKeyFrameAnimation();
    animation.InsertKeyFrame(0.0f, 0.0f);
    animation.InsertKeyFrame(1.0f, 1.0f);
    animation.Duration = TimeSpan.FromMilliseconds(300);

    visual.StartAnimation("Opacity", animation);
}
```

**Complex animation with easing:**
```csharp
private void AnimateSlideAndFade()
{
    var visual = ElementCompositionPreview.GetElementVisual(MyElement);
    var compositor = visual.Compositor;

    // Slide animation
    var offsetAnimation = compositor.CreateVector3KeyFrameAnimation();
    offsetAnimation.InsertKeyFrame(0.0f, new Vector3(0, 50, 0));
    offsetAnimation.InsertKeyFrame(1.0f, new Vector3(0, 0, 0));
    offsetAnimation.Duration = TimeSpan.FromMilliseconds(400);

    // Cubic bezier easing
    var easing = compositor.CreateCubicBezierEasingFunction(
        new Vector2(0.1f, 0.9f), 
        new Vector2(0.2f, 1.0f)
    );
    offsetAnimation.InsertKeyFrame(1.0f, new Vector3(0, 0, 0), easing);

    // Fade animation
    var opacityAnimation = compositor.CreateScalarKeyFrameAnimation();
    opacityAnimation.InsertKeyFrame(0.0f, 0.0f);
    opacityAnimation.InsertKeyFrame(1.0f, 1.0f);
    opacityAnimation.Duration = TimeSpan.FromMilliseconds(400);

    // Start both animations
    visual.StartAnimation("Offset", offsetAnimation);
    visual.StartAnimation("Opacity", opacityAnimation);
}
```

**Animatable Composition Properties:**
- Offset (position)
- Opacity
- Scale
- RotationAngle
- CenterPoint
- Size

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/composition/composition-animation

---

## rendering-connected-animations.md

```markdown
---
title: Implement ConnectedAnimation Efficiently
impact: MEDIUM-HIGH
impactDescription: Smooth page transitions
tags: rendering, connected-animation, navigation, transitions
---
```

### Implement ConnectedAnimation Efficiently

**Impact: MEDIUM-HIGH** (Smooth page transitions with element continuity)

ConnectedAnimation provides smooth element transitions between pages, but requires proper implementation.

**Incorrect** (no connected animation - jarring transition):
```csharp
// Source page
private void Item_Click(object sender, ItemClickEventArgs e)
{
    Frame.Navigate(typeof(DetailsPage), e.ClickedItem);
}

// Destination page  
protected override void OnNavigatedTo(NavigationEventArgs e)
{
    var item = e.Parameter as Item;
    ItemImage.Source = item.ImageSource;
}
```

**Correct** (smooth connected animation):
```csharp
// Source page (ListPage.xaml.cs)
private void Item_Click(object sender, ItemClickEventArgs e)
{
    var item = e.ClickedItem as Item;

    // Get the container for the clicked item
    var container = ItemsListView.ContainerFromItem(item) as ListViewItem;
    var image = FindVisualChild(container);

    // Prepare the connected animation
    ConnectedAnimationService.GetForCurrentView()
        .PrepareToAnimate("itemImage", image);

    Frame.Navigate(typeof(DetailsPage), item);
}

// Destination page (DetailsPage.xaml.cs)
protected override async void OnNavigatedTo(NavigationEventArgs e)
{
    base.OnNavigatedTo(e);

    var item = e.Parameter as Item;
    ViewModel.Item = item;

    // Wait for the layout to complete
    await Task.Delay(1);

    // Try to start the connected animation
    var animation = ConnectedAnimationService.GetForCurrentView()
        .GetAnimation("itemImage");

    if (animation != null)
    {
        // Configure animation
        animation.Configuration = new DirectConnectedAnimationConfiguration();

        // Start the animation
        await DetailImage.StartConnectedAnimationAsync(animation);
    }
}

// Helper to find visual child
private T FindVisualChild(DependencyObject parent) where T : DependencyObject
{
    for (int i = 0; i < VisualTreeHelper.GetChildrenCount(parent); i++)
    {
        var child = VisualTreeHelper.GetChild(parent, i);
        if (child is T typedChild)
            return typedChild;

        var result = FindVisualChild(child);
        if (result != null)
            return result;
    }
    return null;
}
```

**With ListView/GridView:**
```csharp
// Source page
private void ListView_ItemClick(object sender, ItemClickEventArgs e)
{
    // ListView provides built-in support
    ItemsListView.PrepareConnectedAnimation("itemImage", e.ClickedItem, "ItemImage");
    Frame.Navigate(typeof(DetailsPage), e.ClickedItem);
}

// Destination page
protected override async void OnNavigatedTo(NavigationEventArgs e)
{
    base.OnNavigatedTo(e);

    var item = e.Parameter as Item;
    ViewModel.Item = item;

    var animation = ConnectedAnimationService.GetForCurrentView()
        .GetAnimation("itemImage");

    if (animation != null)
    {
        animation.Configuration = new DirectConnectedAnimationConfiguration();
        await DetailImage.StartConnectedAnimationAsync(animation);
    }
}
```

**Animation Configurations:**
- `DirectConnectedAnimationConfiguration`: Fast, direct movement
- `GravityConnectedAnimationConfiguration`: Natural, gravity-like animation
- `BasicConnectedAnimationConfiguration`: Simple fade and scale

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/design/motion/connected-animation

---

## rendering-backdrop-blur.md

```markdown
---
title: Limit Expensive Blur Effects
impact: MEDIUM-HIGH
impactDescription: Blur is GPU-intensive
tags: rendering, blur, acrylic, performance, gpu
---
```

### Limit Expensive Blur Effects

**Impact: MEDIUM-HIGH** (Blur effects are GPU-intensive)

Acrylic and blur effects look beautiful but are expensive. Use sparingly and disable during animations.

**Incorrect** (blur on large animated surfaces):
```xml










```

**Correct** (blur only on static, small surfaces):
```xml




















```

**Disable blur during animations:**
```csharp
private SolidColorBrush _fallbackBrush;
private AcrylicBrush _acrylicBrush;

private void StartAnimation()
{
    // Switch to solid color during animation
    if (MyPanel.Background is AcrylicBrush acrylic)
    {
        _acrylicBrush = acrylic;
        _fallbackBrush = new SolidColorBrush(acrylic.TintColor);
        MyPanel.Background = _fallbackBrush;
    }

    // Perform animation...
}

private void AnimationCompleted()
{
    // Restore acrylic after animation
    if (_acrylicBrush != null)
    {
        MyPanel.Background = _acrylicBrush;
    }
}
```

**Performance Impact:**
```
Full-screen acrylic while scrolling:
- FPS: ~30 FPS
- GPU: 95%

Solid color while scrolling:
- FPS: 60 FPS
- GPU: 35%
```

**Best Practices:**
- Use `BackgroundSource="Backdrop"` instead of `"HostBackdrop"` when possible
- Limit acrylic to small, static surfaces
- Use solid colors during animations
- Consider dark theme alternatives to blur

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/design/style/acrylic

---

## rendering-opacity-vs-visibility.md

```markdown
---
title: Use Visibility Instead of Opacity=0
impact: MEDIUM
impactDescription: Collapsed elements don't render
tags: rendering, visibility, opacity, performance
---
```

### Use Visibility Instead of Opacity=0

**Impact: MEDIUM** (Collapsed elements skip rendering and hit-testing)

Setting `Opacity="0"` still renders the element. Use `Visibility.Collapsed` to completely skip rendering.

**Incorrect** (element still rendered and hit-tested):
```xml






```

**Correct** (element not rendered at all):
```xml
<Grid Visibility="{x:Bind ViewModel.IsVisible, 
                   Converter={StaticResource BoolToVisibilityConverter}}">
    <controls:ExpensiveControl />
    <Canvas>
        <!-- Complex graphics -->
    </Canvas>
</Grid>
```

---

</Canvas>
</Grid>

**Better** (with x:Load for deferred loading):
```xml
<Grid x:Load="{x:Bind ViewModel.IsVisible, Mode=OneWay}">
    <controls:ExpensiveControl />
    <Canvas>
        <!-- Complex graphics -->
    </Canvas>
</Grid>
```

**Performance Comparison:**
```
Hidden element with complex content (1000 sub-elements):

Opacity=0:
- Render pass: ~25ms
- Hit testing: Active
- Memory: Full allocation

Visibility.Collapsed:
- Render pass: ~0ms
- Hit testing: Skipped
- Memory: Full allocation

x:Load=false:
- Render pass: ~0ms
- Hit testing: Skipped
- Memory: Released
```

**When to use each:**
- **Opacity**: Fade animations, visual effects
- **Visibility.Collapsed**: Toggle UI, keep in memory
- **x:Load**: Conditional UI, release memory when hidden

**Reference:** https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.uielement.visibility
````

#### **rendering-shadow-performance.md**
````markdown
---
title: Minimize Drop Shadow Usage
impact: MEDIUM
impactDescription: Shadows are expensive to render
tags: rendering, shadow, dropshadow, performance
---

# Minimize Drop Shadow Usage

**Impact: MEDIUM** (Shadows require multiple render passes)

Drop shadows, especially with large blur radii, are expensive. Use CSS-style solid shadows or limit shadow count.

**Incorrect** (expensive composition shadows on many elements):
```csharp
foreach (var card in CardElements)
{
    var visual = ElementCompositionPreview.GetElementVisual(card);
    var compositor = visual.Compositor;

    var shadow = compositor.CreateDropShadow();
    shadow.BlurRadius = 32f;
    shadow.Offset = new Vector3(0, 8, 0);
    shadow.Color = Colors.Black;
    shadow.Opacity = 0.3f;

    var spriteVisual = compositor.CreateSpriteVisual();
    spriteVisual.Shadow = shadow;
    spriteVisual.Size = new Vector2((float)card.ActualWidth, (float)card.ActualHeight);

    ElementCompositionPreview.SetElementChildVisual(card, spriteVisual);
}
```

**Correct** (limit shadows to key UI elements):
```csharp
// Apply shadows only to top-level cards, not nested elements
private void ApplyCardShadow(FrameworkElement card)
{
    var visual = ElementCompositionPreview.GetElementVisual(card);
    var compositor = visual.Compositor;

    var shadow = compositor.CreateDropShadow();
    shadow.BlurRadius = 16f; // Smaller blur radius
    shadow.Offset = new Vector3(0, 4, 0);
    shadow.Color = Colors.Black;
    shadow.Opacity = 0.2f;

    var spriteVisual = compositor.CreateSpriteVisual();
    spriteVisual.Shadow = shadow;
    spriteVisual.Size = new Vector2((float)card.ActualWidth, (float)card.ActualHeight);

    ElementCompositionPreview.SetElementChildVisual(card, spriteVisual);
}
```

**Alternative: CSS-style border for subtle elevation:**
```xml
<Border Background="White"
        CornerRadius="8"
        BorderBrush="#10000000"
        BorderThickness="0,0,0,2">
    <StackPanel Padding="16">
        <TextBlock Text="Card Content" />
    </StackPanel>
</Border>
```

**Performance with 100 cards:**
````
Composition DropShadow (BlurRadius=32):
- FPS: ~35 FPS
- GPU: 85%

Composition DropShadow (BlurRadius=8):
- FPS: ~52 FPS
- GPU: 60%

Border-based shadow:
- FPS: 60 FPS
- GPU: 25%
````

**Best Practices:**
- Limit to 5-10 shadows on screen at once
- Use smaller blur radii (8-16 instead of 32-64)
- Consider border-based shadows for subtle elevation
- Disable shadows during scrolling/animations

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/composition/composition-shadows
````

#### **rendering-rasterization-cache.md**
````markdown
---
title: Cache Complex Visuals with CacheMode
impact: MEDIUM-HIGH
impactDescription: Reduces redraw overhead
tags: rendering, cache, rasterization, performance
---

# Cache Complex Visuals with CacheMode

**Impact: MEDIUM-HIGH** (Caches rendered output to reduce redraw)

For static or infrequently changing complex visuals, use BitmapCache to cache the rendered output.

**Incorrect** (redraws complex visual every frame):
```xml
<Canvas>
    <!-- Complex vector graphics redrawn constantly -->
    <Path Data="M 100,200 L 200,100 ..." 
          Stroke="Black" 
          StrokeThickness="2">
        <Path.Fill>
            <LinearGradientBrush>
                <GradientStop Color="Red" Offset="0" />
                <GradientStop Color="Blue" Offset="1" />
            </LinearGradientBrush>
        </Path.Fill>
    </Path>
    <!-- 50 more complex paths -->
</Canvas>
```

**Correct** (caches rendered output as bitmap):
```xml
<Canvas CacheMode="BitmapCache">
    <!-- Rendered once, cached as bitmap -->
    <Path Data="M 100,200 L 200,100 ..." 
          Stroke="Black" 
          StrokeThickness="2">
        <Path.Fill>
            <LinearGradientBrush>
                <GradientStop Color="Red" Offset="0" />
                <GradientStop Color="Blue" Offset="1" />
            </LinearGradientBrush>
        </Path.Fill>
    </Path>
    <!-- 50 more complex paths -->
</Canvas>
```

**Dynamic cache invalidation:**
```csharp
private void UpdateComplexVisual()
{
    // Temporarily disable cache during update
    ComplexCanvas.CacheMode = null;

    // Make changes
    UpdatePaths();
    UpdateColors();

    // Re-enable cache after changes
    ComplexCanvas.CacheMode = new BitmapCache();
}
```

**When to use BitmapCache:**
- ✅ Static complex vector graphics
- ✅ Rarely changing rich text
- ✅ Complex layered UI that moves as a unit
- ❌ Frequently changing content
- ❌ Animated elements
- ❌ Simple UI with few elements

**Performance Comparison (complex vector with 100 paths):**
````
Without cache (redraws each frame):
- Render time: ~18ms per frame
- FPS: ~55 FPS

With BitmapCache:
- Initial render: ~18ms
- Cached render: ~0.5ms per frame
- FPS: 60 FPS

36× faster for cached frames
````

**Memory Trade-off:**
````
800x600 cached surface:
- Memory usage: ~1.8 MB per cached element
- CPU saved: ~15ms per frame

Balance memory vs CPU based on your needs.
````

**Reference:** https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.media.bitmapcache
````

---

### **MEMORY SECTION - Management**

#### **memory-event-handlers.md**
````markdown
---
title: Unsubscribe Event Handlers in Unloaded
impact: CRITICAL
impactDescription: Prevents memory leaks
tags: memory, events, leaks, unloaded
---

# Unsubscribe Event Handlers in Unloaded

**Impact: CRITICAL** (Prevents memory leaks)

Event subscriptions keep objects alive. Always unsubscribe in the Unloaded event to prevent memory leaks.

**Incorrect** (memory leak - page never garbage collected):
```csharp
public sealed partial class MyPage : Page
{
    private readonly MyService _service;

    public MyPage()
    {
        InitializeComponent();

        _service = App.Current.GetService<MyService>();
        _service.DataChanged += OnDataChanged; // LEAK: Never unsubscribed

        Application.Current.Suspending += OnSuspending; // LEAK
    }

    private void OnDataChanged(object sender, EventArgs e)
    {
        UpdateUI();
    }
}
```

**Correct** (proper cleanup):
```csharp
public sealed partial class MyPage : Page
{
    private readonly MyService _service;

    public MyPage()
    {
        InitializeComponent();

        _service = App.Current.GetService<MyService>();
        _service.DataChanged += OnDataChanged;

        Application.Current.Suspending += OnSuspending;

        // Subscribe to Unloaded for cleanup
        Unloaded += MyPage_Unloaded;
    }

    private void MyPage_Unloaded(object sender, RoutedEventArgs e)
    {
        // Unsubscribe from all events
        _service.DataChanged -= OnDataChanged;
        Application.Current.Suspending -= OnSuspending;
        Unloaded -= MyPage_Unloaded;
    }

    private void OnDataChanged(object sender, EventArgs e)
    {
        UpdateUI();
    }
}
```

**Better with WeakEventListener (CommunityToolkit):**
```csharp
using CommunityToolkit.WinUI;

public sealed partial class MyPage : Page
{
    private readonly MyService _service;
    private readonly WeakEventListener<MyPage, MyService, EventArgs> _weakListener;

    public MyPage()
    {
        InitializeComponent();

        _service = App.Current.GetService<MyService>();

        // WeakEventListener prevents memory leak
        _weakListener = new WeakEventListener<MyPage, MyService, EventArgs>(this)
        {
            OnEventAction = (instance, source, args) => instance.OnDataChanged(source, args),
            OnDetachAction = (listener) => _service.DataChanged -= listener.OnEvent
        };

        _service.DataChanged += _weakListener.OnEvent;
    }

    private void OnDataChanged(object sender, EventArgs e)
    {
        UpdateUI();
    }
}
```

**Common leak sources:**
- Static event handlers
- Application-level events
- Service events
- Timer events
- Dispatcher events
- Observable collection events

**Reference:** https://docs.microsoft.com/en-us/windows/uwp/debug-test-perf/reduce-memory-usage
````
