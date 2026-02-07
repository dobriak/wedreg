# Design: Guest List Management Interface

## Context

This is the foundational feature for the wedding/event management system. Currently, no guest list functionality exists. The system runs on Next.js 16 with App Router, hosted on Cloudflare Pages, using Cloudflare D1 for persistence and TailwindCSS + shadcn/ui for the UI.

The host dashboard already exists at `app/(host)/dashboard/`, and the guest list management will be a new section at `app/(host)/guests/`. This change needs to establish a clean data model that will be used by future features like invitations and RSVPs.

## Goals / Non-Goals

**Goals:**
- Create a scalable family data model that supports the full guest lifecycle (invitations, RSVPs, reporting)
- Provide CRUD operations for families with validation and error handling
- Enable bulk import/export for efficient guest list management
- Implement search and filtering to quickly find specific guests
- Build a responsive, mobile-friendly UI using existing design system (shadcn/ui)

**Non-Goals:**
- Invitation sending (will be built later using the family data model)
- RSVP collection and tracking (will be built later)
- Multi-event support (single event per system for now)
- Advanced reporting beyond basic CSV export

## Decisions

### Database Schema

**Decision**: Use normalized schema with separate `families` and `family_members` tables connected by a foreign key.

**Rationale**:
- Families can have 1-N members, so separate tables avoid nullable columns or JSON fields
- Easier to query individual members for RSVP tracking and meal preferences
- Supports future features like dietary restrictions, meal preferences at member level
- Normalized approach is better for data integrity and reporting

**Alternative Considered**: Single table with JSON column for family members. Rejected because:
- Harder to query individual members
- JSON fields are less type-safe in SQLite
- Makes filtering and aggregation more complex

### Family Structure

**Decision**: Enforce exactly one primary contact per family, with required email and optional phone.

**Rationale**:
- Primary contact is the single point of communication (invitations, reminders)
- Email is required for magic-link based RSVP system (no passwords)
- Phone is optional for hosts who prefer SMS or need backup contact method
- Simplifies invitation system by always knowing who to send to

**Alternative Considered**: Allow multiple primary contacts. Rejected because:
- Adds complexity to invitation sending logic
- Increases risk of duplicate invitations
- Primary use case is one contact per household

### CSV Import/Export

**Decision**: Use simple CSV parsing with papaparse library for import, native Node.js streams for export.

**Rationale**:
- papaparse is lightweight, handles edge cases (quoted fields, commas in data)
- Stream-based export handles large guest lists without memory issues
- No complex ETL needed - data is flat (family + members in rows)

**Alternative Considered**: Use complex multi-sheet Excel export. Rejected because:
- Adds heavy dependencies
- CSV is sufficient for bulk import/export
- More formats can be added later if needed

### Search Implementation

**Decision**: Client-side search for smaller lists (< 500 families), server-side search with full-text index for larger lists.

**Rationale**:
- Most wedding guest lists are under 500 families, client-side is instant
- Server-side search avoids loading all data into browser for large lists
- Full-text index in D1 provides fast prefix/suffix matching

**Alternative Considered**: Always use server-side search. Rejected because:
- Adds network latency for small lists
- Over-engineering for typical use case

### UI Component Structure

**Decision**: Use shadcn/ui components (Table, Dialog, Form, Button, Input) with custom wrappers for specific use cases.

**Rationale**:
- Maintains consistency with existing design system
- shadcn/ui provides accessible, well-tested components
- Custom wrappers allow for business logic (family form, member list)
- Reduces amount of custom CSS

**Alternative Considered**: Build all components from scratch. Rejected because:
- Duplicates effort - shadcn/ui provides 80% of what we need
- Risk of inconsistent styling
- Less accessibility testing

### API Route Structure

**Decision**: RESTful API with separate routes for CRUD operations and bulk operations.

```
GET/POST    /api/families              # List/create families
GET/PUT/DELETE /api/families/[id]      # Read/update/delete family
GET/POST    /api/families/[id]/members # List/add members
PUT/DELETE  /api/families/[id]/members/[id]  # Update/delete member
POST        /api/families/import       # CSV import
GET         /api/families/export       # CSV export
GET         /api/families/search       # Search endpoint (for large lists)
```

**Rationale**:
- RESTful conventions are familiar and predictable
- Separating bulk operations keeps CRUD endpoints clean
- Nested routes for family members reflect data model

**Alternative Considered**: GraphQL. Rejected because:
- Overkill for this use case
- Adds complexity and dependency
- REST is sufficient for CRUD operations

## Risks / Trade-offs

### Risk: Large guest lists (> 500 families) may cause performance issues with client-side search
**Mitigation**: Implement server-side search endpoint with pagination; add "search on server" toggle for large lists

### Risk: CSV import with invalid data could corrupt database
**Mitigation**: Validate all data before insertion; use transactions for bulk imports; provide detailed error feedback with row numbers

### Risk: Deleting families could orphan RSVP data (when RSVPs are implemented)
**Mitigation**: Soft-delete families (mark as deleted) instead of hard-delete; add cascade delete constraint after RSVP system is in place

### Risk: Concurrency issues if multiple hosts edit same family simultaneously
**Mitigation**: Implement optimistic locking with version column; for initial release, accept last-write-wins with clear timestamp display

### Risk: Complex family situations (separated parents, step-families) don't fit simple model
**Mitigation**: Allow users to create separate family entries; add "notes" field for special cases; document this pattern in help text

### Trade-off: Simple UI vs. Advanced Features
**Decision**: Start simple with basic CRUD; add advanced features (bulk edit, duplicate families) in follow-up changes based on user feedback

## Migration Plan

### Deployment Steps

1. **Database Migration**: Create `families` and `family_members` tables
   - Run migration: `wrangler d1 migrations apply <database_name>`
   - No data migration needed (new feature)

2. **Deploy API Routes**: Add new API routes at `app/api/families/`
   - Deploy to Cloudflare Workers
   - Verify endpoints return correct responses

3. **Deploy UI Components**: Add guest list management pages
   - Deploy to Cloudflare Pages
   - Verify functionality in browser

4. **Testing**: Smoke test core workflows
   - Create family, add members, edit, delete
   - Import CSV, export CSV
   - Search and filter

### Rollback Strategy

- Delete migration (no data to lose)
- Remove API routes
- Remove UI components
- Database schema changes are atomic, can be rolled back cleanly

## Open Questions

1. **Should we support plus-ones as separate families or family members?**
   - Current approach: Add as additional family member in existing family
   - Alternative: Create single-person families for plus-ones
   - Decision: Use additional family members for simplicity; revisit if feedback indicates need

2. **Should email addresses be unique across families?**
   - Current approach: No uniqueness constraint (same person could be invited as part of different family units)
   - Alternative: Enforce uniqueness at DB level
   - Decision: No uniqueness constraint for flexibility; can add validation later if needed

3. **What validation rules for CSV import?**
   - Current approach: Required fields (family name, primary contact email), optional everything else
   - Alternative: Strict validation on all columns
   - Decision: Flexible validation to accommodate incomplete data; warn on missing optional fields
