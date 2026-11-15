# Changelog

All notable changes to PIOKMBox will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Bridge firmware: keyboard injection over the CDC text protocol - `km.press`, `km.type`, `km.release`, and `km.keys(mods[,k1..k5])` (sets the complete keyboard state, so the host controls hold times); `km.human(mode)` to switch the KMBox mouse humanization off / micro / full
- Bridge firmware: host-loss failsafe - releases all keys and mouse buttons when the CDC host closes the port
- Initial release with dual USB stack support
- KMBox serial command compatibility
- Support for RP2040 (Pico) and RP2350 (Pico 2)
- Hardware watchdog system
- Visual status indicators (LED + NeoPixel)
- Automated GitHub Actions build system

### Changed

### Deprecated

### Removed

### Fixed

- Bridge firmware: back-to-back text commands in one CDC read could be dropped while the UART DMA was busy; an uppercase `P` inside a `km.*` line was mistaken for a Makcu frame start; bytes queued before DTR dropped were never read

### Security

## Template for Future Releases

```markdown
## [X.Y.Z] - YYYY-MM-DD

### Added
- New features

### Changed
- Changes in existing functionality

### Deprecated
- Soon-to-be removed features

### Removed
- Now removed features

### Fixed
- Bug fixes

### Security
- Vulnerability fixes
```

## Release Guidelines

- **Added** for new features
- **Changed** for changes in existing functionality
- **Deprecated** for soon-to-be removed features
- **Removed** for now removed features
- **Fixed** for any bug fixes
- **Security** in case of vulnerabilities
