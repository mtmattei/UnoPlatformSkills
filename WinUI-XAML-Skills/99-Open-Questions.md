# Open Questions - WinUI 3 XAML Skills

This document lists areas that could not be conclusively validated or require further investigation.

---

## Performance Claims

### 1. Exact Performance Metrics
**Question**: The original document included specific performance numbers (e.g., "3.75x faster", "50-100ms per image"). Are these accurate for current WinUI 3 and Uno Platform versions?

**Status**: Unverified

**Recommendation**: These numbers should be considered illustrative rather than guaranteed. Actual performance varies by:
- Device hardware
- Uno Platform target (Skia vs native)
- App complexity
- .NET version

**Action needed**: Run benchmarks on target platforms to establish baseline metrics for your specific scenarios.

---

### 2. x:Load Memory Overhead
**Question**: The Microsoft documentation states x:Load adds "approximately 600 bytes" per deferred element. Is this accurate for Uno Platform?

**Status**: Unverified for Uno

**Recommendation**: The 600-byte figure is from Microsoft's WinUI documentation. Uno Platform's implementation may differ. Measure actual memory impact in your app.

---

## API Behavior

### 3. ContainerContentChanging Consistency
**Question**: Does `ContainerContentChanging` fire consistently across all Uno Platform targets (Skia, iOS, Android, WASM)?

**Status**: Partially verified

**Notes**:
- Works on Skia backends
- Native renderer behavior may differ
- `x:Phase` is the declarative alternative that may be more consistent

**Action needed**: Test phased loading behavior on each target platform if using this pattern.

---

### 4. ConnectedAnimation Support
**Question**: What is the full support matrix for `ConnectedAnimation` across Uno Platform targets?

**Status**: Unclear

**Notes**:
- Documentation indicates "support varies by platform"
- Desktop (Skia) likely has best support
- Mobile support unclear

**Action needed**: Test connected animations on each target platform. Have fallback for unsupported platforms.

---

### 5. DispatcherQueueTimer vs DispatcherTimer
**Question**: The examples use `DispatcherQueueTimer`. Is this the correct choice for Uno Platform, or should `DispatcherTimer` be used in some cases?

**Status**: Clarification needed

**Notes**:
- WinUI 3 uses `DispatcherQueue` and `DispatcherQueueTimer`
- Uno Platform supports both
- `DispatcherQueueTimer` is the WinUI 3 pattern

**Recommendation**: Use `DispatcherQueueTimer` for consistency with WinUI 3. If issues arise, fall back to `DispatcherTimer`.

---

## Missing Documentation

### 6. Object Pooling in WinUI 3
**Question**: Are there official WinUI 3 guidelines or APIs for object pooling in UI scenarios?

**Status**: No official guidance found

**Notes**:
- `Microsoft.Extensions.ObjectPool` is used in examples
- No WinUI-specific pooling APIs exist
- Pattern is adapted from ASP.NET Core

**Recommendation**: Use `Microsoft.Extensions.ObjectPool` or implement custom pooling based on scenario requirements.

---

### 7. Optimal Merged Dictionary Count
**Question**: What is the recommended maximum number of merged ResourceDictionaries?

**Status**: No specific guidance

**Notes**:
- General advice is "fewer is better"
- No Microsoft documentation specifies a number
- Impact depends on dictionary size and complexity

**Recommendation**: Consolidate related styles into single files. Aim for <10 merged dictionaries at app level. Measure startup time impact.

---

## Platform Gaps

### 8. High Contrast Mode on Non-Windows
**Question**: How should apps handle high contrast mode on non-Windows platforms (macOS, iOS, Android, Web)?

**Status**: Unclear

**Notes**:
- Windows has explicit high contrast themes
- Other platforms have accessibility settings but different mechanisms
- Uno Platform's `HighContrast` theme dictionary may not apply

**Recommendation**:
- Use `ThemeResource` everywhere to enable theme adaptation
- Test with platform-specific accessibility settings
- Consider adding explicit "high contrast" mode toggle for non-Windows platforms

---

### 9. Keyboard Accelerators on Mobile
**Question**: Should keyboard accelerators be defined if the app targets mobile platforms where they won't be used?

**Status**: Design decision

**Notes**:
- Accelerators won't harm mobile builds
- They won't be displayed or functional
- Adds minimal overhead

**Recommendation**: Define accelerators for desktop users. They have no negative impact on mobile builds.

---

### 10. ISupportIncrementalLoading on ItemsRepeater
**Question**: Does ItemsRepeater automatically detect and use `ISupportIncrementalLoading`?

**Status**: No - requires explicit setup

**Notes**:
- ListView/GridView automatically use `ISupportIncrementalLoading`
- ItemsRepeater requires Uno Toolkit extensions

**Verified Solution**:
```xml
<ItemsRepeater ItemsSource="{x:Bind Items}"
               utu:ItemsRepeaterExtensions.SupportsIncrementalLoading="True"/>
```

---

## Recommendations for Resolution

1. **Performance claims**: Establish project-specific benchmarks rather than relying on documented numbers.

2. **Platform-specific behavior**: Create test pages/samples that exercise key patterns on each target platform.

3. **API gaps**: Check Uno Platform GitHub issues and discussions for known limitations.

4. **Feature requests**: If critical functionality is missing, consider filing issues at https://github.com/unoplatform/uno

---

## How to Contribute

If you have answers to any of these questions or discover additional gaps:

1. Verify the behavior with a reproducible test case
2. Update the relevant skills document with the verified information
3. Add a note to the Validation Notes document
4. Remove the resolved question from this document

---

## Last Updated
2026-01-22
