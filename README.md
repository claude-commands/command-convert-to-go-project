# command-convert-to-go-project

Convert an existing project to the Go + Templ + HTMX + Tailwind stack.

## Installation

```bash
git clone git@github.com:corylanou/command-convert-to-go-project.git ~/projects/command-convert-to-go-project
ln -s ~/projects/command-convert-to-go-project/convert-to-go-project.md ~/.claude/commands/convert-to-go-project.md
```

## Usage

```text
/convert-to-go-project [target-directory]
```

**Examples:**

```text
/convert-to-go-project           # Convert current directory
/convert-to-go-project ./myapp   # Convert specific directory
```

## What It Does

1. Analyzes your existing project structure
2. Detects current technologies (Node.js, Python, PHP, etc.)
3. Asks about conversion scope (full, incremental, backend-only)
4. Asks database and service preferences
5. Creates Go project structure alongside existing files
6. Provides framework-specific migration guides
7. Migrates routes, templates, and database schema

## Supported Source Frameworks

| Framework | Migration Support |
|-----------|-------------------|
| Express.js | Routes, middleware, controllers |
| Next.js | API routes, React to HTMX |
| Django | URLs, views, templates, models |
| Flask | Routes, templates |
| Laravel | Routes, Blade templates, Eloquent |
| Ruby on Rails | Routes, ERB templates |

## Conversion Types

- **Full conversion**: Replace entire stack with Go
- **Incremental**: Add Go alongside existing code for gradual migration
- **Backend only**: Convert API, keep existing frontend (React, Vue, etc.)

## Technology Stack (Target)

| Component | Technology |
|-----------|------------|
| Web Framework | Echo v4 |
| Templating | Templ |
| CSS | Tailwind CSS v4 |
| Interactivity | HTMX |
| Database | sqlc + goose |
| Hot Reload | Air |

## Requirements

- Go 1.23+
- Node.js (for Tailwind CSS)
- direnv (recommended)

## Updates

```bash
cd ~/projects/command-convert-to-go-project && git pull
```
