# Tasks: Guest List Management Interface

## 1. Database Schema

- [ ] 1.1 Create database migration for `families` table with columns: id (primary key), family_name (text, not null), notes (text, nullable), created_at (timestamp), updated_at (timestamp)
- [ ] 1.2 Create database migration for `family_members` table with columns: id (primary key), family_id (foreign key to families), first_name (text, not null), last_name (text, not null), email (text, nullable), phone (text, nullable), age_group (text: 'Adult'|'Child', not null), primary_contact (boolean, not null), notes (text, nullable), created_at (timestamp), updated_at (timestamp)
- [ ] 1.3 Add index on `families.family_name` for sorting and search performance
- [ ] 1.4 Add foreign key constraint from `family_members.family_id` to `families.id` with cascade delete
- [ ] 1.5 Add composite index on `family_members` (family_id, primary_contact) for query optimization
- [ ] 1.6 Run migrations on local development database using `wrangler d1 migrations apply`

## 2. Database Queries and Types

- [ ] 2.1 Create TypeScript types for Family and FamilyMember interfaces in `src/types/family.ts`
- [ ] 2.2 Create database query functions in `src/db/families.ts` for basic CRUD operations (createFamily, getFamily, updateFamily, deleteFamily, listFamilies)
- [ ] 2.3 Create database query functions in `src/db/family-members.ts` for member CRUD operations (addMember, updateMember, deleteMember, getFamilyMembers)
- [ ] 2.4 Create validation function in `src/lib/validation/family.ts` to enforce one primary contact per family
- [ ] 2.5 Create validation function to validate email format using regex
- [ ] 2.6 Create query function for family statistics (total families, total guests, average party size)

## 3. API Routes - Family CRUD

- [ ] 3.1 Create API route `POST /api/families` to create new family with members, including validation of required fields and email format
- [ ] 3.2 Create API route `GET /api/families` to list families with pagination (page, limit, sort parameters)
- [ ] 3.3 Create API route `GET /api/families/[id]` to retrieve single family with all members
- [ ] 3.4 Create API route `PUT /api/families/[id]` to update family-level information (family_name, notes)
- [ ] 3.5 Create API route `DELETE /api/families/[id]` to delete family and all associated members
- [ ] 3.6 Add error handling for 404 (family not found) and 400 (validation errors) in all routes
- [ ] 3.7 Add authentication middleware to protect all family API routes (host-only access)

## 4. API Routes - Family Members

- [ ] 4.1 Create API route `POST /api/families/[id]/members` to add new family member to existing family
- [ ] 4.2 Create API route `PUT /api/families/[id]/members/[memberId]` to update individual member details
- [ ] 4.3 Create API route `DELETE /api/families/[id]/members/[memberId]` to delete member with validation to prevent deleting last member or primary contact
- [ ] 4.4 Add validation to ensure exactly one primary contact per family on member creation/update
- [ ] 4.5 Add error handling for invalid family_id or member_id

## 5. CSV Import/Export Utilities

- [ ] 5.1 Install papaparse dependency: `npm install papaparse @types/papaparse`
- [ ] 5.2 Create CSV export function in `src/lib/csv/export.ts` that converts family data to CSV format with proper escaping
- [ ] 5.3 Create CSV export function with optional family member columns
- [ ] 5.4 Create CSV import parser in `src/lib/csv/import.ts` that validates format and required columns
- [ ] 5.5 Create CSV validation function to check required fields (family_name, primary contact names and email)
- [ ] 5.6 Create CSV validation function to validate email format and age_group values
- [ ] 5.7 Create CSV validation function to parse Additional Members JSON column
- [ ] 5.8 Create CSV import function that creates families and members from validated data using transactions

## 6. API Routes - CSV Import/Export

- [ ] 6.1 Create API route `GET /api/families/export` that generates and returns CSV file with headers and data
- [ ] 6.2 Add query parameter `include_members` to control whether to include member columns in export
- [ ] 6.3 Create API route `GET /api/families/template` that returns downloadable CSV template with example data
- [ ] 6.4 Create API route `POST /api/families/import/validate` that parses uploaded CSV and returns validation summary (valid rows, invalid rows with errors)
- [ ] 6.5 Create API route `POST /api/families/import/confirm` that creates families from validated CSV data
- [ ] 6.6 Add progress feedback for large imports using streaming or batch processing
- [ ] 6.7 Add error handling for malformed CSV files, encoding issues, and large file uploads

