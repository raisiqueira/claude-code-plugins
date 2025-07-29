# CLAUDE.md

This file provides focused development guidance for Claude Code when working with this repository.

# <project name> Development

A brief project description. E.g, Project X is a tool for Y that helps users achieve Z.

> **See `docs/` for complete architecture, installation, and usage documentation**

## Essential Development Commands

Always use `PNPM` for package management.

```bash
pnpm run dev        # To start the development server.
pnpm run test       # Run tests.
pnpm run check      # Run type checks.
pnpm run lint       # Run linters.
pnpm run fmt        # Format code.
pnpm run build      # Build the project.
```

## Key Project Structure

This is an example of how you can describe the project structure:

**Core Files:**
- `src/`: Contains the main source code.
- `tests/`: Contains unit and integration tests.
- `docs/`: Contains documentation files.

**Important Directories:**
- `src/components/`: Reusable components.
- `src/utils/`: Utility functions.
- `src/services/`: Services for API calls and business logic.

## Development Standards

**Code Style:**
- Use `rg` instead of `grep` (https://github.com/BurntSushi/ripgrep)
- Use `fd` instead of `find` (https://github.com/sharkdp/fd)

## Critical Guidelines

**IMPORTANT: Before making changes:**
1. Read existing code patterns in the affected area
2. Use the same coding style and conventions
3. Test with `pnpm run dev` before committing
4. All tests must pass - no exceptions

**YOU MUST:**
- Follow the project architecture (see the `docs/` directory)
- Handle errors gracefully with meaningful messages

**DO NOT:**
- Break the build system or single-file distribution
- Skip linting or testing steps
- Introduce dependencies not already in the project, UNLESS absolutely necessary and only with prior approval
- Change core architecture without understanding impact