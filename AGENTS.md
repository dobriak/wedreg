# Agent Guidelines for Wedding & Event Invitation System

## Development Commands

### Build & Type Checking
- `npm run build` - Build production Next.js app
- `npm run check` - Run build + TypeScript type checking
- `npm run dev` - Start development server
- `npm run start` - Start production server

### Code Quality
- `npm run lint` - Run ESLint for code linting
- `npm run cf-typegen` - Generate Cloudflare environment types (run after adding new Cloudflare bindings)

### Deployment
- `npm run preview` - Build and preview Cloudflare deployment locally
- `npm run deploy` - Deploy to Cloudflare Workers

### Testing
No test framework is currently configured. When adding tests, use Vitest or Jest for unit tests and Playwright for E2E tests.

## Available Skills

The following specialized skills can be loaded to provide domain-specific instructions and workflows:

### cloudflare-nextjs
Deploy Next.js applications (App Router and Pages Router) to Cloudflare Workers using the OpenNext adapter. Use when:
- Setting up new or existing Next.js projects for Cloudflare Workers deployment
- Configuring @opennextjs/cloudflare adapter
- Developing with SSR, ISR, or server components on Cloudflare
- Integrating with Cloudflare services (D1, R2, KV, Workers AI)
- Troubleshooting worker size limits, runtime compatibility, database connection scoping, or security vulnerabilities

### wrangler
Cloudflare Workers CLI for deploying, developing, and managing Workers and related services. Load this skill before running wrangler commands to ensure correct syntax and best practices. Covers:
- Workers, KV, R2, D1, Vectorize, Hyperdrive, Workers AI
- Containers, Queues, Workflows, Pipelines, and Secrets Store

### cloudflare-d1
Build with D1 serverless SQLite database on Cloudflare's edge. Use when:
- Creating D1 databases
- Writing SQL migrations
- Querying D1 from Workers
- Handling relational data
- Troubleshooting D1_ERROR, statement too long, migration failures, or query performance issues
- Prevents 14 documented errors

## Tech Stack

- **Framework**: Next.js 16 (App Router) on Cloudflare Pages via @opennextjs/cloudflare
- **Styling**: TailwindCSS v4 with shadcn/ui components
- **Database**: Cloudflare D1 (SQLite)
- **Email**: resend.com
- **TypeScript**: Strict mode enabled

## Code Style Guidelines

### Imports
- Use ES6 imports: `import { Component } from 'module'`
- Use absolute imports with `@/` alias for src files: `import { Component } from '@/components/...'`
- Group imports in order: external libraries → internal modules → types
- Third-party imports use bare specifiers, not relative paths

### Formatting
- Use tabs for indentation (2 spaces per level)
- Trailing commas in multi-line arrays/objects/functions
- Space after colons in type definitions: `key: Type`
- No semicolons (let TypeScript/ESLint handle)
- Max line length: soft limit at 100 characters

### TypeScript
- Enable strict mode (already configured)
- Use `interface` for object shapes, `type` for unions/intersections
- Prefer explicit return types on public functions
- Use `Readonly` or `as const` for immutable data
- Avoid `any` - use `unknown` if type cannot be determined
- Generic type parameters: `T` for single generic, `K, V` for key-value pairs

### Naming Conventions
- **Components**: PascalCase (`UserProfile`, `GuestList`)
- **Functions/Methods**: camelCase (`getGuestById`, `sendInvitation`)
- **Constants**: UPPER_SNAKE_CASE (`MAX_GUESTS_PER_FAMILY`)
- **Interfaces/Types**: PascalCase, prefixed with `I` only if necessary to avoid name collision
- **Files**: lowercase-kebab-case (`guest-service.ts`, `rsvp-form.tsx`)
- **Private members**: underscore prefix (`_validateEmail`)
- **React hooks**: Start with `use` (`useGuestData`, `useRsvpSubmit`)

### Error Handling
- Use `try/catch` for async operations
- Return specific error types or objects with `message` and `code` fields
- Log errors with context: `console.error('Failed to send invitation:', { email, error })`
- Show user-friendly error messages in UI, technical details in logs
- Use early returns for error conditions to avoid deep nesting

### File Structure
```
src/
  app/                    # Next.js App Router pages
    (host)/              # Protected host routes (dashboard, guests, invitations)
      dashboard/
      guests/
      invitations/
    (guest)/             # Public guest routes
      rsvp/[token]/
    api/                 # API routes
      rsvp/
      invitations/
      families/
  components/            # Reusable UI components
  lib/                   # Utility functions and configurations
  db/                    # Database queries and migrations
  types/                 # Shared TypeScript types
```

### React/Next.js Best Practices
- Use Server Components by default (no `use client` directive)
- Only add `use client` for interactive features (forms, modals, stateful UI)
- Use `useSearchParams()` instead of `window.location.search`
- Use `cookies()` for server-side cookie access
- Avoid `useEffect` for data fetching - use Server Components or Server Actions
- Use Server Actions for form submissions and mutations
- Use `Link` component from `next/link` for navigation
- Use `Image` component from `next/image` for images

### API Routes
- Use Next.js App Router route handlers (`export async function GET/POST()`)
- Return appropriate HTTP status codes (200, 400, 401, 404, 500)
- Use `Response.json()` for JSON responses
- Validate request data before processing
- Handle CORS appropriately for public endpoints

### Database (Cloudflare D1)
- Use prepared statements to prevent SQL injection
- Use parameterized queries: `db.prepare('SELECT * FROM guests WHERE id = ?1').bind(id)`
- Wrap DB operations in try/catch blocks
- Use transactions for multi-step operations
- Define migrations in `migrations/` directory with descriptive names

### Email (Resend)
- Use email templates stored in `templates/` directory
- Include unsubscribe links in marketing emails
- Sanitize all user-generated content before sending
- Test email templates in development environment
- Use rate limiting for bulk email sends

### Styling (TailwindCSS + shadcn)
- Prefer shadcn components when available
- Use Tailwind utility classes for custom styles
- Use CSS variables for theming: `var(--background)`, `var(--foreground)`
- Support dark mode using `prefers-color-scheme` media query
- Use responsive prefixes: `sm:`, `md:`, `lg:`, `xl:`
- Keep component styling close to component logic (colocation)

### Security
- Never commit secrets or API keys (use .env.local)
- Validate all user inputs on server-side
- Use secure random tokens for RSVP links (`crypto.randomUUID()`)
- Set appropriate CSP headers in next.config.ts
- Implement rate limiting on sensitive endpoints
- Use HTTPS in production

### Git Workflow
- Write descriptive commit messages in imperative mood: "Add guest list import feature"
- Run `npm run lint` and `npm run check` before committing
- Use feature branches for new functionality
- Keep PRs focused and reasonably sized
- Ensure all tests pass before merging

### Additional Notes
- Always initialize Cloudflare context in development: imported from `@opennextjs/cloudflare` in next.config.ts
- Environment interface is defined in `env.d.ts` - regenerate with `cf-typegen` after adding bindings
- The app uses magic links for guest authentication (no passwords required)
- RSVP links should expire after event date + 30 days