## 7. Search Functionality

- [ ] 7.1 Create client-side search function in `src/lib/search/client.ts` that filters cached family data by name, email, and member names
- [ ] 7.2 Create debounced search input component with 300ms delay
- [ ] 7.3 Create API route `GET /api/families/search` for server-side search with query parameter `q`
- [ ] 7.4 Implement server-side search query in D1 using LIKE operator on family_name, first_name, last_name, and email columns
- [ ] 7.5 Add logic to automatically switch between client-side and server-side search based on family count (> 500 families = server-side)
- [ ] 7.6 Add manual toggle for search mode in UI preferences

## 8. Filter Functionality

- [ ] 8.1 Create client-side filter function that filters families by RSVP status (Not Sent, Sent, Responded)
- [ ] 8.2 Create client-side filter function that filters families by party size (1 person, 2-4 people, 5-10 people, 10+ people)
- [ ] 8.3 Create combinator function that applies multiple filters together (search + status + party size)
- [ ] 8.4 Update API routes to support server-side filtering with query parameters (status, min_size, max_size)
- [ ] 8.5 Add URL state management to sync search and filter state with query parameters

## 9. UI Components - Family List

- [ ] 9.1 Create page layout at `src/app/(host)/guests/page.tsx` with header, search bar, filters, and family table
- [ ] 9.2 Create FamilyTable component using shadcn/ui Table component to display families with columns: Family Name, Primary Contact, Party Size, RSVP Status, Actions
- [ ] 9.3 Create SearchInput component with debounced typing and clear button
- [ ] 9.4 Create StatusFilter component with dropdown to select RSVP status filter
- [ ] 9.5 Create PartySizeFilter component with buttons or dropdown to select party size filter
- [ ] 9.6 Add pagination controls to FamilyTable component (Previous, Next, Page X of Y)
- [ ] 9.7 Add loading states and empty states to FamilyTable component
- [ ] 9.8 Implement responsive design for family list (table scrolls horizontally on mobile)

## 10. UI Components - Family Form

- [ ] 10.1 Create CreateFamilyDialog component using shadcn/ui Dialog with form for new family
- [ ] 10.2 Create EditFamilyDialog component using shadcn/ui Dialog with form to edit family information
- [ ] 10.3 Create FamilyForm component with fields: Family Name (required), Primary Contact First/Last Name (required), Primary Contact Email (required), Primary Contact Phone (optional), Family Notes (optional)
- [ ] 10.4 Create FamilyMembersList component to display and manage additional family members within FamilyForm
- [ ] 10.5 Create AddMemberForm component to add additional family member with fields: First/Last Name (required), Age Group (dropdown: Adult/Child), Email (optional), Phone (optional), Notes (optional)
- [ ] 10.6 Add validation error display for all form fields
- [ ] 10.7 Add confirmation dialog before deleting family

## 11. UI Components - Member Management

- [ ] 11.1 Create EditMemberDialog component using shadcn/ui Dialog to edit individual member details
- [ ] 11.2 Create DeleteMemberButton component with confirmation for removing additional family members
- [ ] 11.3 Add visual indication of primary contact status in member list (badge or icon)
- [ ] 11.4 Prevent deletion of primary contact when other members exist with clear error message
- [ ] 11.5 Handle deletion of last remaining member (deletes entire family) with confirmation

## 12. UI Components - CSV Import/Export

- [ ] 12.1 Create ExportButton component in family list header to trigger CSV download
- [ ] 12.2 Create ImportButton component using shadcn/ui Button with file upload dialog
- [ ] 12.3 Create ImportPreviewDialog component using shadcn/ui Dialog to show import summary: total rows, valid rows, invalid rows with error messages
- [ ] 12.4 Create ImportProgress component to show progress during large CSV imports
- [ ] 12.5 Create DownloadTemplateButton component to download CSV template
- [ ] 12.6 Add drop zone for CSV file upload in ImportButton component

## 13. Integration and State Management

