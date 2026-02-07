# Proposal: Guest List Management Interface

## Why

The core of any wedding or event management system is the ability to organize and track guest information. Currently, hosts need a comprehensive interface to manage families, import/export guest data, and search/filter their guest lists. This change provides the foundation for all other features (invitations, RSVPs, reporting) by establishing the data model and management capabilities for guest lists.

## What Changes

- **Create family management system**: Full CRUD operations for families with primary contact and additional members
- **Implement CSV import/export**: Template-based import for bulk guest data and export for reporting
- **Add search and filtering**: Real-time search and multi-filter capabilities for guest lists
- **Define data model**: Family structure with primary contact, additional members, and associated metadata
- **Create management UI**: Forms and table views for managing guest lists

## Capabilities

### New Capabilities
- `family-management`: Core CRUD operations for families and family members, including validation rules for primary contacts, family structure, and member relationships
- `guest-list-io`: CSV import/export functionality with template support, bulk operations, and data validation
- `guest-list-search`: Search and filtering capabilities across family data with support for multiple filter criteria

### Modified Capabilities
- None (no existing specs to modify)

## Impact

**Affected Components:**
- New database schema for families, family members, and related tables
- New API routes for family CRUD operations, import/export, and search
- New UI components in `app/(host)/guests/` for management interface
- Shared utilities for CSV parsing, validation, and search logic

**Dependencies:**
- Cloudflare D1 for data persistence
- Resend.com for email (future integration with invitations)
- TailwindCSS + shadcn/ui for UI components

**Integration Points:**
- Will be used by invitation system to send to families
- Will be used by RSVP system to track responses
- Will be displayed in dashboard statistics and reports
