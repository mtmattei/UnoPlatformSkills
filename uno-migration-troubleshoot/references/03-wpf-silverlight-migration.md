# WPF and Silverlight Migration Reference

Detailed patterns for migrating WPF and Silverlight applications to Uno Platform, including API mapping, namespace changes, unsupported feature workarounds, and data access modernization.

---

### Map WPF Core APIs to Uno Platform Equivalents

**Rule**: Use the WPF-to-Uno API mapping table as the first step in migration planning; identify every WPF type in your codebase and find its Uno Platform equivalent before writing any code.

**Why**: WPF and WinUI/Uno Platform share XAML concepts but differ in type names, namespaces, and behavior. Attempting to migrate code file-by-file without a mapping reference leads to cascading errors that are time-consuming to untangle. A systematic mapping pass identifies the full scope of changes upfront, enabling accurate effort estimates and parallel work across the team.

**Example (API mapping table)**:

| WPF Type/Pattern | Uno Platform Equivalent | Notes |
|---|---|---|
| `Window` | `Page` | Uno Platform uses single-window model; each "window" becomes a Page in the navigation frame |
| `{Binding Path=Name}` | `{x:Bind Name}` | `x:Bind` is preferred for type safety and performance; `{Binding}` still works but is less efficient |
| `ICommand` / custom impl | `[RelayCommand]` attribute | Use CommunityToolkit.Mvvm source generators or MVUX auto-commands |
| `DataGrid` | `ListView` with custom template | No built-in DataGrid; use ListView with GridView or third-party via MAUI Embedding |
| `System.Windows.Controls.*` | `Microsoft.UI.Xaml.Controls.*` | Full namespace change required |
| `System.Windows.Media.*` | `Microsoft.UI.Xaml.Media.*` | Brushes, transforms, and visual effects |
| `System.Windows.Data.Binding` | `Microsoft.UI.Xaml.Data.Binding` | Same concept, different namespace |
| `Dispatcher.Invoke()` | `DispatcherQueue.TryEnqueue()` | Asynchronous by default; no synchronous Invoke equivalent |
| `RoutedUICommand` | Button with `Command` binding | WPF command routing model does not exist; use MVVM commands |
| `Style.Triggers` | `VisualStateManager` | Property triggers do not exist; use visual states |
| `DataTrigger` | `VisualStateManager` or converter | Bind to state via converters or computed properties |
| `UserControl` | `UserControl` | Same concept, different namespace |
| `ContentControl` | `ContentControl` | Same concept, different namespace |

**Common Mistakes**:
- Searching for 1:1 replacements for every WPF feature (some have no direct equivalent)
- Using `{Binding}` everywhere instead of adopting `{x:Bind}` during migration (missed performance opportunity)
- Assuming WPF `Window` maps to a WinUI `Window` (Uno Platform apps are primarily Page-based)
- Not accounting for the removal of `RoutedUICommand` and the WPF command routing infrastructure

**Reference**: https://platform.uno/docs/articles/howto-migrate-wpf.html

---

### Replace System.Windows Namespaces with Microsoft.UI.Xaml

**Rule**: Replace all `System.Windows.*` namespace references with their `Microsoft.UI.Xaml.*` equivalents in both XAML and C# files; use find-and-replace across the entire solution with manual verification of each change.

**Why**: WPF uses the `System.Windows` root namespace while WinUI (and by extension Uno Platform) uses `Microsoft.UI.Xaml`. This is not a simple prefix swap in all cases. Some types moved to different sub-namespaces or were renamed. A bulk find-and-replace handles 90% of cases, but the remaining 10% require manual intervention.

