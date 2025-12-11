# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`htop` is a cross-platform interactive process viewer written in C. This is a personal fork with custom modifications including vim-like keybindings (gg/G for top/bottom navigation) and actionToggleFollow functionality.

## Build Commands

### Standard Build Process
```bash
# First time setup (after git clone or when configure.ac changes)
./autogen.sh && ./configure && make

# Regular build after changes
make

# Clean build
make clean
./configure
make

# Install (default to /usr/local)
make install

# Install to custom location
./configure --prefix=/opt/homebrew
make
sudo make install
```

### Quick Build Script
The `./build` script in the repository root provides a convenient build-and-install workflow for macOS (installs to /opt/homebrew).

### Build Options
Configure supports various platform-specific options:
- `--enable-unicode`: Unicode support (default: yes, requires libncursesw)
- `--enable-debug`: Enable assertions and sanity checks
- `--enable-sensors`: Linux temperature data support
- `--enable-hwloc`: CPU affinity via hwloc
- See `./configure --help` for full list

### Dependencies
- C99 compliant compiler
- autoconf, automake, autotools
- ncurses (ncursesw for Unicode support)
- Platform-specific: pkg-config, libsensors, hwloc, libcap, libnl-3

## Code Architecture

### Platform Abstraction Layer
htop uses a platform-specific architecture to support multiple operating systems:
- Platform directories: `linux/`, `darwin/`, `freebsd/`, `openbsd/`, `netbsd/`, `solaris/`, `dragonflybsd/`
- Each platform provides its own implementation of:
  - `Platform.c/h`: Platform-specific initialization and operations
  - `DarwinMachine.c/h` (or LinuxMachine, etc.): Machine-level state management
  - `DarwinProcess.c/h`: Process-specific data structures and methods
  - `DarwinProcessTable.c/h`: Process table scanning and updates
  - `ProcessField.h`: Platform-specific process fields/columns

The build system automatically selects the correct platform directory based on `host_os` detection in configure.ac.

### Core Object Model
htop uses a pseudo-OOP approach in C:
- **Object.h/c**: Base "class" providing reference counting and virtual method dispatch
- **Row.h/c**: Base type for displayable items (processes, table entries)
- **Process.h/c**: Extends Row, represents a single process with common cross-platform fields
- **Panel.h/c**: UI container for displaying lists of Objects with event handling
- **Machine.h/c**: System-wide state (memory, CPUs, process tables, timing)

Key abstractions:
- Classes defined via structs with function pointers (e.g., `PanelClass`, `ObjectClass`)
- Macro-based method dispatch (e.g., `Panel_eventHandler()`, `Object_display()`)
- Platform code extends base types (e.g., `LinuxProcess` extends `Process`)

### UI and Display System
- **CRT.c/h**: Terminal/curses initialization, color schemes, signal handling
- **ScreenManager.c/h**: Multi-screen management and navigation
- **Header.c/h**: Top area showing meters (CPU, memory, etc.)
- **MainPanel.c/h**: Primary process list panel
- **Meter.c/h**: Base for various meters (CPU, Memory, Battery, etc.)
- **RichString.c/h**: Attributed string rendering for colored text
- **FunctionBar.c/h**: Bottom function key hints (F1-F10)

### Action System
- **Action.c/h**: Defines `Htop_Reaction` flags and action handlers
- **State**: Central state struct passed to actions (holds Machine, MainPanel, Header)
- Actions return reaction flags: `HTOP_REFRESH`, `HTOP_RECALCULATE`, `HTOP_QUIT`, etc.
- Key bindings mapped to actions via `Action_setBindings()`

### Settings and Configuration
- **Settings.c/h**: Manages configuration file (~/.config/htop/htoprc)
- Handles: columns, meters, colors, sort order, tree view, display options
- **ColumnsPanel.c/h**, **MetersPanel.c/h**: UI for configuring columns and meters

### Data Structures
- **Vector.c/h**: Dynamic array implementation
- **Hashtable.c/h**: Hash table for process tracking
- **UsersTable.c/h**: UID to username mapping cache
- **Table.c/h**: Generic table abstraction for different data views

### Specialized Screens
- **InfoScreen.c/h**: Base class for auxiliary info screens
- **TraceScreen.c/h**: System call tracing (strace-like)
- **OpenFilesScreen.c/h**: Open files for a process (lsof-like)
- **EnvScreen.c/h**: Process environment variables
- **CommandScreen.c/h**: Display full command line

