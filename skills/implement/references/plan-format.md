# Supported Plan File Formats

The `/implement` skill supports the following plan file format.

## Standard Format (Pipeline-Compatible)

This is the primary format, compatible with the existing `claude-pipeline` tool.

```markdown
# Plan: (overall goal title)

## Task 1: (short task title)
- depends: none
- files: path/to/file1.ts, path/to/file2.ts

(detailed description of what to do)

## Task 2: (short task title)
- depends: 1
- files: path/to/file3.ts

(detailed description)

## Task 3: (short task title)
- depends: 1, 2
- files: path/to/file4.ts, path/to/file5.ts

(detailed description)
```

### Field Reference

| Field | Required | Format | Example |
|-------|----------|--------|---------|
| `## Task N:` | Yes | Heading with task number and title | `## Task 1: Add auth middleware` |
| `- depends:` | No | Comma-separated task numbers, or `none` | `- depends: 1, 2` |
| `- files:` | No | Comma-separated file paths | `- files: src/auth.ts, src/middleware.ts` |
| Description | Yes | Free-form text after the metadata lines | Detailed implementation instructions |

### Rules

- Task numbers must be sequential starting from 1
- `depends: none` or omitting the depends line means no dependencies
- `depends: 1, 3` means this task requires Tasks 1 and 3 to complete first
- The `files:` field is informational — it hints which files will be modified
- Description should be specific enough for implementation without ambiguity
- A blank line should separate the metadata lines from the description

### Example Plan

```markdown
# Plan: Add user authentication to the API

## Task 1: Create user model
- depends: none
- files: src/models/user.ts, prisma/schema.prisma

Add a User model with fields: id, email, passwordHash, createdAt, updatedAt.
Run prisma migrate to create the table.

## Task 2: Add auth middleware
- depends: 1
- files: src/middleware/auth.ts, src/types/express.d.ts

Create JWT-based auth middleware that:
- Extracts token from Authorization header
- Verifies and decodes the token
- Attaches user to req.user
- Returns 401 for invalid/missing tokens

## Task 3: Add login endpoint
- depends: 1
- files: src/routes/auth.ts, src/routes/index.ts

Create POST /api/auth/login that:
- Accepts { email, password }
- Validates credentials against User model
- Returns JWT token on success
- Returns 401 on failure

## Task 4: Protect existing routes
- depends: 2, 3
- files: src/routes/index.ts

Add auth middleware to all routes under /api/ except /api/auth/*.
```
