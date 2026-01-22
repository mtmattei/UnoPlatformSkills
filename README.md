# UnoSkills

Claude Code skills for Uno Platform development and WinUI/XAML best practices.

## Overview

This repository contains specialized skills that can be used with Claude Code to assist with Uno Platform and WinUI XAML development. These skills provide context-aware guidance on performance optimization, best practices, and common patterns.

## Skills

### WinUI XAML Best Practices (`12-WinUI-XAML-best-practices.md`)

A comprehensive guide covering WinUI XAML performance optimization across 16 critical areas:

| Section | Impact | Description |
|---------|--------|-------------|
| Async Operations | CRITICAL | Eliminating UI thread blocking |
| XAML Loading | CRITICAL | Optimizing XAML parsing and compilation |
| Data Binding | HIGH | Efficient binding patterns with x:Bind |
| Collection Virtualization | CRITICAL | Large list performance |
| Layout Performance | HIGH | Minimizing layout passes |
| Rendering & Composition | MEDIUM-HIGH | GPU-accelerated animations |
| Memory Management | HIGH | Preventing leaks and managing allocations |
| Navigation | MEDIUM-HIGH | Fast page transitions |
| Threading | HIGH | Proper background work patterns |
| Controls & Templating | MEDIUM | Efficient control design |
| Graphics & Media | MEDIUM-HIGH | Image and video optimization |
| Text & Typography | LOW-MEDIUM | Text rendering efficiency |
| Input & Interaction | MEDIUM | Responsive input handling |
| Resource Management | MEDIUM | Efficient resource dictionaries |
| Application Lifecycle | MEDIUM | Suspend/resume handling |
| Platform Integration | LOW-MEDIUM | Windows API interop |

## Usage

These skills are designed to be loaded as context for Claude Code when working on Uno Platform projects. They provide:

- Correct vs incorrect code examples
- Performance measurements and benchmarks
- Links to official Microsoft documentation
- Practical implementation patterns

## Related

- [Uno Platform Documentation](https://platform.uno/docs/articles/intro.html)
- [WinUI 3 Documentation](https://docs.microsoft.com/en-us/windows/apps/winui/winui3/)
- [Claude Code](https://claude.ai/claude-code)

## License

MIT
