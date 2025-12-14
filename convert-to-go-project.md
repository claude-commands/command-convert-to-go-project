---
argument-hint: "[target-directory]"
description: "Convert an existing project to the Go + Templ + HTMX + Tailwind stack"
model: claude-opus-4-5-20251101
allowed-tools: ["Bash", "Read", "Write", "Edit", "Glob", "Grep", "AskUserQuestion"]
---

# Convert to Go Project

**If `$ARGUMENTS` is empty or not provided:**

This command converts an existing project to the Go + Templ + HTMX + Tailwind stack.

It will analyze your current project, identify what can be preserved, and incrementally
add Go stack files while migrating your existing logic.

**Usage:** `/convert-to-go-project [target-directory]`

**Examples:**

- `/convert-to-go-project` - Convert current directory
- `/convert-to-go-project ./my-project` - Convert specific directory

**Workflow:**

1. Analyze existing project structure and technologies
2. Ask about conversion scope (full vs. incremental)
3. Ask database and service preferences
4. Create Go project structure alongside existing files
5. Migrate routes, handlers, and templates
6. Provide migration guidance for remaining code

Proceed to analyze the current directory.

---

**If `$ARGUMENTS` is provided:**

Convert the project at `$ARGUMENTS` (or current directory if `.`) to the Go stack.

## Step 1: Project Analysis

Scan the target directory to detect existing technologies.

### Detection Checks

Run these checks to identify the current stack:

| File/Pattern | Technology | Detection Command |
|--------------|------------|-------------------|
| `package.json` | Node.js | Check for express, fastify, next, etc. |
| `requirements.txt` or `pyproject.toml` | Python | Check for django, flask, fastapi |
| `composer.json` | PHP | Check for laravel, symfony |
| `go.mod` | Go (existing) | Already Go - extend rather than convert |
| `Gemfile` | Ruby | Check for rails, sinatra |
| `*.sql` or `schema.prisma` | Database schema | Existing database structure |
| `Dockerfile` | Docker | Container configuration |
| `.env` or `.envrc` | Environment | Existing configuration |

### Present Analysis Results

After scanning, present findings to the user:

```text
## Project Analysis Results

**Detected Technologies:**
- [List detected frameworks/languages]

**Database:**
- [Detected database type or "None detected"]

**Existing Files to Preserve:**
- README.md
- .git/
- [Other important files]

**Files to Migrate:**
- [Routes/controllers]
- [Templates/views]
- [Database models]

**Files to Replace:**
- [Framework-specific config]
```

Ask the user to confirm the analysis is correct.

---

## Step 2: Gather Conversion Requirements

Use AskUserQuestion for each of these:

### Conversion Scope

| Option | Description |
|--------|-------------|
| **Full conversion** | Replace entire stack with Go |
| **Incremental** | Add Go alongside existing code |
| **Backend only** | Convert backend, keep frontend as-is |

**Plain explanations:**

- **Full conversion**: Best for smaller projects. We'll convert everything at once -
  routes, templates, database queries. Old framework files will be removed.

- **Incremental**: Best for larger projects. We'll add Go files alongside your existing
  code. You can run both during transition and migrate piece by piece.

- **Backend only**: Keep your frontend (React, Vue, etc.) and just convert the API
  to Go. Good if you have a separate frontend app.

### Database Selection

Same options as `/create-go-project`:

| Database | Best For |
|----------|----------|
| **PostgreSQL** | Production apps, complex queries |
| **SQLite** | Prototypes, single-user apps |
| **MySQL** | Existing MySQL infrastructure |

### Admin Dashboard

| Option | Description |
|--------|-------------|
| **Yes** | Include admin UI with dark/light mode |
| **No** | Skip admin UI |

### Deployment Platform

| Platform | Best For |
|----------|----------|
| **Vercel** | Free hosting, easy setup |
| **Railway** | Traditional servers with databases |
| **Fly.io** | Global edge deployment |
| **Self-hosted** | Full control |

---

## Step 3: Migration Strategy

Based on the detected framework, provide specific migration guidance.

### Express.js to Echo

**Route Mapping:**

```javascript
// Express (before)
router.get('/users', usersController.list);
router.post('/users', usersController.create);
router.get('/users/:id', usersController.show);
router.put('/users/:id', usersController.update);
router.delete('/users/:id', usersController.delete);
```