**Example (C# - namespace changes)**:
```csharp
// WPF
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
using System.Windows.Input;
using System.Windows.Data;
using System.Windows.Threading;

// Uno Platform
using Microsoft.UI.Xaml;
using Microsoft.UI.Xaml.Controls;
using Microsoft.UI.Xaml.Media;
using Microsoft.UI.Xaml.Input;
using Microsoft.UI.Xaml.Data;
using Microsoft.UI.Dispatching; // Note: different sub-namespace
```

**Example (XAML - namespace declarations)**:
```xml
<!-- WPF -->
<Window xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation">

<!-- Uno Platform -->
<Page xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation">
<!-- The default XAML namespace URI is the same, but the root element changes -->
```

**Example (find-and-replace sequence)**:
```
1. System.Windows.Threading  -->  Microsoft.UI.Dispatching
2. System.Windows.Controls   -->  Microsoft.UI.Xaml.Controls
3. System.Windows.Media      -->  Microsoft.UI.Xaml.Media
4. System.Windows.Input      -->  Microsoft.UI.Xaml.Input
5. System.Windows.Data       -->  Microsoft.UI.Xaml.Data
6. System.Windows.Shapes     -->  Microsoft.UI.Xaml.Shapes
7. System.Windows            -->  Microsoft.UI.Xaml   (do this LAST to avoid partial replacements)
```

**Common Mistakes**:
- Replacing `System.Windows` before the more specific sub-namespaces, causing incorrect results like `Microsoft.UI.Xaml.Controls` becoming `Microsoft.UI.Xaml.Xaml.Controls`
- Missing the `Threading` to `Dispatching` namespace change (it does not follow the same pattern)
- Not updating XAML `xmlns:local` references that point to namespaces containing `System.Windows` types
- Forgetting to update assembly-qualified type names in string-based configurations (e.g., converters registered by name)

**Uno Platform Notes**: The default XAML namespace (`http://schemas.microsoft.com/winfx/2006/xaml/presentation`) is shared between WPF and Uno Platform. You do not need to change `xmlns=` declarations in XAML files. However, custom namespace mappings using `xmlns:local="clr-namespace:..."` must be updated to use `using:` syntax instead.

**Reference**: https://platform.uno/docs/articles/howto-migrate-wpf.html

---

### Replace Unsupported XAML Features with Uno Platform Alternatives

**Rule**: Identify and replace `x:Static`, `MultiBinding`, `DataTrigger`, `Style.Triggers`, and `EventTrigger` usages with Uno Platform-compatible alternatives before attempting to compile.

**Why**: These features rely on WPF-specific XAML parser extensions that do not exist in WinUI or Uno Platform. They produce XAML parse errors at compile time (for `x:Static`) or runtime (for triggers). Identifying them upfront prevents a frustrating cycle of fix-compile-discover-repeat.

**Example (x:Static replacement)**:
```xml
<!-- WPF - NOT SUPPORTED -->
<TextBlock Text="{x:Static local:Constants.AppTitle}" />

<!-- Uno Platform - use StaticResource -->
<!-- In App.xaml or a resource dictionary: -->
<x:String x:Key="AppTitle">My Application</x:String>

<!-- In the page: -->
<TextBlock Text="{StaticResource AppTitle}" />
```

**Example (MultiBinding replacement)**:
```xml
<!-- WPF - NOT SUPPORTED -->
<TextBlock>
    <TextBlock.Text>
        <MultiBinding StringFormat="{}{0} - {1}">
            <Binding Path="FirstName" />
            <Binding Path="LastName" />
        </MultiBinding>
    </TextBlock.Text>
</TextBlock>

<!-- Uno Platform - use multiple Run elements -->
<TextBlock>
    <Run Text="{x:Bind ViewModel.FirstName}" />
    <Run Text=" - " />
    <Run Text="{x:Bind ViewModel.LastName}" />
</TextBlock>

<!-- OR use a computed property in the ViewModel -->
<!-- public string FullName => $"{FirstName} - {LastName}"; -->
<TextBlock Text="{x:Bind ViewModel.FullName}" />
```

**Example (DataTrigger replacement)**:
```xml
<!-- WPF - NOT SUPPORTED -->
<Style.Triggers>
    <DataTrigger Binding="{Binding IsActive}" Value="True">
        <Setter Property="Background" Value="Green" />
    </DataTrigger>
</Style.Triggers>

<!-- Uno Platform - use VisualStateManager -->
<VisualStateManager.VisualStateGroups>
    <VisualStateGroup>
        <VisualState x:Name="Active">
            <VisualState.StateTriggers>
                <StateTrigger IsActive="{x:Bind ViewModel.IsActive, Mode=OneWay}" />
            </VisualState.StateTriggers>
            <VisualState.Setters>
                <Setter Target="RootBorder.Background" Value="Green" />
            </VisualState.Setters>
        </VisualState>
    </VisualStateGroup>
</VisualStateManager.VisualStateGroups>
```

**Common Mistakes**:
- Attempting to create a custom markup extension to emulate `x:Static` (not supported in the Uno Platform XAML parser)
- Using `{Binding StringFormat=...}` as a MultiBinding workaround (StringFormat on Binding is also not supported in Uno Platform)
- Wrapping trigger logic in code-behind event handlers instead of using VisualStateManager (breaks MVVM)
- Not searching for `MultiBinding` in resource dictionaries and styles, only in page XAML

**Uno Platform Notes**: WinUI (and Uno Platform) intentionally dropped WPF triggers in favor of VisualStateManager for performance and consistency. `StateTrigger` and `AdaptiveTrigger` provide declarative state management without the overhead of WPF's trigger evaluation system.

**Reference**: https://platform.uno/docs/articles/howto-migrate-wpf.html

---

### Migrate WPF Navigation to Uno Platform Page Navigation

**Rule**: Replace WPF `NavigationWindow`/`Frame.Navigate` patterns with Uno Platform Navigation Extensions using XAML-based `Navigation.Request` attached properties; do not replicate WPF navigation logic in code-behind.

**Why**: WPF navigation uses a `NavigationWindow` or `Frame` with code-behind `Navigate()` calls and URI-based addressing. Uno Platform Navigation Extensions use a declarative, route-based system with XAML attached properties. The code-behind approach is fragile, hard to test, and does not support deep linking or platform-specific navigation patterns (Android back button, iOS swipe-back).

**Example (WPF - code-behind navigation)**:
```csharp
// WPF pattern - DO NOT replicate in Uno Platform
private void OnDetailsClick(object sender, RoutedEventArgs e)
{
    var item = (MyItem)((Button)sender).DataContext;
    NavigationService.Navigate(new Uri("/DetailsPage.xaml?id=" + item.Id, UriKind.Relative));
}
```

**Example (Uno Platform - declarative navigation)**:
```xml
<!-- XAML - preferred approach with Navigation Extensions -->
<Button Content="View Details"
        uen:Navigation.Request="Details"
        uen:Navigation.Data="{x:Bind ViewModel.SelectedItem}" />
```

```csharp
// Route registration in App.xaml.cs
private static void RegisterRoutes(IViewRegistry views, IRouteRegistry routes)
{
    views.Register(
        new ViewMap<MainPage, MainViewModel>(),
        new ViewMap<DetailsPage, DetailsViewModel>()
    );
    routes.Register(
        new RouteMap("", View: views.FindByViewModel<MainViewModel>(),
            Nested: new RouteMap[]
            {
                new RouteMap("Details", View: views.FindByViewModel<DetailsViewModel>())
            }
        )
    );
}
```

**Common Mistakes**:
- Replicating WPF `Frame.Navigate()` calls in code-behind instead of using Navigation Extensions
- Using URI-based navigation strings instead of typed route names
- Not registering routes in `App.xaml.cs`, causing "Route not found" runtime errors
- Forgetting to add `xmlns:uen="using:Uno.Extensions.Navigation.UI"` to XAML files

**Uno Platform Notes**: Uno Platform supports both Frame-based navigation (similar to UWP) and Region-based navigation (Uno Navigation Extensions). For WPF migrations, Region-based navigation with `Navigation.Request` is recommended because it supports deep linking, back-stack management, and platform-native navigation patterns out of the box.

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Overview/Navigation/NavigationOverview.html

---

### Migrate Silverlight Page/Frame Navigation

**Rule**: Replace Silverlight `Page`/`Frame` navigation with Uno Platform Navigation Extensions; map Silverlight's URI-based navigation to typed route registrations.

**Why**: Silverlight navigation relies on URI fragments (`#/page/id`) and a built-in `NavigationService`. This model does not exist in WinUI or Uno Platform. Uno Navigation Extensions provide an equivalent declarative navigation system with type safety, dependency injection integration, and cross-platform back-button support that Silverlight's model lacked.

**Example (Silverlight - URI-based navigation)**:
```csharp
// Silverlight pattern
NavigationService.Navigate(new Uri("/Views/OrderDetails.xaml?orderId=123", UriKind.Relative));

// In the target page:
protected override void OnNavigatedTo(NavigationEventArgs e)
{
    var orderId = NavigationContext.QueryString["orderId"];
}
```

**Example (Uno Platform - typed navigation)**:
```csharp
// Route registration
views.Register(new ViewMap<OrderDetailsPage, OrderDetailsViewModel>());
routes.Register(
    new RouteMap("OrderDetails", View: views.FindByViewModel<OrderDetailsViewModel>())
);

// ViewModel receives data via constructor injection
public partial record OrderDetailsViewModel(int OrderId);
```

```xml
<!-- XAML navigation with data -->
<Button Content="View Order"
        uen:Navigation.Request="OrderDetails"
        uen:Navigation.Data="{x:Bind ViewModel.SelectedOrderId}" />
```

**Common Mistakes**:
- Trying to replicate Silverlight's `NavigationContext.QueryString` pattern (does not exist in Uno Platform)
- Passing data through static variables or singletons instead of using Navigation.Data
- Not registering ViewModels with the DI container, causing resolution failures during navigation
- Keeping Silverlight-style `OnNavigatedTo` overrides for data loading instead of using MVUX Feeds or async ViewModel initialization

**Uno Platform Notes**: Silverlight's child window (popup) pattern maps to Uno Platform's dialog navigation. Use `Navigation.Request` with a route that is registered as a dialog to get modal behavior equivalent to Silverlight's `ChildWindow`.

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Overview/Navigation/NavigationOverview.html

---

### Replace Silverlight RIA Services with Modern HTTP Clients

**Rule**: Replace WCF RIA Services (DomainService, DomainContext, EntityQuery) with Kiota-generated or Refit-based HTTP clients backed by REST or GraphQL APIs.

**Why**: WCF RIA Services is a Silverlight-era technology with no .NET 9 support and no WinUI equivalent. Uno Platform recommends Kiota (for OpenAPI/REST) or Refit (for typed REST) as modern replacements. These integrate with Uno.Extensions dependency injection and support all target platforms including WebAssembly.

**Example (Silverlight RIA - legacy pattern)**:
```csharp
// Silverlight RIA Services - NOT SUPPORTED
var context = new OrderDomainContext();
var query = context.GetOrdersQuery();
context.Load(query, LoadBehavior.MergeIntoCurrent, OnOrdersLoaded, null);

private void OnOrdersLoaded(LoadOperation<Order> operation)
{
    if (!operation.HasError)
    {
        OrdersList.ItemsSource = operation.Entities;
    }
}
```

**Example (Uno Platform - Kiota HTTP client)**:
```csharp
// Service registration in App.xaml.cs
host.ConfigureServices((context, services) =>
{
    services.AddHttpClient<IOrderApiClient, OrderApiClient>(client =>
    {
        client.BaseAddress = new Uri("https://api.example.com");
    });
});

// ViewModel with MVUX Feed
public partial record OrderListViewModel(IOrderApiClient ApiClient)
{
    public IListFeed<Order> Orders => ListFeed.Async(
        async ct => await ApiClient.Orders.GetAsync(cancellationToken: ct)
    );
}
```

**Example (Uno Platform - Refit)**:
```csharp
// Define the API interface
public interface IOrderApi
{
    [Get("/api/orders")]
    Task<List<Order>> GetOrdersAsync(CancellationToken ct = default);

    [Get("/api/orders/{id}")]
    Task<Order> GetOrderAsync(int id, CancellationToken ct = default);
}

// Registration
host.UseHttp((context, services) =>
{
    services.AddRefitClient<IOrderApi>(context);
});
```

**Common Mistakes**:
- Trying to find a .NET 9 WCF RIA Services replacement (none exists; the architecture must change)
- Creating synchronous HTTP wrappers to mimic RIA Services' `Load()` pattern instead of using async/await
- Not handling cancellation tokens in HTTP calls, causing wasm timeouts and mobile battery drain
- Hardcoding API base URLs instead of using configuration-based endpoints

**Uno Platform Notes**: Kiota is the preferred HTTP client for new Uno Platform projects because it generates strongly-typed clients from OpenAPI specifications. Refit is recommended when you want to define the API contract manually as a C# interface. Both integrate with Uno.Extensions `UseHttp()` for DI registration.

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Overview/Http/HttpOverview.html

---

### Replace Silverlight IsolatedStorage and Toolkit Controls

**Rule**: Replace `IsolatedStorage` with `ApplicationData.Current.LocalFolder` and map Silverlight Toolkit controls to their Uno Toolkit equivalents.

**Why**: `IsolatedStorage` was Silverlight's sandboxed file storage API. It does not exist in WinUI or Uno Platform. `ApplicationData.Current.LocalFolder` provides equivalent cross-platform local storage with proper sandboxing on all targets. Similarly, Silverlight Toolkit controls (Accordion, AutoCompleteBox, Rating, etc.) have no direct WinUI equivalent but many have Uno Toolkit counterparts.

**Example (IsolatedStorage replacement)**:
```csharp
// Silverlight - NOT SUPPORTED
using System.IO.IsolatedStorage;

var store = IsolatedStorageFile.GetUserStoreForApplication();
using var stream = store.CreateFile("settings.json");
// write data

// Uno Platform - cross-platform local storage
var localFolder = Windows.Storage.ApplicationData.Current.LocalFolder;
var file = await localFolder.CreateFileAsync("settings.json",
    CreationCollisionOption.ReplaceExisting);
await FileIO.WriteTextAsync(file, jsonContent);
```

**Example (Toolkit control mapping)**:

| Silverlight Toolkit | Uno Platform Equivalent |
|---|---|
| `Accordion` | Custom expander list using `Expander` control |
| `AutoCompleteBox` | `AutoSuggestBox` (built-in WinUI control) |
| `BusyIndicator` | `LoadingView` (Uno Toolkit) |
| `DatePicker` / `TimePicker` | `DatePicker` / `TimePicker` (built-in WinUI) |
| `NumericUpDown` | `NumberBox` (built-in WinUI) |
| `Rating` | `RatingControl` (built-in WinUI) |
| `TabControl` | `TabBar` (Uno Toolkit) or `TabView` (WinUI) |
| `TreeView` | `TreeView` (built-in WinUI) |
| `WrapPanel` | `ItemsWrapGrid` or Uno Toolkit `AutoLayout` with wrapping |
| `ChildWindow` | Dialog navigation via Navigation Extensions |

**Common Mistakes**:
- Using `System.IO.File` directly instead of `ApplicationData.Current.LocalFolder` (works on Desktop but fails on mobile/wasm due to sandboxing)
- Assuming all Silverlight Toolkit controls have 1:1 equivalents (some require custom implementations)
- Not testing storage APIs on WebAssembly (local storage maps to IndexedDB, with different size limits)
- Migrating Silverlight `ChildWindow` as a custom popup instead of using Uno Platform's dialog navigation

**Uno Platform Notes**: For WebAssembly, `ApplicationData.Current.LocalFolder` maps to the browser's IndexedDB storage. File operations are asynchronous and have platform-specific size limits. For large data sets, consider using Uno.Extensions Storage or a client-side database like LiteDB.

**Reference**: https://platform.uno/docs/articles/features/windows-storage.html
