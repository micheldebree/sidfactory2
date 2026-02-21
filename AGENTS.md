# SID Factory II - Agent Guidelines

This document provides essential information for AI agents working in the SID
Factory II repository.

## Project Overview

SID Factory II is a cross-platform C++/SDL2 editor for composing Commodore 64
music using the reSID emulator. It supports Windows, macOS, and Linux.

## Build & Development Commands

### Compilation

- **macOS (Primary):** `cd macos && make raw` (creates a DMG in `artifacts/`)
- **macOS (Universal):** `cd macos && make universal`
- **Linux:** `make dist`
- **Windows:** `build_windows.bat`
- **Local Dev (macOS/Linux):** `make` (builds to `artifacts/SIDFactoryII`)

### Execution & Debugging

- **Run:** `make run` (runs the compiled binary with a default tune)
- **Debug:** `make debug` (runs under `lldb`)
- **Clean:** `make clean`

### Testing

There is no comprehensive unit test suite for the main application.

- Library-specific tests (e.g., `miniz_tester.cpp`) exist but are generally
  excluded from main builds.
- Use `make run` or manual execution for verification.

## Code Style & Formatting

The project uses `.clang-format` to enforce style.

### General Rules

- **Indentation:** Always use **Tabs** (width 4).
- **Braces:** Always on a **new line** for classes, functions, and control statements.
- **Pointers:** Left-aligned (e.g., `Type* variableName`).
- **Headers:** Use `#pragma once` for header guards.
- **Namespaces:** Use namespaces to organize code (e.g., `Foundation`,
  `Emulation`, `Editor`).
- **Standard Library:** Limited usage of heavy STL components; prefers simple
  types or manual memory management for performance-critical code.
  `std::shared_ptr` is used for resource management in some areas.

## Naming Conventions

The project follows a strict PascalCase convention.

- **Classes / Structs:** `PascalCase` (e.g., `CPUMemory`, `Point`).
- **Interfaces:** `IPascalCase` (e.g., `IPlatform`, `IMemoryRandomReadAccess`).
- **Methods / Functions:** `PascalCase` (e.g., `GetByte()`, `SetData()`).
- **Member Variables:** `m_PascalCase` (e.g., `m_Memory`, `m_nSize`).
- **Parameters:** `inPascalCase` (e.g., `inAddress`, `inSize`). Sometimes
  prefixes like `n`, `uc`, or `p` are used for primitives/pointers (e.g.,
  `nAddress`, `ucByte`, `pSource`).
- **Namespaces:** `PascalCase`.

## Project Structure & Architecture

- `SIDFactoryII/source/`: Core source code.
  - `foundation/`: Low-level base abstractions.
    - `base/`: Fundamental types (`Point`, `Rect`, `Extent`), assertions, and
      optional types.
    - `graphics/`: Rendering abstractions.
    - `sound/`: Audio output and mixing.
    - `platform/`: Mutexes, file I/O, and platform-specific interfaces.
  - `runtime/`: Application-specific logic.
    - `emulation/`: C64 emulation core (CPU, SID, Memory).
    - `execution/`: Execution loop and timing.
    - `editor/`: UI components, data sources, and command handling.
  - `utils/`: Common helpers like string manipulation or math.
- `macos/`: macOS-specific build files, Info.plist, and app bundle resources.
- `libs/`: External pre-compiled libraries (SDL2, etc.).
- `resources/`: Icons and static assets.
- `drivers/`: C64 music drivers (binaries).

## Coding Patterns

### Memory Management

- **Manual Management:** Often used in performance-critical sections (e.g.,
  emulation buffers). Use `new[]` and `delete[]` in constructors/destructors.
- **Smart Pointers:** `std::shared_ptr` is used for shared resources like
  mutexes or platform interfaces.
- **RAII:** Encapsulate resources in classes.

### Abstraction & Interfaces

- The project heavily uses interfaces (prefixed with `I`) to decouple modules.
- Example: `IMemoryRandomReadAccess` provides a read-only view of memory,
  implemented by `CPUMemory`.
- Use `final` for leaf classes to help with compiler optimizations.

### Performance Considerations

- Avoid heavy STL containers in the inner emulation loops.
- Use fixed-size buffers where possible.
- Prefer passing by `const Reference&` to avoid copies of non-primitive types.

## Common Includes & Macros

- Always include `foundation/base/assert.h` for `FOUNDATION_ASSERT`.
- Include `foundation/base/types.h` for `Point`, `Rect`, etc.
- Emulation constants are in `runtime/environmentdefines.h`.
- Use `SDL_Log` (via SDL2) for logging if needed.

## Error Handling & Practices

- **Assertions:** Use `FOUNDATION_ASSERT(expression)` (aliased to `SDL_assert`)
  for internal logic checks and programmer errors.
- **Memory Management:** The codebase uses both manual `new`/`delete`
  (especially in performance-sensitive emulation code) and `std::shared_ptr`.
  Always ensure proper cleanup in destructors.
- **Initialization:** Prefer member initializer lists in constructors.
- **Abstraction:** Use interfaces (abstract classes with `I` prefix) to
  decouple platform-specific logic from core emulation.

## Environment Defines

Key emulation constants are defined in
`SIDFactoryII/source/runtime/environmentdefines.h` (e.g.,
`EMULATION_CYCLES_PER_SECOND_PAL`).

## Documentation

- `README.md`: General info and changelog.
- `DEVELOPMENT.md`: Build instructions and release process.

## AI Instructions

- Respect the existing PascalCase naming and tab-based indentation.
- Always include `foundation/base/assert.h` if using assertions.
- When adding new files, follow the namespace-per-directory pattern.
- Be cautious with STL; if a pattern for a container doesn't exist in a file,
  check if a simpler approach is preferred.