```go
// Echo (after)
e.GET("/users", h.UserList)
e.POST("/users", h.UserCreate)
e.GET("/users/:id", h.UserShow)
e.PUT("/users/:id", h.UserUpdate)
e.DELETE("/users/:id", h.UserDelete)
```

**Middleware Conversion:**

```javascript
// Express middleware
app.use(cors());
app.use(express.json());
app.use(morgan('dev'));
```

```go
// Echo middleware
e.Use(middleware.CORS())
e.Use(middleware.Logger())
e.Use(middleware.Recover())
```

**Controller to Handler:**

```javascript
// Express controller
exports.list = async (req, res) => {
  const users = await User.findAll();
  res.json(users);
};
```

```go
// Go handler
func (h *Handler) UserList(c echo.Context) error {
    ctx := c.Request().Context()
    users, err := h.db.Queries.ListUsers(ctx)
    if err != nil {
        return err
    }
    return c.JSON(http.StatusOK, users)
}
```

### Django/Flask to Go

**URL Patterns:**

```python
# Django
urlpatterns = [
    path('users/', views.user_list, name='user_list'),
    path('users/<int:pk>/', views.user_detail, name='user_detail'),
]

# Flask
@app.route('/users')
def user_list():
    ...
```

```go
// Echo
e.GET("/users", h.UserList)
e.GET("/users/:id", h.UserDetail)
```

**Django Template to Templ:**

```django
{% extends "base.html" %}
{% block content %}
  <h1>{{ user.name }}</h1>
  {% for post in posts %}
    <article>{{ post.title }}</article>
  {% endfor %}
{% endblock %}
```

```templ
package pages

import "myapp/templates/layouts"

templ UserDetail(user User, posts []Post) {
    @layouts.Base("User") {
        <h1>{ user.Name }</h1>
        for _, post := range posts {
            <article>{ post.Title }</article>
        }
    }
}
```

**Django Model to goose Migration:**

```python
# Django model
class User(models.Model):
    email = models.EmailField(unique=True)
    name = models.CharField(max_length=100)
    created_at = models.DateTimeField(auto_now_add=True)
```

```sql
-- goose migration
-- +goose Up
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(100),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- +goose Down
DROP TABLE IF EXISTS users;
```

### Laravel to Go

**Route Mapping:**

```php
// Laravel
Route::get('/users', [UserController::class, 'index']);
Route::post('/users', [UserController::class, 'store']);
Route::get('/users/{user}', [UserController::class, 'show']);
```

```go
// Echo
e.GET("/users", h.UserIndex)
e.POST("/users", h.UserStore)
e.GET("/users/:id", h.UserShow)
```

**Blade to Templ:**

```blade
@extends('layouts.app')

@section('content')
    <h1>{{ $user->name }}</h1>
    @foreach($posts as $post)
        <article>{{ $post->title }}</article>
    @endforeach
@endsection
```

```templ
package pages

import "myapp/templates/layouts"

templ UserShow(user User, posts []Post) {
    @layouts.Base("User") {
        <h1>{ user.Name }</h1>
        for _, post := range posts {
            <article>{ post.Title }</article>
        }
    }
}
```

**Eloquent to sqlc:**

```php
// Laravel Eloquent
$users = User::where('active', true)->orderBy('name')->get();
```

```sql
-- sqlc query
-- name: ListActiveUsers :many
SELECT * FROM users WHERE active = true ORDER BY name;
```

### Next.js to Go + HTMX

**API Routes:**

```javascript
// Next.js API route (pages/api/users.js)
export default async function handler(req, res) {
  if (req.method === 'GET') {
    const users = await prisma.user.findMany();
    res.json(users);
  }
}
```

```go
// Go handler
func (h *Handler) UserList(c echo.Context) error {
    users, err := h.db.Queries.ListUsers(c.Request().Context())
    if err != nil {
        return err
    }
    return c.JSON(http.StatusOK, users)
}
```

**React Component to Templ + HTMX:**

