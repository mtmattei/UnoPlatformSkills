---
name: uno-platform-agent
description: "Comprehensive Uno Platform development patterns for Single Project architecture, MVVM/MVUX, navigation, styling, platform-specific code, and custom controls. Use when: (1) Creating new Uno Platform projects, (2) Implementing MVVM or MVUX patterns, (3) Setting up navigation or styling, (4) Writing platform-specific code, (5) Building custom controls, (6) Optimizing build configuration with Uno.Sdk"
license: "Apache 2.0 (patterns from Uno Platform Extensions)"
---

# Uno Platform Development Patterns

Best practices extracted from uno.extensions (50+ modules) and 17+ production applications.

## Critical Rules

**Project Setup**
- Use Uno.Sdk with `UnoSingleProject=true` for all new projects (5.2+)
- Configure UnoFeatures based on app requirements
- Target frameworks: net9.0-android;net9.0-ios;net9.0-browserwasm;net9.0-desktop
- Always include net9.0 (or net8.0) as base TFM at the end - NOT first, due to VS2022 debugging issues (UNOB0011, UNOB0013)
- Organize platform-specific code in Platforms/ folder

**Build Configuration**
- Use Directory.Build.props for shared configuration across projects
- Enable central package management with Directory.Packages.props
- Conditional TFM assignment based on platform flags
- Build performance: enable hard links, binding redirects
- Output paths: `bin\$(MSBuildProjectName)` to avoid conflicts

## High Priority Rules

**MVVM & MVUX**
- Prefer MVUX (reactive) for new applications; CommunityToolkit.Mvvm for traditional MVVM
- `[ObservableProperty]` and `[RelayCommand]` attributes for MVVM
- IListFeed for read-only data, IListState for mutable data
- MVUX auto-generates commands from public async methods
- `{Binding StringFormat=...}` is NOT supported - use multiple `<Run>` elements or computed properties

**Navigation**
- Set up with `.UseNavigation(RegisterRoutes)`
- Register ViewMaps and RouteMaps in RegisterRoutes method
- Prefer XAML-based navigation (`Navigation.Request`) over code-behind
- INavigator service for programmatic navigation

**Platform-Specific Code**
- Compile-time: `#if __ANDROID__`, `#if __IOS__`, `#if __WINDOWS__`, `#if __WASM__`
- Runtime: `OperatingSystem.IsAndroid()`, `IsBrowser()`, etc.
- Partial classes for platform-specific implementations

## Medium Priority Rules

**Styling & Theming**
- MaterialToolkitTheme with ColorPaletteOverride.xaml
- Never hardcode colors - use `{ThemeResource}` references
- Material Design 3 color naming; predefined typography styles (TitleLarge, BodyMedium, etc.)
- ThemeShadow for elevation with Translation property
- Responsive design with `{utu:Responsive}` extension
- AutoLayout over StackPanel for consistent spacing

**Data & HTTP**
- Configure HTTP clients with `.UseHttp()`
- HttpKiota preferred for new projects; Refit also supported
- Configuration in appsettings.json as EmbeddedResource
- ValueTask for async operations that often complete synchronously

**Custom Controls**
- SKCanvasElement for custom rendering with SkiaSharp
- Separate Renderer from Control for clean architecture
- ItemsRepeater for better performance than ListView
- Lottie animations for splash screens and loading indicators

## uno.extensions Conventions

### Dependency Injection
```csharp
public static IHostBuilder UseAuthentication(
    this IHostBuilder builder,
    Action<IAuthenticationBuilder> build)
{
    return builder.ConfigureServices((ctx, services) =>
    {
        services.TryAddSingleton<IAuthenticationService, AuthenticationService>();
    });
}
```

### Builder Pattern
```csharp
public interface IFeatureBuilder : IBuilder
{
    IHostBuilder? HostBuilder { get; init; }
}

public record FeatureBuilder : BaseBuilder, IFeatureBuilder
{
    public FeatureSettings Settings { get; set; } = new();
}
```

### Named Service Resolution
```csharp
services.AddNamedSingleton<TService, TImplementation>("name");
var service = serviceProvider.GetRequiredNamedService<TService>("name");
```

### Extension Method Conventions
- HostBuilderExtensions.cs for IHostBuilder extensions
- ServiceCollectionExtensions.cs for IServiceCollection extensions
- Always return the builder for chaining
- Use TryAdd* for services to allow override

## Detailed References

Read the reference file matching the task at hand:

- [references/01-project-setup-critical.md](references/01-project-setup-critical.md) - Read when creating a new project or configuring Uno.Sdk, UnoFeatures, or GlobalUsings
- [references/02-build-configuration-critical.md](references/02-build-configuration-critical.md) - Read when setting up Directory.Build.props, package management, or TFMs
- [references/03-mvvm-mvux-patterns-high.md](references/03-mvvm-mvux-patterns-high.md) - Read when implementing MVVM/MVUX, data binding, or dependency injection
- [references/04-navigation-high.md](references/04-navigation-high.md) - Read when setting up navigation, Shell pattern, ViewMaps, or RouteMaps
- [references/05-styling-theming-medium.md](references/05-styling-theming-medium.md) - Read when theming, styling, choosing typography, or building responsive layouts
- [references/06-platform-specific-high.md](references/06-platform-specific-high.md) - Read when writing conditional compilation, platform folders, or native services
- [references/07-data-http-medium.md](references/07-data-http-medium.md) - Read when configuring HTTP clients, Refit/Kiota, LiteDB, or appsettings
- [references/08-custom-controls-medium.md](references/08-custom-controls-medium.md) - Read when building SkiaSharp rendering, ItemsControl, Lottie, or image handling