## File Organization

### Naming Conventions
- Source files: CamelCase with uppercase first letter (e.g., `ProcessTable.c`)
- Exceptions: `htop.c`, `pcp-htop.c` (main entry points)
- Platform directories: lowercase (e.g., `linux/`, `darwin/`)
- Function names: `Module_functionName` pattern (e.g., `Process_makeCommandStr`)
- Static functions: Should be marked `static` unless part of public module API

### Header Structure
All headers use include guards placed **before** the copyright:
```c
#ifndef HEADER_ModuleName
#define HEADER_ModuleName
/*
htop - ModuleName.h
(C) YEAR htop dev team
Released under the GNU GPLv2+, see the COPYING file
in the source distribution for its full text.
*/
```

### Include Order
1. `#include "config.h" // IWYU pragma: keep` (if needed for autoconf or feature guards)
2. Module's own header (for .c files only)
3. System headers (non-conditional)
4. Project headers
5. Conditional includes (system first, then project)

Sort alphabetically within each group, with subdirectories after parent (e.g., `unistd.h` before `sys/time.h`).

When using `XUtils.h`, the .c file MUST include `config.h` first or compilation will fail.

## Code Style

### Formatting
- **Indentation**: 3 spaces (never tabs)
- **Braces**: Opening brace on same line, closing brace on new line
- **Spacing**: Space after keywords (e.g., `if (condition)`, `while (x)`)
- **Line length**: Break long lines with continuation indented one more level
- **Single statements**: Braces optional for simple statements but required if any branch of an if/else uses braces

Example:
```c
if (condition)
   doSomething();
else if (other_condition &&
         another_long_check) {
   doComplexThing();
   andAnotherThing();
} else {
   fallback();
}
```

### Auto-formatting
```bash
astyle -r -xb -s3 -p -xg -c -k1 -W1 -H *.c *.h
```

### Memory Management
- Use `xMalloc()`, `xCalloc()`, `xRealloc()` instead of raw malloc/calloc/realloc
  - These wrappers check for allocation failure and exit on error
  - Never allocate 0 bytes; use `NULL` explicitly instead
- For arrays, use `xReallocArray()` and `xReallocArrayZero()`
- Always free resources properly

### String Utilities
Prefer `String_*` functions from `XUtils.h`:
- `String_eq(a, b)` instead of `!strcmp(a, b)`
- `String_startsWith(str, prefix)` instead of `strncmp(str, prefix, len) == 0`

### Symbol Exports
- Mark functions `static` unless they're part of the module's public API
- Public functions must have declarations in the module's .h file
- Avoid function-like macros; prefer inline functions

## Platform Development

When modifying or adding platform-specific code:

1. **Understand the Platform Interface**: Each platform must implement the interface defined by the base types (Machine, Process, ProcessTable, Platform)

2. **Platform Files**: Located in platform directories (e.g., `darwin/Platform.c`)
   - Platform_init(): Initialize platform-specific subsystems
   - Platform_done(): Cleanup
   - Platform_setBindings(): Platform-specific key bindings
   - Platform_getUptime(), Platform_getMaxPid(), etc.

3. **Process Scanning**: Platform's ProcessTable scans /proc, kvm, sysctl, or other OS-specific APIs to populate process data

4. **Platform Fields**: Define in platform's `ProcessField.h` using the `PLATFORM_PROCESS_FIELDS` macro

5. **Compatibility**: Provide fallbacks for features unavailable on older systems; gracefully degrade if necessary

## Recent Modifications (This Fork)

- **Vim-like navigation**: Added `gg` (jump to top) and `G` (jump to bottom) keybindings
- **Follow toggle**: Implemented `actionToggleFollow` function and keymap
- Custom keybindings defined in `Action.c:710` and related help menu updates

## Contributing Guidelines

- Follow the [style guide](docs/styleguide.md)
- Keep commits minimal and self-contained
- Write descriptive commit messages explaining the "why"
- Rebase instead of merge when updating branches
- Use `git commit --fixup` and `git rebase -i --autosquash` to squash fixups
- Acknowledge AI tool usage with `Co-authored-by:` clauses if applicable
- Review and understand all code before proposing (especially AI-generated)

## Compatibility and Standards

- Code should compile with any C99-compliant compiler
- Support older systems when possible; provide fallbacks for modern APIs
- Avoid breaking changes to configuration file format
- Test on multiple platforms when changing core code