```jsx
// React with fetch
function UserList() {
  const [users, setUsers] = useState([]);
  useEffect(() => {
    fetch('/api/users').then(r => r.json()).then(setUsers);
  }, []);
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

```templ
// Templ with HTMX
templ UserList(users []User) {
    <ul hx-get="/users" hx-trigger="load" hx-swap="innerHTML">
        for _, user := range users {
            <li>{ user.Name }</li>
        }
    </ul>
}
```

---

## Step 4: Create Go Project Structure

Add Go files to the project. Create the same structure as `/create-go-project`:

```text
[existing-project]/
├── cmd/server/           # NEW
├── internal/             # NEW
├── templates/            # NEW (migrate existing views)
├── migrations/           # NEW (from existing schema)
├── sqlc/                 # NEW
├── static/               # KEEP or merge with existing
├── .air.toml             # NEW
├── Makefile              # NEW
├── go.mod                # NEW
├── [existing files]      # PRESERVE during transition
```

### File Creation Order

1. **Foundation files:** go.mod, .gitignore additions, .air.toml, Makefile
2. **Configuration:** internal/config/config.go, sqlc/sqlc.yaml
3. **Database:** internal/database/database.go, migrations (from existing schema)
4. **SEO/Meta:** internal/meta/meta.go, templates/layouts/meta.templ
5. **Core Go:** internal/middleware, internal/handler
6. **Templates:** Convert existing views to Templ (including meta integration)
7. **Entry point:** cmd/server/main.go, cmd/server/slog.go

### SEO and Meta Tag Migration

**Key principle:** In Go, handlers do NOT construct meta. Templates own their meta.

#### Create Files for SEO

1. `internal/ctxkeys/keys.go` - Typed context keys
2. `internal/meta/meta.go` - PageMeta struct
3. `internal/meta/context.go` - Context helpers (SiteNameFromCtx)
4. `templates/layouts/meta.templ` - Meta tag component

#### internal/ctxkeys/keys.go

```go
package ctxkeys

type siteConfigKey struct{}

var SiteConfig = siteConfigKey{}
```

#### internal/meta/meta.go

```go
package meta

type PageMeta struct {
    Title       string
    Description string
    OGType      string
    OGImage     string
    Canonical   string
    NoIndex     bool
}

func New(title, description string) PageMeta {
    return PageMeta{
        Title:       title,
        Description: description,
        OGType:      "website",
    }
}

func (m PageMeta) WithOGImage(url string) PageMeta {
    m.OGImage = url
    return m
}

func (m PageMeta) AsArticle() PageMeta {
    m.OGType = "article"
    return m
}

func (m PageMeta) AsProduct() PageMeta {
    m.OGType = "product"
    return m
}
```

#### internal/meta/context.go

```go
package meta

import (
    "context"
    "yourapp/internal/config"
    "yourapp/internal/ctxkeys"
)

func SiteFromCtx(ctx context.Context) config.SiteConfig {
    if cfg, ok := ctx.Value(ctxkeys.SiteConfig).(config.SiteConfig); ok {
        return cfg
    }
    return config.SiteConfig{Name: "MyApp"}
}

func SiteNameFromCtx(ctx context.Context) string {
    return SiteFromCtx(ctx).Name
}
```

#### Framework-Specific Migration

**From Next.js (handler passes meta):**

```jsx
// Next.js (before)
export const metadata = {
  title: 'My Page',
  description: 'Page description',
};
```

```templ
// Go template (after) - template constructs meta
templ MyPage() {
    @layouts.Base(meta.New("My Page", "Page description")) {
        // content
    }
}
```

**From Django (template blocks):**

```django
{% raw %}
{% block meta %}
<title>{{ page_title }}</title>
<meta name="description" content="{{ page_description }}">
{% endblock %}
{% endraw %}
```

```templ
// Go template (after)
templ MyPage() {
    @layouts.Base(meta.New("Page Title", "Page description")) {
        // content
    }
}
```

**From Laravel (controller passes vars):**

```php
// Laravel (before)
return view('page', ['title' => 'My Page']);
```

```go
// Go handler (after) - does NOT pass meta
func (h *Handler) Page(c echo.Context) error {
    return pages.Page().Render(c.Request().Context(), c.Response().Writer)
}
```

```templ
// Go template - owns its meta
templ Page() {
    @layouts.Base(meta.New("My Page", "Description")) {
        // content
    }
}
```

**From Express (res.render with vars):**

```javascript
// Express (before)
res.render('page', { title: 'My Page' });
```

```go
// Go handler (after) - clean, no meta
func (h *Handler) Page(c echo.Context) error {
    return pages.Page().Render(c.Request().Context(), c.Response().Writer)
}
```

#### Migrating Site-Wide Config

Move hardcoded site names to environment variables:

```bash
# .envrc
export SITE_NAME="My App"
export SITE_URL="https://example.com"
```

Middleware injects into context, templates access via `meta.SiteNameFromCtx(ctx)`.

#### Preserve Existing OG Images

When converting, identify and preserve existing OG images:

1. Check `public/`, `static/`, `assets/` for og-*.png files
2. Copy to Go project's `static/images/` directory
3. Reference in templates: `meta.New("Title", "Desc").WithOGImage("/static/images/og.png")`

### Database Schema Migration

If the project has an existing database:

1. Export current schema
2. Convert to goose migration format
3. Create sqlc queries from existing SQL or ORM code

**From Prisma schema:**

```prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
}
```

Convert to goose migration (see Django example above).

**From existing SQL files:**

Copy and wrap with goose markers:

```sql
-- +goose Up
[existing CREATE TABLE statements]