- [ ] 13.1 Create React hook `useFamilies` to fetch and cache family list data
- [ ] 13.2 Create React hook `useFamily` to fetch single family by ID
- [ ] 13.3 Create React hook `useCreateFamily` to handle family creation with optimistic updates
- [ ] 13.4 Create React hook `useUpdateFamily` to handle family updates
- [ ] 13.5 Create React hook `useDeleteFamily` to handle family deletion with confirmation
- [ ] 13.6 Create React hook `useImportCSV` to handle CSV import flow (upload, validate, confirm)
- [ ] 13.7 Implement optimistic updates for family edits and member additions
- [ ] 13.8 Implement cache invalidation after successful mutations (create, update, delete, import)

## 14. Navigation and Routing

- [ ] 14.1 Add "Guests" link to host dashboard navigation menu
- [ ] 14.2 Implement URL query parameter sync for search and filters (e.g., `/guests?q=smith&status=sent`)
- [ ] 14.3 Preserve search and filter state when navigating away and back to guests page
- [ ] 14.4 Add breadcrumb navigation from dashboard to guests page
- [ ] 14.5 Add page title and metadata for guests page

## 15. Error Handling and Notifications

- [ ] 15.1 Add toast notification system using shadcn/ui toast for success/error messages
- [ ] 15.2 Show success notification after creating, updating, or deleting families
- [ ] 15.3 Show error notifications for failed operations (validation errors, API errors)
- [ ] 15.4 Display import summary notification after CSV import completes
- [ ] 15.5 Add retry mechanism for failed API calls (network errors, timeout)
- [ ] 15.6 Add user-friendly error messages for common errors (e.g., "Primary contact email is required")

## 16. Testing

- [ ] 16.1 Write unit tests for database query functions in `src/db/families.test.ts`
- [ ] 16.2 Write unit tests for database query functions in `src/db/family-members.test.ts`
- [ ] 16.3 Write unit tests for CSV export function in `src/lib/csv/export.test.ts`
- [ ] 16.4 Write unit tests for CSV import and validation in `src/lib/csv/import.test.ts`
- [ ] 16.5 Write unit tests for client-side search function in `src/lib/search/client.test.ts`
- [ ] 16.6 Write unit tests for validation functions in `src/lib/validation/family.test.ts`
- [ ] 16.7 Write integration tests for API routes using Next.js test utilities
- [ ] 16.8 Write E2E tests for core workflows (create family, import CSV, search families) using Playwright

## 17. Documentation

- [ ] 17.1 Add API documentation for all family API routes using OpenAPI/Swagger format
- [ ] 17.2 Create user guide for guest list management features
- [ ] 17.3 Document CSV template format and import/export process
- [ ] 17.4 Add inline code comments for complex business logic
- [ ] 17.5 Update README.md with new features and links to documentation

## 18. Performance Optimization

- [ ] 18.1 Add virtualization to FamilyTable component for large lists (react-window or react-virtual)
- [ ] 18.2 Implement pagination caching to avoid re-fetching pages
- [ ] 18.3 Add loading skeletons for better perceived performance
- [ ] 18.4 Optimize database queries with proper indexing and query patterns
- [ ] 18.5 Add request debouncing for search input
- [ ] 18.6 Implement request cancellation for abandoned search queries

## 19. Accessibility

- [ ] 19.1 Add ARIA labels to all interactive elements
- [ ] 19.2 Ensure keyboard navigation works for all forms and dialogs
- [ ] 19.3 Add focus management for modal dialogs
- [ ] 19.4 Test color contrast ratios for text and buttons
- [ ] 19.5 Add screen reader announcements for dynamic content updates
- [ ] 19.6 Ensure forms have proper error associations and descriptions

## 20. Deployment and Verification

- [ ] 20.1 Run `npm run lint` and fix all linting errors
- [ ] 20.2 Run `npm run check` to verify TypeScript types are correct
- [ ] 20.3 Build production bundle with `npm run build`
- [ ] 20.4 Test application locally with `npm run dev` and verify all features work
- [ ] 20.5 Deploy database migrations to production Cloudflare D1
- [ ] 20.6 Deploy application to Cloudflare Pages
- [ ] 20.7 Smoke test core workflows in production environment
- [ ] 20.8 Verify email templates and CSV downloads work correctly in production