-- +goose Down
[DROP TABLE statements in reverse order]
```

---

## Step 5: Incremental Migration Plan

For large projects, suggest this phased approach:

### Phase 1: Foundation (Day 1)

- Add Go project structure alongside existing code
- Set up shared database connection
- Create base templates
- Configure Air hot reload

Both apps can run simultaneously:

- Existing app: port 3000
- Go app: port 3000

### Phase 2: Static Pages (Days 2-3)

- Convert static/simple pages to Templ
- Add Tailwind CSS
- Set up HTMX for simple interactions

### Phase 3: Core Features (Days 4-7)

- Convert main CRUD operations
- Migrate database queries to sqlc
- Add authentication middleware (if using Clerk)

### Phase 4: Cleanup (Day 8+)

- Remove old framework files
- Update deployment configuration
- Update documentation

---

## Step 6: Execute Conversion

After planning, execute the conversion:

1. **Create Go structure** without removing existing files
2. **Generate migrations** from existing database schema
3. **Convert templates** one at a time
4. **Migrate routes** starting with simplest endpoints
5. **Test each conversion** before moving to next

### Verification Checklist

After conversion, verify:

- [ ] `make dev` starts the server
- [ ] All routes respond correctly
- [ ] Database queries work
- [ ] Templates render properly
- [ ] Static assets load
- [ ] Forms submit successfully

---

## Step 7: Display Summary

After conversion, display a summary to the user showing:

**Conversion Complete Header:**

- Project name
- Converted from: [detected framework]
- Conversion type: full/incremental/backend-only

**Files Created Table:**

| Category | Count |
|----------|-------|
| Go source files | X |
| Templ templates | X |
| Migrations | X |
| Config files | X |

**Files Preserved:**

List the files that were preserved (README, .git, etc.)

**Files to Remove (after testing):**

List old framework files that can be safely removed after verification.

**Next Steps:**

1. Test the Go app: `make dev`
1. Verify routes: Visit each page and test functionality
1. Run tests: `make test`
1. Remove old files (when ready): `rm -rf [old-framework-files]`
1. Update deployment: Follow deployment instructions for your platform

**Migration Notes:**

Include any specific notes about the conversion, pending items, or manual steps needed.

---

## Error Handling

If conversion encounters issues:

### Unsupported Framework

If the detected framework isn't in the migration guides:

```text
I detected [framework] which doesn't have a specific migration guide yet.

Would you like me to:
1. Attempt a general conversion (analyze your code and convert patterns)
2. Stop and let you provide more context about your project

The general conversion will:
- Analyze your route definitions
- Convert controllers/views to Go handlers
- Migrate templates to Templ syntax
- Create database migrations from your schema
```

### Complex ORM Queries

If the project uses complex ORM queries that don't map directly to SQL:

```text
I found some complex ORM patterns that need manual review:

[Example query]

This query uses [feature] which doesn't have a direct SQL equivalent.

Options:
1. Convert to multiple simpler queries
2. Keep as a raw SQL query with manual implementation
3. Skip for now and add TODO comment

Which approach would you prefer?
```

### Frontend-Heavy Projects

If the project is heavily frontend-focused (SPA):

```text
This appears to be a frontend-heavy project with [framework].

For projects with complex frontend logic, consider:
1. **Keep frontend, convert API only** - Your [React/Vue/etc] app stays the same,
   only the backend becomes Go
2. **Progressive enhancement** - Convert to HTMX gradually, keeping some React
   components where needed
3. **Full conversion** - Replace everything with Templ + HTMX (significant rewrite)

Which approach fits your needs?
```
